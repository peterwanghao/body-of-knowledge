# 2 MySQL的四种事务隔离级别

## **事务的基本要素（ACID）**

　　**1、原子性（Atomicity）：事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。**

　　 **2、一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。**

　　 **3、隔离性（Isolation）：同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。**

　　 **4、持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。**



## **事务的并发问题**

**1、脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据**

**2、不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。**

**3、幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。**

**小结：<font color=#FF0000 >不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表</font>**



## MySQL事务隔离级别

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 读未提交（read-uncommitted） | 是   | 是         | 是   |
| 不可重复读（read-committed） | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 串行化（serializable）       | 否   | 否         | 否   |

在MySQL中，默认的隔离级别是REPEATABLE-READ（可重复读），并且解决了幻读问题。



**1.READ UNCOMMITTED**

这种隔离级别下普通select语句是不加事务锁的，因此会产生脏读，这种事务隔离级别是应当完全避免的。除select语句以外的其他语句加锁模式与READ COMMITTED一样。

**2.READ COMMITTED**

同REPEATABLE READ一样，这种隔离级别下也实现了一致性非锁定读，但区别在于此隔离级别下的一致性读是语句级的，即只能避免脏读，不能避免不可重复读和幻读。其实现方式大致是：

- 一致性非锁定读的select语句检测要锁定的索引记录上是否有独占锁（在server层也会添加S模式的record lock，此锁为server层的元数据库锁，非innodb事务锁）。
- 如果有独占锁那么到undo中寻找最近的前镜像。
- 如果没有独占锁那么直接读取数据。

在这种隔离级别下，InnoDB的锁定读（SELECT with FOR UPDATE or LOCK IN SHARE MODE）只使用record lock类型的行锁，不使用gap锁。

此外：如果你使用READ COMMITTED事物隔离级别，那么binlog模式必须修改为row模式！

关于具体的MVCC实现方式，MySQL官网并未提供具体的实现步骤，可以选择去查看源码，也可以参考Oracle和SQL Server的实现机制。

**3.REPEATABLE READ**

这是MySQL的默认事务隔离级别。在一个事务当中第一次读会建立一个全库snapshot，同事务下的select语句会读取这个snapshot来实现一致性非锁定读。而第一个snapshot的建立猜测与READ COMMITTED下的读取机制一样。同事务的select语句会读取这个snapshot的数据来实现一致性非锁定读，这个snapshot是针对整个数据库中所有支持MVCC机制的表的，即在snapshot建立后读取任意其他表都只会读取到snapshot中的快照数据，示例如下：

时刻一，A会话执行：select * from T2 where id=1; 
时刻二，A会话执行：start transaction; select * from T1; --snapshot建立
时刻三，B会话执行：针对T1表和T2表的DML语句修改数据
时刻四，A会话执行：select * from T1; --发现读到的数据与时刻二一模一样，证明表T1实现了一致性读
时刻五，A会话执行：select * from T2 where id=1; --发现读到的数据与时刻一一致，证明snapshot非表级，而是库级

这种隔离级别下可以避免脏读、不可重复读和幻读。

对于select for update/select lock in share mode/update/delete这些锁定读，加行锁模式取决于索引的类型：

- 对唯一索引的访问只会添加record lock，而不会使用gap lock（即也没有next-key lock）。
- 对非唯一索引的访问使用gap lock或者next-key lock，如果访问的记录不存在就是gap lock，否则就是next-key lock。



**InnoDB和XtraDB存储引擎通过多版本并发控制（MVCC，MultiversionConcurrencyControl）解决了幻读的问题。**

**4.SERIALIZABLE**

这种事务隔离级下select语句即便不加lock in share mode也使用lock_mode=S的行锁，select自成事务，锁直到事务结束才释放。

这种隔离级别下可以避免脏读、不可重复读和幻读。

DML语句的加锁模式与REPEATABLE READ一样。

官网对于这个隔离级别的解释是只有将autocommit设置为0后select才会被隐式转换为lock in share mode的加锁模式，但是经测验发现在此模式下只要为select语句开启事务就会阻塞其他事物的更改，因此官网解释应该有误。

 

**总结**

一般来说我们没必要去修改默认的事务隔离级别，当然如果你的数据库并不在意幻读和不可重复读，可以修改未read committed隔离级别，这样可以增加并发减少阻塞，据说淘宝也是这么干的。Oracle默认的事务隔离级别也是read committed，同样不可避免幻读和不可重复读。

关于MySQL的锁机制，可以参考：http://www.cnblogs.com/leohahah/p/8862216.html

其他：

关于一致性非锁定读和锁定读的解释，详见MySQL各类SQL语句的加锁机制。