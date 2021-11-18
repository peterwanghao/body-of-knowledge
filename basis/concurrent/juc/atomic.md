# 5.1 原子变量

https://blog.csdn.net/lmb55/article/details/79547685

https://blog.csdn.net/u011116672/article/details/51068828

https://www.cnblogs.com/theRhyme/p/12129120.html

https://www.jianshu.com/p/4a99f2e65dcc

https://blog.csdn.net/jiangtianjiao/article/details/103844801/

https://www.cnblogs.com/wyq1995/p/12242984.html



https://www.cnblogs.com/wangzun/p/8268581.html

java.util.concurrent.atomic原子操作类包里面提供了一组原子变量类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。可以对基本数据、数组中的基本数据、对类中的基本数据进行操作。原子变量类相当于一种泛化的volatile变量，能够支持原子的和有条件的读-改-写操作。

**java.util.concurrent.atomic中的类可以分成4组：**

- 标量类：AtomicBoolean，AtomicInteger，AtomicLong，AtomicReference
- 数组类：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
- 更新器类：AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
- 复合变量类：AtomicMarkableReference，AtomicStampedReference
  

valueOffSet：value 在 AtomicInteger 实例中的内存偏移地址

## 标量类

AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference这四种基本类型用来处理布尔，整数，长整数，对象四种数据，其内部实现不是简单的使用synchronized，而是一个更为高效的方式CAS (compare and swap) + volatile和native方法，从而避免了synchronized的高开销，执行效率大为提升。其实例各自提供对相应类型单个变量的访问和更新。每个类也为该类型提供适当的实用工具方法。



## 数组类

AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray类进一步扩展了原子操作，对这些类型的数组提供了支持。这些类在为其数组元素提供volatile访问语义方面也引人注目，这对于普通数组来说是不受支持的。其内部并不是像AtomicInteger一样维持一个volatile变量，而是全部由native方法实现。数组变量进行volatile没有意义，因此set/get就需要unsafe来做了，但是多了一个index来指定操作数组中的哪一个元素。


## 更新器类

AtomicReferenceFieldUpdater,AtomicIntegerFieldUpdater和AtomicLongFieldUpdater 是基于反射的实用工具，可以提供对关联字段类型的访问，可用于获取任意选定volatile字段上的compareAndSet操作。它们主要用于原子数据结构中，该结构中同一节点的几个 volatile 字段都独立受原子更新控制。这些类在如何以及何时使用原子更新方面具有更大的灵活性，但相应的弊端是基于映射的设置较为拙笨、使用不太方便，而且在保证方面也较差。

使用中要注意一下几点：

（1）字段必须是volatile类型的
（2）字段的描述类型（修饰符public/protected/default/private）是与调用者与操作对象字段的关系一致。也就是说 调用者能够直接操作对象字段，那么就可以反射进行原子操作。但是对于父类的字段，子类是不能直接操作的，尽管子类可以访问父类的字段。
（3）只能是实例变量，不能是类变量，也就是说不能加static关键字。
（4）只能是可修改变量，不能使final变量，因为final的语义就是不可修改。实际上final的语义和volatile是有冲突的，这两个关键字不能同时存在。
（5）对于AtomicIntegerFieldUpdater 和AtomicLongFieldUpdater 只能修改int/long类型的字段，不能修改其包装类型（Integer/Long）。如果要修改包装类型就需要使用AtomicReferenceFieldUpdater 。


## 复合变量类

AtomicMarkableReference 类将单个布尔值与引用关联起来。维护带有标记位的对象引用，可以原子方式更新带有标记位的引用类型。

AtomicStampedReference 类将整数值与引用关联起来。维护带有整数“标志”的对象引用，可以原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于**原子的更新数据和版本号**，可以解决使用CAS进行原子更新时，可能出现的ABA问题。


## LongAdder DoubleAdder

## LongAccumulator DoubleAccumulator
