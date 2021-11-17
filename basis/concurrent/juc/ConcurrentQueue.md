# 5.4 并发队列

LinkedBlockingQueue

SynchronousQueue 

DelayedWorkQueue

如果新请求的到达速率超过了线程池的处理速率，那么新到来的请求将累积起来。在线程池中，这些请求会在一个由Executor管理的Runnable队列中等待，而不会像线程那样去竞争CPU资源。常见的工作队列有以下几种，前三种用的最多。

1. ArrayBlockingQueue：列表形式的工作队列，必须要有初始队列大小，有界队列，先进先出。
2. LinkedBlockingQueue：链表形式的工作队列，可以选择设置初始队列大小，有界/无界队列，先进先出。
3. SynchronousQueue：SynchronousQueue不是一个真正的队列，而是一种在线程之间移交的机制。要将一个元素放入SynchronousQueue中, 必须有另一个线程正在等待接受这个元素. 如果没有线程等待，并且线程池的当前大小小于最大值，那么ThreadPoolExecutor将创建 一个线程, 否则根据饱和策略，这个任务将被拒绝。使用直接移交将更高效，因为任务会直接移交 给执行它的线程，而不是被首先放在队列中, 然后由工作者线程从队列中提取任务. 只有当线程池是无界的或者可以拒绝任务时，SynchronousQueue才有实际价值.
4. PriorityBlockingQueue：优先级队列，无界队列，根据优先级来安排任务，任务的优先级是通过自然顺序或Comparator（如果任务实现了Comparator）来定义的。
5. DelayedWorkQueue：延迟的工作队列，无界队列。



https://blog.csdn.net/educast/article/details/77102360

BlockingQueue  BlockingDeque  TransferQueue
