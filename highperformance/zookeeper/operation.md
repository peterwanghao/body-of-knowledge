# 1 Zookeeper 使用

## 配置 zoo.cfg

| tickTime       | 作为Zookeeper服务器之间或客户端与服务器之间维护心跳的时间间隔 |
| -------------- | ------------------------------------------------------------ |
| dataDir        | Zookeeper保存数据的目录，日志文件也保存在这个目录            |
| clientPort     | 客户端连接Zookeeper服务器的端口                              |
| initLimit      | Zookeeper服务器集群中连接到Leader的Follower初始化连接时最长能忍受多少个心跳时间间隔 |
| syncLimit      | Leader和Follower之间发送消息，请求和应答时间长度             |
| server.A=B:C:D | A是一个数字 表示这个是第几号服务器B是这个服务器的IP地址C表示这个服务器与集群中的Leader服务器交换信息的端口D表示万一集群中的Leader服务器挂了，需要重新进行选举。这个端口就是用来   执行选举时服务器相互通信的端口 |

集群模式下还要配置一个文件myid，这个文件在dataDir目录下。

Zookeeper每个子目录项都被称为znode，znode可以有子节点目录，并且每个znode可以存储数据。

Zookeeper的客户端和服务端通信采用长连接方式，通过心跳来保持连接。这个连接状态称为session，如果znode是临时节点，这个session失效，znode也就删除了。

znode可以被监控，一旦变化可以通知设置监控的客户端。



Zookeeper从设计模式角度来看，是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据。然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就负责通知已经在Zookeeper上注册的那些观察者做出相应反应。

## ZooKeeper Client API

ZooKeeper Client Library提供了丰富直观的API供用户程序使用，下面是一些常用的API：

- create(path, data, flags): 创建一个ZNode, path是其路径，data是要存储在该ZNode上的数据，flags常用的有: PERSISTEN, PERSISTENT_SEQUENTAIL, EPHEMERAL, EPHEMERAL_SEQUENTAIL
- delete(path, version): 删除一个ZNode，可以通过version删除指定的版本, 如果version是-1的话，表示删除所有的版本
- exists(path, watch): 判断指定ZNode是否存在，并设置是否Watch这个ZNode。这里如果要设置Watcher的话，Watcher是在创建ZooKeeper实例时指定的，如果要设置特定的Watcher的话，可以调用另一个重载版本的exists(path, watcher)。以下几个带watch参数的API也都类似
- getData(path, watch): 读取指定ZNode上的数据，并设置是否watch这个ZNode
- setData(path, watch): 更新指定ZNode的数据，并设置是否Watch这个ZNode
- getChildren(path, watch): 获取指定ZNode的所有子ZNode的名字，并设置是否Watch这个ZNode
- sync(path): 把所有在sync之前的更新操作都进行同步，达到每个请求都在半数以上的ZooKeeper Server上生效。path参数目前没有用
- setAcl(path, acl): 设置指定ZNode的Acl信息
- getAcl(path): 获取指定ZNode的Acl信息



## Zookeeper典型的应用场景

- 统一命名服务（Name Service）

- 配置管理（Configuration Management）

- 集群管理（Group Membership）

- 共享锁（Locks）

- 队列管理

- - 当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列
  - 队列按照FIFO方式进入入队和出队操作，实现生产者和消费者模式

**1. 名字服务(NameService)** 

分布式应用中，通常需要一套完备的命令机制，既能产生唯一的标识，又方便人识别和记忆。 我们知道，每个ZNode都可以由其路径唯一标识，路径本身也比较简洁直观，另外ZNode上还可以存储少量数据，这些都是实现统一的NameService的基础。下面以在HDFS中实现NameService为例，来说明实现NameService的基本布骤:

