+++
title = "Executing in Background in Java, Kotlin, Android"
date = 2024-07-22
[taxonomies]
  tags = ["Android", "Note", "Multi-Threads", "Kotlin"]

[extra]
  toc = true
+++

There are some recommendations about the methods chosen for executing a background task.

*Ref:  Background Task Overview, [developer.android](https://developer.android.com/develop/background-work/background-tasks#background-work).*

<!-- 
![For a user-initiated background task, consider whether the task should be independent from the app process visibility.](https://developer.android.com/static/images/develop/background-work/background-tasks/index/user-tasks-flowchart.svg)

*For a user-initiated background task, consider whether the task should be independent from the app process visibility.*

![For an event-triggered background task, consider how much time the task would take.](https://developer.android.com/static/images/develop/background-work/background-tasks/index/event-tasks-flowchart.svg)

*For an event-triggered background task, consider how much time the task would take.* -->

<div style="display: flex;">
    <div style="flex: 50%;">
        <figure>
            <img src="https://developer.android.com/static/images/develop/background-work/background-tasks/index/user-tasks-flowchart.svg" alt="For a user-initiated background task, consider whether the task should be independent from the app process visibility.">
            <figcaption>For a user-initiated background task, consider whether the task should be independent from the app process visibility.</figcaption>
        </figure>
    </div>
        <div style="flex: 50%;">
        <figure>
            <img src="https://developer.android.com/static/images/develop/background-work/background-tasks/index/event-tasks-flowchart.svg" alt="For an event-triggered background task, consider how much time the task would take.">
            <figcaption>For an event-triggered background task, consider how much time the task would take.</figcaption>
        </figure>
    </div>
</div>

Besides methods decision, it is necessary to clear the difference between a thread and a coroutine. In one word, a thread can hold multiple coroutines, while a process can hold multiple threads.

# Threads in Java

For single simple one, extend `Thread` and start it, or implement `Runnable` and put it in a thread and start.

For multiple threads, typically, create a `ThreadPoolExecutor` to manage multiple threads, or use its factory `Executors` to create one.

A `Worker` created in a thread pool is a `Runnable` , it is put into a thread when executing and a `ReentrantLock` is used to manage the workers. Specifically, the `ReentrantLock` is used to protect the manipulation of a `HashSet<Worker>` maintain in the thread pool.

*Reference: [Java线程池详解-线程池原理分析](https://javaguide.cn/java/concurrent/java-thread-pool-summary.html#%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90-%E9%87%8D%E8%A6%81).*

- More readings:
    
    [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
    
    [JUC线程池：ThreadPoolExecutor详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html)
    
    [JUC线程池：BlockingQueue详解 - 同步队列 SynchronousQueue](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-BlockingQueue.html#%E5%90%8C%E6%AD%A5%E9%98%9F%E5%88%97-synchronousqueue), it is used in `Executors.newCachedThreadPool()` .
    
    [JUC线程池：ScheduledThreadPoolExecutor详解 - 数据结构](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ScheduledThreadPoolExecutor.html#scheduledthreadpoolexecutor%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84), about the `DelayedWorkQueue` used in `Executors.newScheduledThreadPool()`.
    
    [基本功｜一文讲清多线程和多线程同步](https://tech.meituan.com/2024/07/19/multi-threading-and-multi-thread-synchronization.html)

    Package summary: java.util.concurrent, [developer.android](https://developer.android.com/reference/java/util/concurrent/package-summary)
    

# Coroutines in Kotlin

Kotlin Features *and Reference (https://developer.android.com/kotlin/coroutines)*

- Suspension
- Structured concurrency with constrained coroutine scope, fewer memory leaks by automatically cancel
- Cancellation support

Coroutines execution are scheduled on multiple threads internally, internal implementation is sort of like a nice-wrapping executor service without bothering of creating and closing alike one. While `Dispatchers` provide some thread-level control before scheduling and executing a coroutine by specifying `Default or IO, Main`, guaranteeing some [main-safe executing](https://developer.android.com/kotlin/coroutines/coroutines-adv#main-safety).

## Basic usage

- use `CoroutineScope.launch{}`

```kotlin
viewModelScope.launch(Dispatchers.IO) {
	// some blocking operations
	val jsonBody = “{xxx xxx}”
	loginRepository.makeLoginRequest(jsonBody)
}
```

viewModelScope::a CoroutineScope
launch::create a coroutine on [Dispatchers.IO](http://dispatchers.io/) if specified, or else on UI thread
[Dispatchers.IO](http://dispatchers.io/)::indicator of IO thread (type)

- use `suspend function` and `withContext{}`

```kotlin
 				// Create a new coroutine on the UI thread
        viewModelScope.launch {
            val jsonBody = “{xxx xxx}”

						// Make the network call and suspend execution until it finishes
						// suspend here
            val result = loginRepository.makeLoginRequest(jsonBody)

            // Display result of the network request to the user
            when (result) {
                is Result.Success<LoginResponse> -> // Happy path
                else -> // Show error in UI
            }
        }

		    suspend fun makeLoginRequest(
		        jsonBody: String
		    ): Result<LoginResponse> {
		
		        // Move the execution of the coroutine to the I/O dispatcher
		        return withContext(Dispatchers.IO) {
		            // Blocking network request code
		        }
		    }
```

## Readings

- How does a Kotlin coroutine get to execute when using a suspend function, from a perspective of de-wrapping the simple form of coroutine? See here ([图解Kotlin协程原理](https://blog.xlxs.top/archives/kotlin-coroutine#id-795988185)). 

  → It is transformed in to a `Continuation` using CPS (continuation pass-style transform) by the compiler and executed as a state machine (a `switch-case` ).

- How to specified a thread for coroutines to run on? See here ([Coroutines support: Force execution on certain thread](https://discuss.kotlinlang.org/t/coroutines-force-execution-on-certain-thread/15766)). 

  → Use `withContext(Handler.asCoroutineDispatcher()) { doXXX() }`.

- How to make coroutines run on single thread? See here ([Coroutines support: Coroutine dispatcher confined to a single thread](https://discuss.kotlinlang.org/t/coroutine-dispatcher-confined-to-a-single-thread/17978)). 

  → Use `Executors.newSingleThreadExecutor().asCoroutineDispatcher()`.
      
  → Also, ⚠️ do not use `runBlocking` inside a coroutine, especially inside default or IO dispatcher, as it may lead to dead lock for the thread pool inside coroutines’ internal implementation. Specifically, `runBlocking` would block the current thread executing the coroutine, which holds the `runBlocking` (See `runBlocking` source code comment, and here: [How I Fell in Kotlin’s RunBlocking Deadlock Trap, and How you can avoid it](https://betterprogramming.pub/how-i-fell-in-kotlins-runblocking-deadlock-trap-and-how-you-can-avoid-it-db9e7c4909f1)). For the case in above context, do something like this would be fine: ([code snip from here](https://discuss.kotlinlang.org/t/coroutine-dispatcher-confined-to-a-single-thread/17978/10))

  ```kotlin
  val singleDispatcher = Executors.newSigleThreadExecutor().asCoroutineDispatcher()
  // ...
  launch(Dispatchers.Default){
    // do something multithreaded...
    val res = withContext(singleDispatcher){
      // do somthething on a single thread...
    }//return to Default
  }
  ```

  A same opinion comes from this reading ([How to Avoid the Kotlin Coroutine Deadlocks?](https://www.alibabacloud.com/blog/601215?spm=a3c0i.23458820.2359477120.4.7e177d3fizwcf7)).

      The IO dispatcher has a default size of core threads with value 64. 

      The default dispatcher has a default size of core threads with value of the number of CPU cores (available processors), having a least bound of 2.