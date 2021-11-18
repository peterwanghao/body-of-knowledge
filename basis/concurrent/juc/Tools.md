# 5.6 同步工具

## 1、CountDownLatch

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）。

CountDownLatch能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行。使用一个计数器进行实现。计数器初始值为线程的数量。当每一个线程完成自己任务后，计数器的值就会减一。当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务。

构造函数

```java
//参数count为计数值
public CountDownLatch(int count) {  };  
```

重要方法

```java
//调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public void await() throws InterruptedException { };   
//和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  
//将count值减1
public void countDown() { };  
```

### 示例

```java
public class CountDownLatchTest {

    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(2);
        System.out.println("主线程开始执行…… ……");
        //第一个子线程执行
        ExecutorService es1 = Executors.newSingleThreadExecutor();
        es1.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                    System.out.println("子线程："+Thread.currentThread().getName()+"执行");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                latch.countDown();
            }
        });
        es1.shutdown();

        //第二个子线程执行
        ExecutorService es2 = Executors.newSingleThreadExecutor();
        es2.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("子线程："+Thread.currentThread().getName()+"执行");
                latch.countDown();
            }
        });
        es2.shutdown();
        System.out.println("等待两个线程执行完毕…… ……");
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("两个子线程都执行完毕，继续执行主线程");
    }
}
```

**运行结果：**

```java
主线程开始执行…… ……
等待两个线程执行完毕…… ……
子线程：pool-1-thread-1执行
子线程：pool-2-thread-1执行
两个子线程都执行完毕，继续执行主线程
```

### **CountDownLatch实现原理**

**1、创建计数器**

当我们调用CountDownLatch countDownLatch=new CountDownLatch(4) 时候，此时会创建一个AQS的同步队列，并把创建CountDownLatch 传进来的计数器赋值给AQS队列的 state，所以state的值也代表CountDownLatch所剩余的计数次数；

```java
  public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);//创建同步队列，并设置初始计数器值
    }
```

**2、阻塞线程**

当我们调用countDownLatch.wait()的时候，会创建一个节点，加入到AQS阻塞队列，并同时把当前线程挂起。

```java
  public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```



判断计数器是技术完毕，未完毕则把当前线程加入阻塞队列

```java
  public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        //锁重入次数大于0 则新建节点加入阻塞队列，挂起当前线程
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```



构建阻塞队列的双向链表，挂起当前线程

```java
 private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        //新建节点加入阻塞队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                //获得当前节点pre节点
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);//返回锁的state
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                //重组双向链表，清空无效节点，挂起当前线程
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

**3、计数器递减**

当我们调用countDownLatch.down()方法的时候，会对计数器进行减1操作，AQS内部是通过释放锁的方式，对state进行减1操作，当state=0的时候证明计数器已经递减完毕，此时会将AQS阻塞队列里的节点线程全部唤醒。



```java
 public void countDown() {
        //递减锁重入次数，当state=0时唤醒所有阻塞线程
        sync.releaseShared(1);
    }
```



```java
public final boolean releaseShared(int arg) {
        //递减锁的重入次数
        if (tryReleaseShared(arg)) {
            doReleaseShared();//唤醒队列所有阻塞的节点
            return true;
        }
        return false;
    }
 private void doReleaseShared() {
        //唤醒所有阻塞队列里面的线程
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {//节点是否在等待唤醒状态
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))//修改状态为初始
                        continue;
                    unparkSuccessor(h);//成功则唤醒线程
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```



## 2、CyclicBarrier

CyclicBarrier字面意思是“可重复使用的栅栏”。大概的意思就是一个可循环利用的屏障。

它的作用就是会让所有线程都等待完成后才会继续下一步行动。

举个例子，就像生活中我们会约朋友们到某个餐厅一起吃饭，有些朋友可能会早到，有些朋友可能会晚到，但是这个餐厅规定必须等到所有人到齐之后才会让我们进去。这里的朋友们就是各个线程，餐厅就是 CyclicBarrier。

CyclicBarrier 使用场景：可以用于多线程计算数据，最后合并计算结果的场景。其源码没有什么高深的地方，它是 ReentrantLock 和 Condition 的组合使用。CyclicBarrier 的源码实现和 CountDownLatch 大同小异，CountDownLatch 基于 AQS 的共享模式的使用，而 CyclicBarrier 基于 Condition 来实现的。

在CyclicBarrier类的内部有一个计数器，每个线程在到达屏障点的时候都会调用await方法将自己阻塞，此时计数器会减1，当计数器减为0的时候所有因调用await方法而被阻塞的线程将被唤醒。这就是实现一组线程相互等待的原理。

### 方法

```java
public CyclicBarrier(int parties)
public CyclicBarrier(int parties, Runnable barrierAction)
```

**解析：**

- parties 是参与线程的个数
- 第二个构造方法有一个 Runnable 参数，这个参数的意思是最后一个到达线程要做的任务

```java
public int await() throws InterruptedException, BrokenBarrierException
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException
```

**解析：**

- 线程调用 await() 表示自己已经到达栅栏
- BrokenBarrierException 表示栅栏已经被破坏，破坏的原因可能是其中一个线程 await() 时被中断或者超时

### 示例

```java
public class CyclicBarrierDemo {

