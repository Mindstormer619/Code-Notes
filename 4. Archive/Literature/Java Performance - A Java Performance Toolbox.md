# Java Performance - A Java Performance Toolbox
> Created: 2022-06-01 23:20
> #book 

## Operating System Tools

On Unix systems, you have `sar` (System Accounting Report), `vmstat`, `iostat`, `prstat` etc. On Windows, there are graphical resource monitors, and command-line utils like `typeperf`.

### CPU Usage

**User time:** The time that the CPU executes application code.
**System time (Privileged Time on Windows):** The time that the CPU executes kernel code.

System time is still related to the application — if the app performs I/O, the kernel is executing code to read files from the disk etc. Any time you use an underlying system resource, you are increasing system time spent.

The *higher* the CPU usage, the better. You want the CPU to be as utilized as possible for as short a time as possible. If you can tune the code to use double the CPU but execute in half the time, that's better performance.

CPUs generally have idle time in one of these situations:
1. It's waiting on something like a synchronization lock.
2. It's waiting for a response to come back, maybe from the database or network.
3. It genuinely has nothing to do.

Generally speaking, you're trying to minimize the third bit by optimizing CPU usage of your application. For the first two, you can improve them by reducing contention on threaded locks, or improving the database response time.

#### Java - Single CPU Usage

Driving the CPU usage higher is *always* the goal for batch processing jobs -- because if it's idle, it just means it is running slower than it could be.

For server-style applications (receive request, process it and send response back), the CPU utilization is also expected to hit 100%, except in very short bursts. The average utilization is lower than 100% because for most of the time the server is generally waiting for new requests to come in -- it is idle because it genuinely has no work.

In this case you can still drive up overall CPU utilization by directing more requests to this server. 

#### Java - Multi CPU Usage

In the case of multiple threads, we have a case where the CPU can be idle even when there's work to do, and that's when there are no available threads to service the task. In this case, we generally need to increase the thread pool size.

However, just because there's idle time despite pending work doesn't mean the thread pool needs to be increased -- it could be because of synchronization bottlenecks, or waiting for external resources. 

### The CPU Run Queue

The run queue or processor queue is the number of unblocked (available) threads -- these are not blocked on I/O, sleeping etc. `vmstat` or `typeperf` can give this information.

On Unix, it's the number of threads that *are running* currently or *could run* if there were an available CPU.

On Windows, it does not include the number of threads that are currently running, only the number of threads that are waiting to run (available but not currently running).

In general, you want the run queue length to be as small as possible (0 on Windows, equal to number of CPUs on Linux). This means that you don't have available threads which are looking for CPUs to run on. If the run queue length is too high for extended periods, it's a good sign that you need to move workload to other machines because the current machine is overloaded.

### Disk Usage

If the application is doing a lot of disk I/O, that can quite easily become a bottleneck. Measuring this is tricky: if the app is not efficiently buffering data, disk I/O stats are *too low*, and performance can be improved. However, if the app is performing more I/O than the disk can handle, disk I/O stats are *too high*, and performance can again be improved but in a different way.

The other reason to monitor disk I/O is to identify if the application is causing the system to *swap memory*. If the app is constantly running out of physical memory, it'll start mapping into virtual memory space which is backed by the disk. The app swaps unused portions of physical memory into the disk to make space in the RAM. Tools like `vmstat` can indicate when data is being swapped in or out of the disk.

### Network Usage

Similar to disk usage, network usage can be under or over utilized by the same patterns.

`nicstat` (Unix) provides information summaries of traffic on each network interface, including the degree of utilization.

**Networks cannot sustain a 100% utilization rate**. For LAN Ethernet, a sustained rate of over 40% indicates that the interface is saturated.

## Java Monitoring Tools

+ `jcmd`. Basic class, thread, VM info.
+ `jconsole`. Graphical view of JVM activities -- thread use, class use, GC.
+ `jhat, jmap`. Heap dump / memory usage checks.
+ `jstack`. Stack dump.
+ `jinfo`. Visibility into system properties.
+ `jvisualvm`. GUI tool to monitor a JVM. Application profiling, JVM heap space analysis etc.

----

## References
1. [Java Performance: The Definitive Guide](https://www.oreilly.com/library/view/java-performance-the/9781449363512/) Chapter 3