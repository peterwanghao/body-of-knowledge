# 原子变量

https://blog.csdn.net/lmb55/article/details/79547685

https://blog.csdn.net/u011116672/article/details/51068828

https://www.cnblogs.com/theRhyme/p/12129120.html

https://www.jianshu.com/p/4a99f2e65dcc

https://blog.csdn.net/jiangtianjiao/article/details/103844801/

https://www.cnblogs.com/wyq1995/p/12242984.html

java.util.concurrent.atomic原子操作类包里面提供了一组原子变量类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。可以对基本数据、数组中的基本数据、对类中的基本数据进行操作。原子变量类相当于一种泛化的volatile变量，能够支持原子的和有条件的读-改-写操作。

**java.util.concurrent.atomic中的类可以分成4组：**

- 标量类：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
- 数组类：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
- 更新器类：AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
- 复合变量类：AtomicMarkableReference，AtomicStampedReference
  

valueOffSet：value 在 AtomicInteger 实例中的内存偏移地址