    static class TaskThread extends Thread {
        
        CyclicBarrier barrier;
        
        public TaskThread(CyclicBarrier barrier) {
            this.barrier = barrier;
        }
        
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
                System.out.println(getName() + " 到达栅栏 A");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 A");
                
                Thread.sleep(2000);
                System.out.println(getName() + " 到达栅栏 B");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 B");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) {
        int threadNum = 5;
        CyclicBarrier barrier = new CyclicBarrier(threadNum, new Runnable() {
            
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " 完成最后任务");
            }
        });
        
        for(int i = 0; i < threadNum; i++) {
            new TaskThread(barrier).start();
        }
    }
    
}
```

**运行结果：**

```java
Thread-1 到达栅栏 A
Thread-3 到达栅栏 A
Thread-0 到达栅栏 A
Thread-4 到达栅栏 A
Thread-2 到达栅栏 A
Thread-2 完成最后任务
Thread-2 冲破栅栏 A
Thread-1 冲破栅栏 A
Thread-3 冲破栅栏 A
Thread-4 冲破栅栏 A
Thread-0 冲破栅栏 A
Thread-4 到达栅栏 B
Thread-0 到达栅栏 B
Thread-3 到达栅栏 B
Thread-2 到达栅栏 B
Thread-1 到达栅栏 B
Thread-1 完成最后任务
Thread-1 冲破栅栏 B
Thread-0 冲破栅栏 B
Thread-4 冲破栅栏 B
Thread-2 冲破栅栏 B
Thread-3 冲破栅栏 B
```

从打印结果可以看出，所有线程会等待全部线程到达栅栏之后才会继续执行，并且最后到达的线程会完成 Runnable 的任务。

### CyclicBarrier与CountDownLatch的区别

至此我们难免会将CyclicBarrier与CountDownLatch进行一番比较。这两个类都可以实现一组线程在到达某个条件之前进行等待，它们内部都有一个计数器，当计数器的值不断的减为0的时候所有阻塞的线程将会被唤醒。

有区别的是**CyclicBarrier的计数器由自己控制，而CountDownLatch的计数器则由使用者来控制**，在CyclicBarrier中线程调用await方法不仅会将自己阻塞还会将计数器减1，而在CountDownLatch中线程调用await方法只是将自己阻塞而不会减少计数器的值。

另外，**CountDownLatch只能拦截一轮，而CyclicBarrier可以实现循环拦截**。一般来说用CyclicBarrier可以实现CountDownLatch的功能，而反之则不能。

除此之外，CyclicBarrier还提供了：resert()、getNumberWaiting()、isBroken()等比较有用的方法。



## 3、Phaser

Phaser又称“阶段器”，用来解决控制多个线程分阶段共同完成任务的情景问题。它与CountDownLatch和CyclicBarrier类似，都是等待一组线程完成工作后再执行下一步，协调线程的工作。但在CountDownLatch和CyclicBarrier中我们都不可以动态的配置parties，而Phaser可以动态注册需要协调的线程，相比CountDownLatch和CyclicBarrier就会变得更加灵活。

Phaser支持通过register()和bulkRegister(int parties)方法来动态调整注册任务的数量，此外也支持通过其构造函数进行指定初始数量。在适当的时机，Phaser支持减少注册任务的数量，例如 arriveAndDeregister()。单个Phaser实例允许的注册任务数的上限是**65535**。



正如Phaser类的名字所暗示，每个Phaser实例都会维护一个phase number，初始值为0。每当所有注册的任务都到达Phaser时，phase number累加，并在超过Integer.MAX_VALUE后清零。arrive()和arriveAndDeregister()方法用于记录到 达，arriveAndAwaitAdvance()方法用于记录到达，并且等待其它未到达的任务。


Phaser支持层次结构，即通过构造函数Phaser(Phaser parent)和Phaser(Phaser parent, int parties)构造一个树形结构。这有助于减轻因在单个的Phaser上注册过多的任务而导致的竞争，从而提升吞吐量，代价是增加单个操作的开销。

### 常用方法
1、register方法  动态添加一个parties
2、bulkRegister方法  动态添加多个parties
3、getRegisteredParties方法 获取当前的parties数
4、arriveAndAwaitAdvance方法  到达并等待其他线程到达
5、arriveAndDeregister方法  到达并注销该parties，这个方法不会使线程阻塞
6、arrive方法  到达，但不会使线程阻塞
7、awaitAdvance方法  等待前行，可阻塞也可不阻塞，判断条件为传入的phase是否为当前phaser的phase。如果相等则阻塞，反之不进行阻塞
8、awaitAdvanceInterruptibly方法  该方法与awaitAdvance类似，唯一不一样的就是它可以进行打断。
9、getArrivedParties方法  获取当前到达的parties数
10、getUnarrivedParties方法  获取当前未到达的parties数
11、getPhase方法  获取当前属于第几阶段，默认从0开始，最大为integer的最大值
12、isTerminated方法  判断当前phaser是否关闭
13、forceTermination方法  强制关闭当前phaser

### 使用Phaser设置多个阶段

- 这边使用的案例是运动员，模拟多个运动员参加多个项目。

```java
package com.test.part3.lock;

