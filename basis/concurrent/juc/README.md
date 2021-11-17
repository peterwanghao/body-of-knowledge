# 常用的并发工具类有哪些？

- CountDownLatch 
- CyclicBarrier 
- Semaphore 
- Exchanger



# 同步集合与并发集合的区别

同步集合与并发集合都为多线程和并发提供了合适的线程安全的集合，不过并发 集合的可扩展性更高。同步集合比并发集合会慢得多，主要原因是锁，同步集合 会对整个 May 或 List 加锁，而并发集合例如 ConcurrentHashMap， 把整个 Map 划分成几个片段，只对相关的几个片段上锁，同时允许多线程访问 其他未上锁的片段(JDK1.8 版本底层加入了红黑树)。 



# ConcurrentHashMap 的并发度是什么 

ConcurrentHashMap 的并发度就是 segment 的大小，默认为 16，这意味着最多同时可以有 16 条线程操作 ConcurrentHashMap，这也是ConcurrentHashMap 对 Hashtable 的最大优势，任何情况下，Hashtable 能同时有两条线程获取Hashtable 中的数据吗？



# CyclicBarrier 和 CountDownLatch 的应用场景？

**CountDownLatch** : **一个线程**(或者多个)，等待另外 **N 个线程**完成**某个事情**之后才能执行。 

**CyclicBarrier** : **N 个线程**相互等待，任何一个线程完成之前，所有的线程都必须等待。 

**CountDownLatch 的使用场景：**

在一些应用场合中，需要等待某个条件达到要求后才能做后面的事情；同时当线程都完成后也会触发事件，以便进行后面的操作, 这个时候就可以使用 CountDownLatch。 

**CyclicBarrier 使用场景** ：

CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。 

# CyclicBarrier和CountDownLatch的区别

1. CountDownLatch 简单的说就是一个线程等待，直到他所等待的其他线程都执行完成并且调用 countDown()方法发出通知后，当前线程才可以继续执行。 
2. cyclicBarrier 是所有线程都进行等待，直到所有线程都准备好进入 await()方法之后，所有线程同时开始执行！ 
3. CountDownLatch 的计数器只能使用一次。而 CyclicBarrier 的计数器可以使用 reset() 方法重置。所以 CyclicBarrier 能处理更为复杂的业务场景，比如如果计算发生错误，可以重置计数器，并让线程们重新执行一次。
4. CyclicBarrier 还提供其他有用的方法，比如 getNumberWaiting 方法可以获得 CyclicBarrier 阻塞的线程数量。isBroken 方法用来知道阻塞的线程是否被中断。如果被中断返回 true，否则返回 false。 

# FutureTask 是什么

这个其实前面有提到过，FutureTask 表示一个异步运算的任务。FutureTask 里面可以传入一个 Callable 的具体实现类，可以对这个异步运算的任务的结果进行等待获取、判断是否已经完成、取消任务等操作。当然，由于 FutureTask 也是 Runnable 接口的实现类，所以 FutureTask 也可以放入线程池中。

# Semaphore 有什么作用

Semaphore 就是一个信号量，它的作用是**限制某段代码块的并发数**。 

Semaphore 有一个构造函数，可以传入一个 int 型整数 n，表示某段代码最多只有 n 个线程可以访问，如果超出了 n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果 Semaphore 构造函数中传入的 int 型整数 n=1，相当于变成了一个 synchronized 了。

# Executors 类是什么？

Executors 为 Executor，ExecutorService，ScheduledExecutorService， ThreadFactory 和 Callable 类提供了一些工具方法。 

Executors 可以用于方便的创建线程池。



# 类

java并发工具包-原子变量 atomic

java并发工具包-锁 locks

java并发工具包-执行器与线程池 

​	接口：

​	java.util.concurrent.Executor

​	java.util.concurrent.ExecutorService

​	java.util.concurrent.ScheduledExecutorService

​	java.util.concurrent.Future

​	java.util.concurrent.RunnableFuture

​	实现：

​	java.util.concurrent.ThreadPoolExecutor

​	java.util.concurrent.ScheduledThreadPoolExecutor

​	java.util.concurrent.Executors

​	java.util.concurrent.FutureTask

​	java.util.concurrent.ExecutorCompletionService

​	java.util.concurrent.ForkJoinPool

​	java.util.concurrent.ForkJoinTask

java并发工具包-并发队列 

​	java.util.concurrent.BlockingQueue

​		java.util.concurrent.ArrayBlockingQueue

​		java.util.concurrent.LinkedBlockingQueue

​		java.util.concurrent.PriorityBlockingQueue

​		java.util.concurrent.SynchronousQueue

​		java.util.concurrent.DelayQueue

​		java.util.concurrent.ConcurrentLinkedQueue

​	java.util.concurrent.BlockingDeque

​		java.util.concurrent.LinkedBlockingDeque

​		java.util.concurrent.ConcurrentLinkedDeque

​	java.util.concurrent.TransferQueue

​		java.util.concurrent.LinkedTransferQueue

​	

java并发工具包-并发集合 

​	java.util.concurrent.ConcurrentHashMap

​	java.util.concurrent.ConcurrentSkipListMap

​	java.util.concurrent.ConcurrentSkipListSet

​	java.util.concurrent.CopyOnWriteArrayList

​	java.util.concurrent.CopyOnWriteArraySet

java并发工具包-同步工具

​	java.util.concurrent.Semaphore

​	java.util.concurrent.CountDownLatch

​	java.util.concurrent.CyclicBarrier

​	java.util.concurrent.Phaser

​	java.util.concurrent.Exchanger

java并发工具包-时间处理 

​	java.util.concurrent.TimeUnit
