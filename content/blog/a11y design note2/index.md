+++
title = "Accessibility 设计调研 note 2"
date = 2022-11-16
[taxonomies]
  tags = ["note", "accessibility", "Android"]

[extra]
  toc = true
+++

# 系统AccessibilityService何时向view索取节点？

首先自定义AccessibilityService通过父类AccessibilityService提供的接口来完成提示操作，父类AccessibilityService能够提供的能力即描述在了 IAccessibilityServiceConnection.aidl 当中，并且会经由AccessibilityInteractionClient。
那么来看看父类AccessibilityService是怎么使用的系统给无障碍服务提供的API的，索取节点总之是发生在这个过程中的。而这些使用是自定义AccessibilityService通过父类主动发起的呢，还是父类接收到了 IAccessibilityServiceClient.onAccessibilityEvent 的响应要求，再去被动发起的呢？即需求的原点是在自定义AccessibilityService这边，还是被辅助app这边。
//@lixiaowei.xw 打印下View.AccessibilityDelegate的调用堆栈分析看看

对于onAccessibilityEvent接口，被辅助app应当是发起服务的源头。

1. 首先，onAccessibilityEvent是一个什么样的接口？
```java
/**
 * 这是 View.AccessibilityEvents 的回调，owner是View。
 * Callback for {@link android.view.accessibility.AccessibilityEvent}s.
 *
 * @param event The new event. This event is owned by the caller and cannot be used after
 * this method returns. Services wishing to use the event after this method returns should
 * make a copy.
 */
public abstract void onAccessibilityEvent(AccessibilityEvent event);
```

2. 先看看自定义AccessibilityService - TalkbackService 是如何处理 onAccessibilityEvent的。
```java
//TalkbackService
@Override
public void onAccessibilityEvent(AccessibilityEvent event) {
  Performance perf = Performance.getInstance();
  EventId eventId = perf.onEventReceived(event);
  // 统一交给一个processor处理
  accessibilityEventProcessor.onAccessibilityEvent(event, eventId);
  perf.onHandlerDone(eventId);
 
  //盲文服务
  if (brailleDisplay != null) {
    brailleDisplay.onAccessibilityEvent(event);
  }
}
```
 <br/>

```java
//AccessibilityEventProcessor
public void onAccessibilityEvent(AccessibilityEvent event, EventId eventId) {

  //要不要因为重新聚焦的问题而放弃此次event
  if (shouldDropRefocusEvent(event)) {
    return;
  }
  //要不要放弃此次event
  if (shouldDropEvent(event)) {
    return;
  }

  if (event.getEventType() == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {
    lastWindowStateChanged = SystemClock.uptimeMillis();
  }

  if (event.getEventType() == AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED
      || event.getEventType() == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED
      || event.getEventType() == AccessibilityEvent.TYPE_WINDOWS_CHANGED) {
            service.setRootDirty(true);
    }

  //要保留一下event的备份
  // We need to save the last focused event so that we can filter out related selected events.
  if (event.getEventType() == AccessibilityEvent.TYPE_VIEW_FOCUSED) {
    if (lastFocusedEvent != null) {
      lastFocusedEvent.recycle();
    }
    //备份event
    lastFocusedEvent = AccessibilityEvent.obtain(event);
  }

  if (AccessibilityEventUtils.eventMatchesAnyType(event, MASK_DELAYED_EVENT_TYPES)) {
    //延迟一下再处理
    handler.postProcessEvent(event, eventId);
  } else {
    //现在就继续处理
    processEvent(event, eventId);
  }

  handler.refreshIdleMessage();
}
```

- 是否有自定义AccessibilityService主动发起的服务呢？

    首先看看哪些接口是自定义AccessibilityService有可能主动发起的。这些接口的使用必要要依赖于父类AccessibilityService，因为需要这些接口通过AccessibilityInteractionClient 被约束了使用，并且通过connectionId，又被限制为只能经由父类AccessibilityService来使用。尽管如此，只要在父类AccessibilityService中是public的，自定义AccessibilityService就能够使用到。
```kotlin
getWindows
disableSelf
dispatchGesture
getScale
getCenterX
getCenterY
getManificationRegion
reset(magnification)
setScale
setCenter
getShowMode
setShowMode
switchToInputMethod
getAccessibilityButtonController
getSystemActions
performGlobalAction
findFocus
getServiceInfo
takeScreenShot
setGestureDescriptionPassthroughRegion
setTouchExplorationPassthroughRegion

----------
getWindowsOnAllDisplays
getRootInActiveWindow
getFingerprintGestureController
```

@todo