import java.util.Random;
import java.util.concurrent.Phaser;
import java.util.concurrent.TimeUnit;

public class PhaserExample2 {
    private static Random random = new Random(System.currentTimeMillis());
    public static void main(String[] args) {
        //初始化5个parties
        Phaser phaser = new Phaser(5);
        for (int i=1;i<6;i++){
            new Athlete(phaser,i).start();
        }
    }
    //创建运动员类
    private static class Athlete extends Thread{
        private Phaser phaser;
        private int no;//运动员编号

        public Athlete(Phaser phaser,int no) {
            this.phaser = phaser;
            this.no = no;
        }

        @Override
        public void run() {
            try {
                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start running");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end running");
                //等待其他运动员完成跑步
                phaser.arriveAndAwaitAdvance();

                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start bicycle");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end bicycle");
                //等待其他运动员完成骑行
                phaser.arriveAndAwaitAdvance();

                System.out.println(no+": 当前处于第："+phaser.getPhase()+"阶段");
                System.out.println(no+": start long jump");
                TimeUnit.SECONDS.sleep(random.nextInt(5));
                System.out.println(no+": end long jump");
                //等待其他运动员完成跳远
                phaser.arriveAndAwaitAdvance();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**运行结果：**

```java
1: 当前处于第：0阶段
3: 当前处于第：0阶段
3: start running
2: 当前处于第：0阶段
2: start running
1: start running
4: 当前处于第：0阶段
4: start running
5: 当前处于第：0阶段
5: start running
5: end running
2: end running
1: end running
4: end running
3: end running
3: 当前处于第：1阶段
5: 当前处于第：1阶段
5: start bicycle
1: 当前处于第：1阶段
1: start bicycle
2: 当前处于第：1阶段
2: start bicycle
3: start bicycle
4: 当前处于第：1阶段
4: start bicycle
1: end bicycle
3: end bicycle
4: end bicycle
5: end bicycle
2: end bicycle
3: 当前处于第：2阶段
1: 当前处于第：2阶段
1: start long jump
4: 当前处于第：2阶段
4: start long jump
5: 当前处于第：2阶段
5: start long jump
2: 当前处于第：2阶段
2: start long jump
3: start long jump
2: end long jump
4: end long jump
1: end long jump
5: end long jump
3: end long jump

Process finished with exit code 0

```




## 4、Exchanger

Exchanger 是 JDK 1.5 开始提供的一个用于两个工作线程之间交换数据的封装工具类（**对是两个之间而不是三个或者更多个线程之间**），简单说就是一个线程在完成一定的事务后想与另一个线程交换数据，则第一个先拿出数据的线程会一直等待第二个线程，直到第二个线程拿着数据到来时才能彼此交换对应数据。其定义为 `Exchanger<V>` 泛型类型，其中 V 表示可交换的数据类型，对外提供的接口很简单，具体如下：

- `Exchanger()：`无参构造方法。
- `V exchange(V v)：`等待另一个线程到达此交换点（除非当前线程被中断），然后将给定的对象传送给该线程，并接收该线程的对象。
- `V exchange(V v, long timeout, TimeUnit unit)：`等待另一个线程到达此交换点（除非当前线程被中断或超出了指定的等待时间），然后将给定的对象传送给该线程，并接收该线程的对象。

可以看出，当一个线程到达 exchange 调用点时，如果其他线程此前已经调用了此方法，则其他线程会被调度唤醒并与之进行对象交换，然后各自返回；如果其他线程还没到达交换点，则当前线程会被挂起，直至其他线程到达才会完成交换并正常返回，或者当前线程被中断或超时返回。

```java
package com.securitit.serialize.juc;

import java.util.concurrent.Exchanger;

public class ExchangerTester {

	// Exchanger实例.
	private static final Exchanger<String> exchanger = new Exchanger<String>();

	public static void main(String[] args) {
		// 模拟阻塞线程.
		new Thread(() -> {
			try {
				String wares = "红烧肉";
				System.out.println(Thread.currentThread().getName() + "商品方正在等待金钱方，使用货物兑换为金钱.");
				Thread.sleep(2000);
				String money = exchanger.exchange(wares);
				System.out.println(Thread.currentThread().getName() + "商品方使用商品兑换了" + money);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
		}).start();
		// 模拟阻塞线程.
		new Thread(() -> {
			try {
				String money = "人民币";
				System.out.println(Thread.currentThread().getName() + "金钱方正在等待商品方，使用金钱购买食物.");
				Thread.sleep(4000);
				String wares = exchanger.exchange(money);
				System.out.println(Thread.currentThread().getName() + "金钱方使用金钱购买了" + wares);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
		}).start();
	}

}
```

输出结果：

```java
Thread-0商品方正在等待金钱方，使用货物兑换为金钱.
Thread-1金钱方正在等待商品方，使用金钱购买食物.
Thread-0商品方使用商品兑换了人民币
Thread-1金钱方使用金钱购买了红烧肉
```

## TimeUnit

TimeUnit是java.util.concurrent包下面的一个类，表示给定单元粒度的时间段

主要作用

- 时间颗粒度转换
- 延时

### 常用的颗粒度

```java
TimeUnit.DAYS          //天
TimeUnit.HOURS         //小时
TimeUnit.MINUTES       //分钟
TimeUnit.SECONDS       //秒
TimeUnit.MILLISECONDS  //毫秒 1秒=1000豪秒
TimeUnit.MICROSECONDS  //微秒 1毫秒=1000微秒
TimeUnit.NANOSECONDS   //毫微秒 1微秒=1000毫微秒＝1000纳秒
```
### 1、时间颗粒度转换 

```java
public long toNanos(long duration) //转化为毫微秒
public long toMicros(long duration) //转化为微秒
public long toMillis(long d)    //转化成毫秒
public long toSeconds(long d)  //转化成秒
public long toMinutes(long d)  //转化成分钟
public long toHours(long d)    //转化成小时
public long toDays(long d)     //转化天
```

　　例子

```java
package com.app;
 
import java.util.concurrent.TimeUnit;
 
public class Test {
 
    public static void main(String[] args) {
        //1天有24个小时    1代表1天：将1天转化为小时
        System.out.println( TimeUnit.DAYS.toHours( 1 ) );
         
        //结果： 24
         
 
        //1小时有3600秒
        System.out.println( TimeUnit.HOURS.toSeconds( 1 ));
         
        //结果3600
         
         
        //把3天转化成小时
        System.out.println( TimeUnit.HOURS.convert( 3 , TimeUnit.DAYS ) );
        //结果是：72
 
    }
}
```

　　

###  2、延时

-  一般的写法

```java
package com.app;
 
public class Test2 {
 
    public static void main(String[] args) {
 
        new Thread( new Runnable() {
 
            @Override
            public void run() {
                try {
                    Thread.sleep( 5 * 1000 );
                    System.out.println( "延时完成了");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();  ;
    }
     
}
```

- TimeUnit 写法

```java
package com.app;
 
import java.util.concurrent.TimeUnit;
 
public class Test2 {
 
    public static void main(String[] args) {
 
        new Thread( new Runnable() {
 
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep( 5 );
                    System.out.println( "延时5秒，完成了");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();  ;
    }
     
}
```
