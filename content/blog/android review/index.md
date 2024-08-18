+++
title = "Android Review"
date = 2024-07-10
[taxonomies]
  tags = ["note", "Android"]

[extra]
  toc = true
+++


- Backbone tutorial 🧭
    developer.android, [Platform Architecture](https://developer.android.com/guide/platform)

    developer.android, [Application fundamentals](https://developer.android.com/guide/components/fundamentals#Components)

    source.android, AOSP (Android Open System Platform), [Architecture Overview](https://source.android.com/docs/core/architecture)

    developer.android, [Guide to app architecture](https://developer.android.com/topic/architecture)

- *Android Code Search (*[https://cs.android.com/](https://cs.android.com/))

- Further readings 🐠

    Build MVP apps: MVP Part I, [a 2010 blog](https://www.gwtproject.org/articles/mvp-architecture.html#presenter)

    Demo see here : [android/platform/development - main](https://android.googlesource.com/platform/development/+/refs/heads/main)

# What types of process would be killed by system

*Ref:* 

*- Process and app lifecycle, [developer.android](https://developer.android.com/guide/topics/processes/process-lifecycle)*

*- Persistent work, [developer.android](https://developer.android.com/develop/background-work/background-tasks/persistent)*

*- Background tasks overview, [developer.android](https://developer.android.com/develop/background-work/background-tasks#choose-right-option)*

To determine which processes to kill and reclaim memory when the system is low on memory, there are some types to distinguish their level of importance: (from most important to less)

A foreground process;

A visible process;

A service process <30mins;

————————————

A cached process. 🚬

# Why WorkManager

Use WorkManager to support long-running workers for persistent work, which can run longer than 10 mins.
And use WorkManager to do reliable works, in cases like even if the users navigates off a screen, the app exists, or the device restarts.

- for Kotlin, use CoroutineWorker

- for Java, use Worker or ListenableWorker

    - Relationship to other APIs.
    Do not use these easy-getting-confused APIs to do persistent work.

|  |  |
| --- | --- |
| Kotlin Coroutines <br/> (Asynchronous work, others eg Java Thread) | This is a concurrency framework, not for persistent work. <br/> They are the standard means of leaving the main thread in Kotlin. They leave memory once the app closes, thus is not guaranteed to finish. |
| AlarmManager | This would wakes device from Doze mode. Not efficient in terms of power and resource management. Do not use it for background work. |
| Foreground services | This can put potentially heavy load on device and sometimes have privacy and security implications, thus get many restrictions from the system. <br/> WorkManager provides some convenient APIs to make it simpler to create a foreground service. [See here](https://developer.android.com/develop/background-work/background-tasks#foreground-services) |

- WorkManager is recommended to replace some deprecated APIs for Android background scheduling: 
FirebaseJobDispatcher, GcmNetworkManager, JobScheduler.

- Code samples for WorkManager:

- googlecodelabs/android-workmanager@github

- android/nowinandroid@github

- When considering whether to use foreground services or WorkManager API, WorkManager would be recommended in most cases as it is able to run persistent jobs as foreground services if needed. [[See here](https://developer.android.com/develop/background-work/services#Types-of-services)] 

    Also in some cases considering whether to use background services or WorkManager API, WorkManager is preferable in cases like accessing location information from the background, as there are some restrictions on running background services. [[See here](https://developer.android.com/about/versions/oreo/background)]

- WorkManager’s built-in threading interoperability provides Java Worker / Kotlin CoroutineWorker / RxJava RxWorker / their based class Java ListenableWorker, so to do threading in a work.

# The Components

## Services

- Normal services: run in foreground or background.
- Bound services: client-server IPC.

    i) Create a bound service

    ii) Add binding to a started service

- Lifecycle
    
    ![*Ref: developer.android, [Services overview - Managing the lifecycle of a service](https://developer.android.com/develop/background-work/services#LifecycleCallbacks)*](Android%20Review%207c25a3c65dff4ef295adbbc9a644b372/Untitled.png)
    
    *Ref: developer.android, [Services overview - Managing the lifecycle of a service](https://developer.android.com/develop/background-work/services#LifecycleCallbacks)*
    

### Create a started service

🍤 This kind of service must be stopped manually.

- A Sample of extending the `Service` class to handle each incoming intent by `Handler`.
*Ref: [Services overview - Creating a started service](https://developer.android.com/develop/background-work/services#ExtendingService)*
    
    ```kotlin
    // The Service implements a Handler to handle each Intent obtained in
    // the onStartCommand() 
    class HelloService : Service() {
    
        private var serviceLooper: Looper? = null
        private var serviceHandler: ServiceHandler? = null
    
        // Handler that receives messages from the thread
        private inner class ServiceHandler(looper: Looper) : Handler(looper) {
    
            override fun handleMessage(msg: Message) {
                // Normally we would do some work here, like download a file.
                // For our sample, we just sleep for 5 seconds.
                try {
                    Thread.sleep(5000)
                } catch (e: InterruptedException) {
                    // Restore interrupt status.
                    Thread.currentThread().interrupt()
                }
    
                // Stop the service using the startId, so that we don't stop
                // the service in the middle of handling another job
                stopSelf(msg.arg1)
            }
        }
    
        override fun onCreate() {
            // Start up the thread running the service.  Note that we create a
            // separate thread because the service normally runs in the process's
            // main thread, which we don't want to block.  We also make it
            // background priority so CPU-intensive work will not disrupt our UI.
            HandlerThread("ServiceStartArguments", Process.THREAD_PRIORITY_BACKGROUND).apply {
                start()
    
                // Get the HandlerThread's Looper and use it for our Handler
                serviceLooper = looper
                serviceHandler = ServiceHandler(looper)
            }
        }
    
        override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
            Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()
    
            // For each start request, send a message to start a job and deliver the
            // start ID so we know which request we're stopping when we finish the job
            serviceHandler?.obtainMessage()?.also { msg ->
                msg.arg1 = startId
                serviceHandler?.sendMessage(msg)
            }
    
            // If we get killed, after returning from here, restart
            return START_STICKY
        }
    
        override fun onBind(intent: Intent): IBinder? {
            // We don't provide binding, so return null
            return null
        }
    
        override fun onDestroy() {
            Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show()
        }
    }
    
    // the Client, an activity, just start the Service as usual.
    // Context#startService()
    startService(Intent(this, HelloService::class.java))
    ```
    
- Do rsemember to stop the service manually.
    - by `Service.stopSelf()` 
    - by `Context.stopService()`

### Create a bound service and bind to a service

🍤 This kind of service can be stopped by the system once no more binding clients.

i) Extend the `Binder` class.

when the service need to be accessed across separated processes:

ii) Use a `Messenger`.
  This does not require thread-safe.

when the service need to be accessed across separated processes and handle them simultaneously:

iii) Use AIDL.
  This requires thread-safe.

- A Sample of extending the `Binder` class.
*Ref: [Bound services overview - Create a bound service](https://developer.android.com/develop/background-work/services/bound-services#Binder)*
    
    ```kotlin
    // The Service creates an instance of Binder, 
    // and the Binder is returned by the onBind().
    class LocalService : Service() {
        // Binder given to clients.
        private val binder = LocalBinder()
    
        // Random number generator.
        private val mGenerator = Random()
    
        /** Method for clients.  */
        val randomNumber: Int
            get() = mGenerator.nextInt(100)
    
        /**
         * Class used for the client Binder. Because we know this service always
         * runs in the same process as its clients, we don't need to deal with IPC.
         */
        inner class LocalBinder : Binder() {
            // Return this instance of LocalService so clients can call public methods.
            fun getService(): LocalService = this@LocalService
        }
    
        override fun onBind(intent: Intent): IBinder {
            return binder
        }
    }
    
    // The Client receive the Binder from onServiceConnected() callback,
    // and use the public methods exposed by Service.
    class BindingActivity : Activity() {
        private lateinit var mService: LocalService
        private var mBound: Boolean = false
    
        /** Defines callbacks for service binding, passed to bindService().  */
        private val connection = object : ServiceConnection {
    
            override fun onServiceConnected(className: ComponentName, service: IBinder) {
                // We've bound to LocalService, cast the IBinder and get LocalService instance.
                val binder = service as LocalService.LocalBinder
                mService = binder.getService()
                mBound = true
            }
    
            override fun onServiceDisconnected(arg0: ComponentName) {
                mBound = false
            }
        }
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.main)
        }
    
        override fun onStart() {
            super.onStart()
            // Bind to LocalService.
            Intent(this, LocalService::class.java).also { intent ->
                bindService(intent, connection, Context.BIND_AUTO_CREATE)
            }
        }
    
        override fun onStop() {
            super.onStop()
            unbindService(connection)
            mBound = false
        }
    
        /** Called when a button is clicked (the button in the layout file attaches to
         * this method with the android:onClick attribute).  */
        fun onButtonClick(v: View) {
            if (mBound) {
                // Call a method from the LocalService.
                // However, if this call is something that might hang, then put this request
                // in a separate thread to avoid slowing down the activity performance.
                val num: Int = mService.randomNumber
                Toast.makeText(this, "number: $num", Toast.LENGTH_SHORT).show()
            }
        }
    }
    ```
    
- Sample of one-way messaging using a `Messager`:
*Ref: [Bound services overview - Create a bound service](https://developer.android.com/develop/background-work/services/bound-services#Messenger)*
    
    ```kotlin
    /** Command to the service to display a message.  */
    private const val MSG_SAY_HELLO = 1
    
    // The Service uses Handler to handle Message send from Client.
    class MessengerService : Service() {
    
        /**
         * Target we publish for clients to send messages to IncomingHandler.
         */
        private lateinit var mMessenger: Messenger
    
        /**
         * Handler of incoming messages from clients.
         */
        internal class IncomingHandler(
                context: Context,
                private val applicationContext: Context = context.applicationContext
        ) : Handler() {
            override fun handleMessage(msg: Message) {
                when (msg.what) {
                    MSG_SAY_HELLO ->
                        Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show()
                    else -> super.handleMessage(msg)
                }
            }
        }
    
        /**
         * When binding to the service, we return an interface to our messenger
         * for sending messages to the service.
         */
        override fun onBind(intent: Intent): IBinder? {
            Toast.makeText(applicationContext, "binding", Toast.LENGTH_SHORT).show()
            mMessenger = Messenger(IncomingHandler(this))
            return mMessenger.binder
        }
    }
    
    // The Client creates a Message and sends to the Service.
    class ActivityMessenger : Activity() {
        /** Messenger for communicating with the service.  */
        private var mService: Messenger? = null
    
        /** Flag indicating whether we have called bind on the service.  */
        private var bound: Boolean = false
    
        /**
         * Class for interacting with the main interface of the service.
         */
        private val mConnection = object : ServiceConnection {
    
            override fun onServiceConnected(className: ComponentName, service: IBinder) {
                // This is called when the connection with the service has been
                // established, giving us the object we can use to
                // interact with the service.  We are communicating with the
                // service using a Messenger, so here we get a client-side
                // representation of that from the raw IBinder object.
                mService = Messenger(service)
                bound = true
            }
    
            override fun onServiceDisconnected(className: ComponentName) {
                // This is called when the connection with the service has been
                // unexpectedly disconnected&mdash;that is, its process crashed.
                mService = null
                bound = false
            }
        }
    
        fun sayHello(v: View) {
            if (!bound) return
            // Create and send a message to the service, using a supported 'what' value.
            val msg: Message = Message.obtain(null, MSG_SAY_HELLO, 0, 0)
            try {
                mService?.send(msg)
            } catch (e: RemoteException) {
                e.printStackTrace()
            }
    
        }
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.main)
        }
    
        override fun onStart() {
            super.onStart()
            // Bind to the service.
            Intent(this, MessengerService::class.java).also { intent ->
                bindService(intent, mConnection, Context.BIND_AUTO_CREATE)
            }
        }
    
        override fun onStop() {
            super.onStop()
            // Unbind from the service.
            if (bound) {
                unbindService(mConnection)
                bound = false
            }
        }
    }
    ```
    
- Sample of two-way messaging:
*[MessageService.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java) (service);*
*[MessageServiceActivities.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java) (client).*
- More samples see here: [ApiDemos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos)

## Broadcasts

BroadcastReceivers are context-registered receivers, ie they would be active as long as their registering context is valid, eg registering within an Activity context or an Application context.

## Activities

- an Activity’s lifecycle. [see [developer.android](https://developer.android.com/guide/components/activities/activity-lifecycle#alc), The activity lifecycle]
- a Fragment’s lifecycle. [see [developer.android](https://developer.android.com/guide/fragments/lifecycle#states), Fragment lifecycle]

## ContentProvider

Persistent data storage with data sharing across processes. This is more like a manage container, if comparing with Room. See more [here](https://developer.android.com/training/data-storage) (*developer.android, Data and file storage overview)*. 

# App Resources

## Quick note of AAPT2

*Ref: developer android, [AAPT2](https://developer.android.com/tools/aapt2)*

AAPT2 (Android Asset Packaging Tool) is a build tool that Android Studio & Android Gradle Plugin use to compile and package your app’s resources.
It supports faster compilation of resources by enabling incremental compilation, where resource processing is separated into two steps:
- compile: resource files (xml files, others) → binary files (*.arsc.flat, *.flat). These are intermediate files.
- link: merge compiled files, packages them into one.
When one file needs change, recompile that one only, and link again.

## Quick note for handling large bitmaps

Other than using Glide to load resource efficiently, some methods below are worth to try:
(*Ref: [Handling bitmaps](https://developer.android.com/develop/ui/views/graphics), developer.android*)

- Use a smaller size to decoding a large bitmap (for loading into memory), when it will show in a target container having smaller size. ([*Loading large bitmaps efficiently*](https://developer.android.com/topic/performance/graphics/load-bitmap))

    i) read bitmap original dimension to see if it is really a large one ready to decode, by using `BitmapFactory.Options().inJustDecodeBounds = true`;

    ii) load a scaled down version of it into memory.

- Use cache to load bitmaps helps frequent loading to be efficient. ([*Caching bitmaps*](https://developer.android.com/topic/performance/graphics/cache-bitmap))
    - consider memory cache, eg by `LruCache` ;
    - consider disk cache, eg by `DiskLruCache`(use `Writer` inside).
    
    Remember to handle in a thread-safe way, though the classes above are thread-safe, reminding for associated UI update.
- Manage the memory cache of bitmaps efficiently. ([*Managing bitmap memory*](https://developer.android.com/topic/performance/graphics/manage-memory))
    - on ≤ Android 2.3.3 (api level 10), use `Bitmap.recycle()` promptly.
    - on ≥ Android 3.0 (api level 11), use `BitmapFactory.Options.inBitmap=true` properly, also together with `BitmapFactory.Options.inMutable=true` as required by reusing an inBitmap-available bitmap.
    
    This option setting allows BitmapFactory to load a bitmap by reusing an existing bitmap’s memory, if their sizes match requirement.

- Note on reducing image download sizes.
    
    *Reference: [developer.android](https://developer.android.com/develop/ui/views/graphics/reduce-image-sizes)*
    

    ![Deciding on a compression scheme.](./Android%20Review%207c25a3c65dff4ef295adbbc9a644b372/image_res_compression_scheme.png)

    *Deciding on a compression scheme.*

## Note for Android drawing models

*Ref: Fix custom-drawing issues, [developer.android](https://developer.android.com/develop/ui/views/graphics/hardware-accel#model)*

- Software-based drawing model, which is without hardware-accelerated. Drawing views by this method includes:
    
    i) invalidate the hierarchy;
    ii) draw the hierarchy.

    UI is updated by invoking `View.invalidate()`. Always call `View.incalidate()` on custom views when associated data changes probably bringing content changes on them.
- Hardware accelerated drawing model. This method contains:

    i) invalidate the hierarchy;
    ii) record & update display lists, containing output of view hierarchy’s drawing code;
    iii) draw the display lists.

Hardware acceleration (enabled by default for target api level ≥14) can be turned off on the level of application / activity / window / view.

Hardware acceleration would surely have optimal invalidation in changing these properties of a view by not redrawing the target view:
alpha; <position> x, y, translationX, translationY; <size> scaleX, scaleY; <orientation> rotation, rotationX, rotationY; <transformation origin> pivotX, pivotY.

It is even better for memory efficiency to disable hardware layers when properties changes (animations) are ended.

### Tips & tricks for using GPU effectively

*Ref: [Fix custom-drawing issues.](https://developer.android.com/develop/ui/views/graphics/hardware-accel#tips)*

- Reduce the number of views in application. 
Good for software rendering pipeline as well.
- Avoid overdraw. Less layers on top of others. 
Typically, do not draw 2.5x more of the pixels on screen per frame (transparent pixels in a bitmap count).
- Don’s create render objects (eg `new Paint` or `new Path`) in draw methods.
This would make GC tired, and ignore caches & optimizations in hardware pipeline.
- Don’t modify shapes too often, because complex shapes/paths/circles/etc are rendered using new texture masks created by hardware pipeline, nothing but expensive.
- Don’t modify bitmaps too often, since every change of content in a bitmap would be uploaded again as a GPU texture for next-time drawing.
- Use alpha with care, recommended with setting view’s layer type to `LAYER_TYPE_HARDWARE`, since a translucent view (`setAlpha()`, `AlphaAnimation`, `ObjectAnimator`) is rendered in an off-screen buffer, or layer, doubling the required fill-rate.

## Note for memory management

*Ref: Overview of memory management, [developer.android](https://developer.android.com/topic/performance/memory-overview)*

*Ref: Tool of Investigating RAM usage guide in android studio (Inspect your app’s memory usage with Memory Profiler, [developer.android](https://developer.android.com/studio/profile/memory-profiler#overview). )*

*Ref: Tool of Inspect app memory from command line - dumpsys (dumpsys, [developer.android](https://developer.android.com/tools/dumpsys#ViewingAllocations) )*

*Ref: Tool of View GC events in logcat (View logs with logcat, [developer.android](https://developer.android.com/studio/debug/logcat#memory-logs) )*

*Ref: Tips for programming practices to reduce app’s memory use (Manage your app’s memory, [developer.android](https://developer.android.com/topic/performance/memory))*

*Ref: Garbage Collection, explained ([wiki](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)))*

*Ref: Debug ART garbage collection ([source.android](https://source.android.com/docs/core/runtime/gc-debug))*

- PSS (proportional set size): a measurement of app’s RAM use, taking into account the sharing pages across processes.
- Look for memory information of an app:
    - Use Profiler in Android studio
    - Use dumpsys in command line
        
        ```bash
        $ adb shell ps -p|grep com.google.mlkit.vision.demo 
        $ adb shell dumpsys meminfo  com.google.mlkit.vision.demo -d
        ```
        

### Tips for managing app’s memory

- Monitor available memory & memory usage
    - Release memory in response to events, by implementing `ComponentCallbacks2` in `Activity`, where implementing `onTrimMemory()` callback would respond to GC memory reclaiming with level of memory-related events.
    - Check how much memory the app need, by querying from `ActivityManager.getMemoryInfo()`.
- Use more memory-efficient code structures
    - Use service sparsely, try `WorkManager` to schedule background processes.
    - Use mobile-optimized data containers, eg instead of `HashMap`, use `SparseArray`.
    - Be careful with code abstractions, when inheriting deeper.
    - Use lite protobufs (lite Protocol buffers) for serialized data.
    - Avoid memory churn.
        
        Memory churn is used to describe when the GC is triggered often due to frequent allocation & reclaim of temporary objects within the app. It could be measured by the number of allocated temporary objects occurring in a given amount of time.
        Possible solutions include factory-method-based allocation structure, or an object pool (this pool is a trade-off between the number of GC invocations & the amount of work needed for each invocation).
- Remove memory-intensive resources & libraries
    - Reduce overall APK size, including resources and external dependencies.
    - Use Hilt or Dagger2 (static compile-time implementation) rather than reflection initialize processes (in running-time), for dependency injection.
    - Be careful about using external libraries, especially for using some simple classes having wide swaths of dependencies.

## Who will reclaim the memory in low memory situations?

- Low-memory killer Daemon ([LMKD](https://android.googlesource.com/platform/system/memory/lmkd/+/master/)).
System will notify the app by `onTrimMemory()` callback.
There is an out-of-memory score named `oom_adj_score` (higher score, shorter live, lower priority to keep) for lkmd to decide which process to kill.

![Ref: [Memory allocation among processes, Low-memory killer](https://developer.android.com/topic/performance/memory-management#low-memory_killer). Top one with higher score would be killed first by lmkd.](Android%20Review%207c25a3c65dff4ef295adbbc9a644b372/Untitled%201.png)

*Ref: [Memory allocation among processes, Low-memory killer](https://developer.android.com/topic/performance/memory-management#low-memory_killer). Top one with higher score would be killed first by lmkd.*

- There is another way to manage low memory, called kswapd (kernel swap daemon).
It is a very low-level processes as in the native of system. It reclaims cached pages.

![Ref: [Memory allocation among processes](https://developer.android.com/topic/performance/memory-management#memory_pages). kswapd manipulates on pages to reclaim memory.](Android%20Review%207c25a3c65dff4ef295adbbc9a644b372/Untitled%202.png)

*Ref: [Memory allocation among processes](https://developer.android.com/topic/performance/memory-management#memory_pages). kswapd manipulates on pages to reclaim memory.*

## Note for saving UI states
*Reference: Save UI states, [developer.android](https://developer.android.com/topic/libraries/architecture/saving-states)*

*Other References: See [Handling the configuration change yourself](https://developer.android.com/guide/topics/resources/runtime-changes#HandlingTheChange) . `DataStore` is a modern data storage solution that you should use instead of `SharedPreferences`. Read the [DataStore guide](https://developer.android.com/topic/libraries/architecture/datastore) for more information. To learn how to implement saved instance state using `onSaveInstanceState`, see Saving and restoring transient UI state in the [Activity Lifecycle guide](https://developer.android.com/guide/components/activities/activity-lifecycle#asem).*

*Prerequisite: storage (local & persistent) vs memory (within the app).*

i) `ViewModel`. Use it to handle configuration changes.

ii) Saved instance states. Use it as backup to handle system-initiated process death.

    - Jetpack Compose `rememberSaveable`
    - Views  `onSaveInstanceState()`
    - ViewModels `SavedStateHandle` 

iii) Local persistence for non-transient needs. Use it to handle process death for complex or large data.

![Options for preserving UI state.](./Android%20Review%207c25a3c65dff4ef295adbbc9a644b372/image_table_storage.png)


*Reference: [Options for preserving UI state.](https://developer.android.com/topic/libraries/architecture/saving-states#options)* 

***Key Point:** [Saved instance state] APIs only save data written to it when the **`Activity`** is stopped. Writing into it in between this lifecycle state defers the save operation till the next stopped lifecycle event.*

***Important:** The **`SavedStateHandle`** only saves data written to it when the **`Activity`** is stopped. Writes to **`SavedStateHandle`** while the **`Activity`** is stopped aren't saved unless the **`Activity`** receives **`onStart`** followed by **`onStop`** again.*

***Note:** In order for the Android system to restore the state of the views in your activity, each view must have a unique ID, supplied by the **`android:id`** attribute.*

***Note: `onSaveInstanceState()`** is not called when the user explicitly closes the activity or in other cases when **`finish()`** is called.*