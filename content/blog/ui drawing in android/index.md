+++
title = "UI Drawing in Android"
date = 2024-07-23
[taxonomies]
  tags = ["Android", "Note", "UI", "render"]

[extra]
  toc = true
+++

- Android drawing process (Android 6.0): from ViewRootImpl.performTraversals → to child View.draw.

![Reference: [Android硬件加速原理与实现简介](https://tech.meituan.com/2017/01/19/hardware-accelerate.html)](UI%20Drawing%20in%20Android%20941636133aa347d5bf94dbb12027accc/Untitled.png)

*Reference: [Android硬件加速原理与实现简介](https://tech.meituan.com/2017/01/19/hardware-accelerate.html)*

- What happens after calling View.invalidate()?
    
    *Reference: i) [源码分析-Android-View-invalidate 绘制流程](https://juejin.cn/post/7100121390090551332), ii) [Android-requestLayout与invalidate方法解析](https://ljd1996.github.io/2020/11/23/Android-requestLayout%E4%B8%8Einvalidate%E6%96%B9%E6%B3%95%E8%A7%A3%E6%9E%90/).*
    
    ```
    View.invalidate()
    -View.invalidate(boolean invalidateCache)
    --View.invalidateInternal(..)
    ---ViewGroup.invalidateChild(..)
    ----ViewParent.invalidateChildInParent(..)
    ----ViewRootImpl.invalidateChildInParent(..)
    -----ViewRootImpl.invalidateRectOnScreen(..)
    ------ViewRootImpl.scheduleTraversals(..)
    -------ViewRootImpl.mTraversalRunnable.run()
    --------ViewRootImpl.performTraversals()
    ---------ViewRootImpl.performMeasure(..)
    ----------View.measure(..)
    ---------ViewRootImpl.performLayout(..)
    ----------..
    ---------ViewRootImpl.performDraw()
    ----------..
    ```
    
    - From `ViewRootImpl.performDraw()` to `View.onDraw()` calling, print the stack trace of calling hierarchy.
    - For `View.requestLayout()`, it will call `ViewRootImpl.requestLayout()`, then `ViewRootImpl.mLayoutRequested` would be set to true, and this true variable would be used in the processing of deciding whether to perform measure & layout.
- How does Choreographer receive the VSync time plusing?
    
    [android drawing note.pdf](UI%20Drawing%20in%20Android%20941636133aa347d5bf94dbb12027accc/android_drawing_note.pdf)

    ![android drawing note](UI%20Drawing%20in%20Android%20941636133aa347d5bf94dbb12027accc/android_drawing_note.png)
    
- Main works done by Choreographer for each frame after receiving a VSync signal:
    
    *Reference: [Android基于Choreographer的渲染机制详解](https://androidperformance.com/2019/10/22/Android-Choreographer/#/%E6%BA%90%E7%A0%81%E5%B0%8F%E7%BB%93)*
    
    ```java
    // Choreographer.java (android.view)
    
    // From onVSync() to self.run(), then doFrame()
    void doFrame(long frameTimeNanos, int frame) {
        // i) calculate skipped frames
        // ii) log frames in FrameInfo
        // iii) doCallbacks()
        ...
        doCallbacks(Choreographer.CALLBACK_INPUT, frameIntervalNanos);
        doCallbacks(Choreographer.CALLBACK_ANIMATION, frameIntervalNanos);
        doCallbacks(Choreographer.CALLBACK_INSETS_ANIMATION, frameIntervalNanos);
        doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameIntervalNanos);
        doCallbacks(Choreographer.CALLBACK_COMMIT, frameIntervalNanos);
        ...
    }
    ```
    
- A Stack dumped within a custom imageView’s onDraw().
    
    ```
    onDraw:34, LogImageView (com.tbuonomo.viewpagerdotsindicator.log)
    draw:24289, View (android.view)
    updateDisplayListIfDirty:23099, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    drawChild:233, BaseDotsIndicator (com.tbuonomo.viewpagerdotsindicator)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    1 hidden frame
    draw:24292, View (android.view)
    updateDisplayListIfDirty:23099, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    updateDisplayListIfDirty:23090, View (android.view)
    draw:23997, View (android.view)
    drawChild:4662, ViewGroup (android.view)
    dispatchDraw:4415, ViewGroup (android.view)
    draw:24292, View (android.view)
    draw:909, DecorView (com.android.internal.policy)
    updateDisplayListIfDirty:23099, View (android.view)
    updateViewTreeDisplayList:713, ThreadedRenderer (android.view)
    updateRootDisplayList:719, ThreadedRenderer (android.view)
    draw:821, ThreadedRenderer (android.view)
    draw:5634, ViewRootImpl (android.view)
    performDraw:5279, ViewRootImpl (android.view)
    performTraversals:4309, ViewRootImpl (android.view)
    doTraversal:2799, ViewRootImpl (android.view)
    run:10351, ViewRootImpl$TraversalRunnable (android.view)
    run:1525, Choreographer$CallbackRecord (android.view)
    run:1534, Choreographer$CallbackRecord (android.view)
    doCallbacks:1089, Choreographer (android.view)
    doFrame:979, Choreographer (android.view)
    run:1508, Choreographer$FrameDisplayEventReceiver (android.view)
    handleCallback:958, Handler (android.os)
    dispatchMessage:99, Handler (android.os)
    loopOnce:255, Looper (android.os)
    loop:364, Looper (android.os)
    main:8938, ActivityThread (android.app)
    1 hidden frame
    run:572, RuntimeInit$MethodAndArgsCaller (com.android.internal.os)
    main:1053, ZygoteInit (com.android.internal.os)
    ```