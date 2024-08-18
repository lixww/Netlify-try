+++
title = "JVM and ART"
date = 2024-08-15
[taxonomies]
  tags = ["note", "Android", "JVM", "ART", "GC", "Memory"]

[extra]
  toc = true
+++

*Java Virtual Machine (JVM).* 

*Dalvik → Android Runtime (ART).*

<div style="display: flex;">
    <div style="flex: 50%;">
        <figure>
            <img src="JVM%20and%20ART%2020ee4621a9a6473ab0bb4c0f83f29a37/%25E5%259B%25BE%25E5%2583%258F.png" alt="The structure of JVM.">
            <figcaption>The structure of JVM.</figcaption>
        </figure>
    </div>
    <div style="flex: 50%;">
        <figure>
            <img src="JVM%20and%20ART%2020ee4621a9a6473ab0bb4c0f83f29a37/%25E5%259B%25BE%25E5%2583%258F%201.png" alt="GC in JVM, vs GC in ART.">
            <figcaption>GC in JVM, vs GC in ART.</figcaption>
        </figure>
    </div>
</div>

# Garbage Collection

How it works? What optimizations? What impacts on performance? Tools for analysis?

| Concepts in JVM (Hotspot implementation) | Concepts in Android VM (ART) |
| --- | --- |
| generational GC heap, ie young, old, and permanent | generational GC heap, ie young and old  |
|  | concurrent copying (improving GC, reducing background memory usage & fragmentation, see Linux Memory Management) |
|  | read barrier (improvement in GC implementation detail) |
|  | Reference: Android 8.0 ART improvements, https://source.android.com/docs/core/runtime/improvements |

Typical process on heap:

Allocation for objects → generational GC triggered → reclaim unused allocation.

- Where does GC happen?
    
    Mainly on heap.
    
- What is the priority of allocation places?
    
    Normal objects go to young generation (of heap). Big objects go to old generation. Long-live objects will go to old generation.
    
- How to mark one as garbage, or say decide one need to be reclaim for it is dying?
    
    1) Reference counting, count the references of objects, easy but cycle could exist.
    2) Reachable tracing, trace the reachable objects through a chain of references. The roots of the chain might be objects referenced in local stacks of threads, native method stack, static and constants in method area, synchronized locks, and JNI.
    
- How to reclaim the memory of garbages?
    
    Algorithms like i) mark-and-sweep, ii) copying then sweep, iii) mark-and-compact, iv) generational collection.
    

*Reference: [JVM垃圾回收详解](https://javaguide.cn/java/jvm/jvm-garbage-collection.html)，[Java内存区域详解](https://javaguide.cn/java/jvm/memory-area.html)，[类加载过程](https://javaguide.cn/java/jvm/class-loading-process.html#%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B)*

*Reference: [JVM Specification (SE22) - Chapter 5 Loading, Linking, and Initializing](https://docs.oracle.com/javase/specs/jvms/se22/html/jvms-5.html)*

*Reference: Overview of Memory Management, [developer.android](https://developer.android.com/topic/performance/memory-overview?hl=en)*

# Memory Management

*Reference: Manage your app’s memory, [developer.android](https://developer.android.com/topic/performance/memory?hl=en#code).*

*Reference: [内存管理（Linux）](https://xiaolincoding.com/os/3_memory/vmem.html)，[操作系统常见面试题总结-内存管理](https://javaguide.cn/cs-basics/operating-system/operating-system-basic-questions-02.html#%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86)，[JMM（Java内存模型）详解](https://javaguide.cn/java/concurrent/jmm.html#java-%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E5%92%8C-jmm-%E6%9C%89%E4%BD%95%E5%8C%BA%E5%88%AB)*


🫑 Some memory-efficient code suggestions, from [developer.android](https://developer.android.com/topic/performance/memory?hl=en#code).

- Don’t leave unnecessary services running, use them sparingly. Consider `WorkManager` to schedule background processes and persistent works.

- Use data containers optimized for mobile devices, eg `HashMap` → `SparseArray`.

- Thought abstraction is good for flexibility and reuse? They are costly to be executed too.

- Lite protobufs (protocol buffers) would be good on client-side, as they are designed for serializing structural data. Notice regular protobufs would often get verbose.

- Memory churn would cause frequent garbage collection, and then would drain the battery faster. It is the number of allocated temporary objects in a given amount of time. Eg create heavy objects like `Bitmap` or `Paint` in a `for` loop or in `onDraw()` of a view might consume the available memory in young generation quickly. 

    Use Memory Profiler ([*Investigating Your RAM Usage*](https://developer.android.com/studio/profile/investigate-ram)) to inspect the code where remains high memory churn. Consider moving things out of inner loops or moving them into a factory-based allocation structure (*Factory method pattern)*. In some cases, an object pool may help. Evaluate its allocations-savings and other overheads (eg synchronization, lifecycle, capacity) thoroughly to determine if it is really helpful.
