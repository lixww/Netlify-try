+++
title = "Main Thread Review"
date = 2024-07-17
[taxonomies]
  tags = ["Note", "Android", "Thread", "UI"]

[extra]
  toc = true
+++


> The MainThread, the UIThread, in android.
> 

These services can create an ActivityThread by `ActivityThread.systemMain()` and get the system context by `ActivityThread#systemMain()` .

    frameworks/base/cmds/svc/src/com/android/commands/svc/NfcCommand.java

    frameworks/base/cmds/svc/src/com/android/commands/svc/UsbCommand.java

    frameworks/base/cmds/telecom/src/com/android/commands/telecom/Telecom.java 

    frameworks/base/services/java/com/android/server/SystemServer.java

Here we focus on `SystemServer` , as it create the `ActivityThread` and prepare the main `Looper` in the process of launching an application and creating the associated main thread, in the `SystemServer#run()`.

So the main thread of an application, and its main looper are created by the SystemServer, within the same process of the SystemServer, by `ActivityThread.systemMain()` and `Looper.prepareMainLooper()` . 

- This is the source of the main looper. The main looper is stored using `ThreadLocal` , as a local variable in the associated thread with type `Looper`.

- The main thread `mUiThread` in an activity is specified in `Activity#attach()`, called by `ActivityThread`, who manages lifecycle of activities. (While there is a `mMainThread` variable with `ActivityThread` type declared in `Activity`, do not get confused for the name, as the main thread `mUiThread` is a variable with `Thread` type.)

I got some beautiful figures to explain.

![The process from launching an app to create UI thread and main Looper, where SystemServer and ActivityThread are important roles to know.](Main%20Thread%20Review%20c093a0921c4b46a6b6e2ade028c5a5ab/%25E6%2588%25AA%25E5%25B1%258F_2024-07-17_16.12.28.jpeg)

*The process from launching an app to create UI thread and main Looper, where SystemServer and ActivityThread are important roles to know.*

![How to update UI from other thread using Handler mechanism.](Main%20Thread%20Review%20c093a0921c4b46a6b6e2ade028c5a5ab/Untitled.png)

*How to update UI from other thread using Handler mechanism.*

![Methods to execute something in a background thread.](Main%20Thread%20Review%20c093a0921c4b46a6b6e2ade028c5a5ab/%25E6%2588%25AA%25E5%25B1%258F_2024-07-17_16.15.56.jpeg)

*Methods to execute something in a background thread.*