- 目标：通过简单的名字来访问指定的HDFS机群
- 定义命名规则：这里要做到简洁易记忆。下面是一种可选的方案： [serviceScheme://][zkCluster]-[clusterName]，比如hdfs://lgprc-example/表示基于lgprc ZooKeeper集群的用来做example的HDFS集群
- 配置DNS映射: 将zkCluster的标识lgprc通过DNS解析到对应的ZooKeeper集群的地址
- 创建ZNode: 在对应的ZooKeeper上创建/NameService/hdfs/lgprc-example结点，将HDFS的配置文件存储于该结点下
- 用户程序要访问hdfs://lgprc-example/的HDFS集群，首先通过DNS找到lgprc的ZooKeeper机群的地址，然后在ZooKeeper的/NameService/hdfs/lgprc-example结点中读取到HDFS的配置，进而根据得到的配置，得到HDFS的实际访问入口

**2. 配置管理(Configuration Management)** 

在分布式系统中，常会遇到这样的场景: 某个Job的很多个实例在运行，它们在运行时大多数配置项是相同的，如果想要统一改某个配置，一个个实例去改，是比较低效，也是比较容易出错的方式。通过ZooKeeper可以很好的解决这样的问题，下面的基本的步骤：

- 将公共的配置内容放到ZooKeeper中某个ZNode上，比如/service/common-conf
- 所有的实例在启动时都会传入ZooKeeper集群的入口地址，并且在运行过程中Watch /service/common-conf这个ZNode
- 如果集群管理员修改了了common-conf，所有的实例都会被通知到，根据收到的通知更新自己的配置，并继续Watch /service/common-conf

**3. 组员管理(Group Membership)** 

在典型的Master-Slave结构的分布式系统中，Master需要作为“总管”来管理所有的Slave, 当有Slave加入，或者有Slave宕机，Master都需要感知到这个事情，然后作出对应的调整，以便不影响整个集群对外提供服务。以HBase为例，HMaster管理了所有的RegionServer，当有新的RegionServer加入的时候，HMaster需要分配一些Region到该RegionServer上去，让其提供服务；当有RegionServer宕机时，HMaster需要将该RegionServer之前服务的Region都重新分配到当前正在提供服务的其它RegionServer上，以便不影响客户端的正常访问。下面是这种场景下使用ZooKeeper的基本步骤：

- Master在ZooKeeper上创建/service/slaves结点，并设置对该结点的Watcher
- 每个Slave在启动成功后，创建唯一标识自己的临时性(Ephemeral)结点/service/slaves/${slave_id}，并将自己地址(ip/port)等相关信息写入该结点
- Master收到有新子结点加入的通知后，做相应的处理
- 如果有Slave宕机，由于它所对应的结点是临时性结点，在它的Session超时后，ZooKeeper会自动删除该结点
- Master收到有子结点消失的通知，做相应的处理

**4. 简单互斥锁(Simple Lock)** 

我们知识，在传统的应用程序中，线程、进程的同步，都可以通过操作系统提供的机制来完成。但是在分布式系统中，多个进程之间的同步，操作系统层面就无能为力了。这时候就需要像ZooKeeper这样的分布式的协调(Coordination)服务来协助完成同步，下面是用ZooKeeper实现简单的互斥锁的步骤，这个可以和线程间同步的mutex做类比来理解：

- 多个进程尝试去在指定的目录下去创建一个临时性(Ephemeral)结点 /locks/my_lock
- ZooKeeper能保证，只会有一个进程成功创建该结点，创建结点成功的进程就是抢到锁的进程，假设该进程为A
- 其它进程都对/locks/my_lock进行Watch
- 当A进程不再需要锁，可以显式删除/locks/my_lock释放锁；或者是A进程宕机后Session超时，ZooKeeper系统自动删除/locks/my_lock结点释放锁。此时，其它进程就会收到ZooKeeper的通知，并尝试去创建/locks/my_lock抢锁，如此循环反复

**5. 互斥锁(Simple Lock without Herd Effect)** 

上一节的例子中有一个问题，每次抢锁都会有大量的进程去竞争，会造成羊群效应(Herd Effect)，为了解决这个问题，我们可以通过下面的步骤来改进上述过程：

- 每个进程都在ZooKeeper上创建一个临时的顺序结点(Ephemeral Sequential) /locks/lock_${seq}
- ${seq}最小的为当前的持锁者(${seq}是ZooKeeper生成的Sequenctial Number)
- 其它进程都对只watch比它次小的进程对应的结点，比如2 watch 1, 3 watch 2, 以此类推
- 当前持锁者释放锁后，比它次大的进程就会收到ZooKeeper的通知，它成为新的持锁者，如此循环反复

这里需要补充一点，通常在分布式系统中用ZooKeeper来做Leader Election(选主)就是通过上面的机制来实现的，这里的持锁者就是当前的“主”。

**6. 读写锁(Read/Write Lock)** 

我们知道，读写锁跟互斥锁相比不同的地方是，它分成了读和写两种模式，多个读可以并发执行，但写和读、写都互斥，不能同时执行行。利用ZooKeeper，在上面的基础上，稍做修改也可以实现传统的读写锁的语义，下面是基本的步骤:

- 每个进程都在ZooKeeper上创建一个临时的顺序结点(Ephemeral Sequential) /locks/lock_${seq}
- ${seq}最小的一个或多个结点为当前的持锁者，多个是因为多个读可以并发
- 需要写锁的进程，Watch比它次小的进程对应的结点
- 需要读锁的进程，Watch比它小的最后一个写进程对应的结点
- 当前结点释放锁后，所有Watch该结点的进程都会被通知到，他们成为新的持锁者，如此循环反复

**7. 屏障(Barrier)** 

在分布式系统中，屏障是这样一种语义: 客户端需要等待多个进程完成各自的任务，然后才能继续往前进行下一步。下用是用ZooKeeper来实现屏障的基本步骤：

- Client在ZooKeeper上创建屏障结点/barrier/my_barrier，并启动执行各个任务的进程
- Client通过exist()来Watch /barrier/my_barrier结点
- 每个任务进程在完成任务后，去检查是否达到指定的条件，如果没达到就啥也不做，如果达到了就把/barrier/my_barrier结点删除
- Client收到/barrier/my_barrier被删除的通知，屏障消失，继续下一步任务

**8. 双屏障(Double Barrier)**

双屏障是这样一种语义: 它可以用来同步一个任务的开始和结束，当有足够多的进程进入屏障后，才开始执行任务；当所有的进程都执行完各自的任务后，屏障才撤销。下面是用ZooKeeper来实现双屏障的基本步骤：

进入屏障：

- Client Watch /barrier/ready结点, 通过判断该结点是否存在来决定是否启动任务
- 每个任务进程进入屏障时创建一个临时结点/barrier/process/${process_id}，然后检查进入屏障的结点数是否达到指定的值，如果达到了指定的值，就创建一个/barrier/ready结点，否则继续等待
- Client收到/barrier/ready创建的通知，就启动任务执行过程

离开屏障：

- Client Watch /barrier/process，如果其没有子结点，就可以认为任务执行结束，可以离开屏障

- 每个任务进程执行任务结束后，都需要删除自己对应的结点/barrier/process/${process_id}