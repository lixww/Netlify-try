+++
title = "Service Metamorphosis"
date = 2024-07-29
[taxonomies]
  tags = ["Android", "Note", "service", "aidl"]

[extra]
  toc = true
+++


# AIDL to a ‘Service'

Example: `IAccessibilityServiceClient.aidl` 

```java
IAccessibilityServiceClient.aidl 
→ Rebuild
- .intermediates/public interface IAccessibilityServiceClient extends andorid.os.IInterface
-- .intermediates/public static abstract class Stub extends android.os.Binder implements android.accessibilityservice.IAccessibilityServiceClient
//here the Binder is implemented by builder.

→ As server
- AccessibilityService#IAccessibilityServiceClientWrapper extends IAccessibilityServiceClient.Stub

→ As client
- AccessibilityServiceConnection#onServiceConnected()
-- (IAccessibilityServiceClient)mServiceInterface = IAccessibilityServiceClient.Stub.asInterface((IBinder)service);
```

*Reference: Android Interface Definition Language (AIDL), [developer.android](https://developer.android.com/develop/background-work/services/aidl)*

*Reference: [不得不说的Android Binder机制与AIDL](https://juejin.cn/post/6994057245113729038)*

<br/>

👇 We could see that there are 3 processes that need to go across in the using of accessibility service on an app, based on AIDLs. 

From which the red flow happens when the assisted app try sending an accessibility event to an accessibility service through two AIDLs (`IAccessibilityManager.aidl` and `IAccessibilityServiceClient.aidl`), where AccessibilityManagerService acts as an proxy communicating the assisted app and the accessibility service. And the blue flow happens when the accessibility service try requesting some informations from the assisted app, again through two AIDLs (`IAccessibilityServiceConnection.aidl` and `IAccessibilityInteractionConnection.aidl`), where AccessibilityManagerService again acts as the broker between them.

![Reference: [抖音Android无障碍开发知识总结](https://blog.csdn.net/ByteDanceTech/article/details/119686890)](Service%20Metamorphosis%202cffb104a5aa4d488c0665a4ac27194b/Untitled.png)

*Reference: [抖音Android无障碍开发知识总结](https://blog.csdn.net/ByteDanceTech/article/details/119686890)*

# Service as a component

Example: `AccessibilityService.java` and its subclass `TalkbackService.java`

```java
public abstract class AccessibilityService extends Service {}

public class TalkBackService extends AccessibilityService .. {}
```

# Readings

Cross-processes services used in the launch of an activity.

*Reference: [基于Android13的Activity启动流程分析](https://juejin.cn/post/7211801284709548093)*

![Untitled](Service%20Metamorphosis%202cffb104a5aa4d488c0665a4ac27194b/Untitled%201.png)