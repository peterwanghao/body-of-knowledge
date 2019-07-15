# 2.1 CAP原理

**分布式领域CAP理论**

Consistency(一致性), 数据一致更新，所有数据变动都是同步的

Availability(可用性), 好的响应性能

Partition tolerance(分区容忍性) 可靠性



定理：任何分布式系统只可同时满足二点，没法三者兼顾。

忠告：架构师不要将精力浪费在如何设计能满足三者的完美分布式系统，而是应该进行取舍。



关系数据库的ACID模型拥有 高一致性 + 可用性 很难进行分区：

Atomicity原子性：一个事务中所有操作都必须全部完成，要么全部不完成。

Consistency一致性. 在事务开始或结束时，数据库应该在一致状态。

Isolation隔离层. 事务将假定只有它自己在操作数据库，彼此不知晓。

Durability. 一旦事务完成，就不能返回。



跨数据库两段提交事务：2PC (two-phase commit)， 2PC is the anti-scalability pattern (Pat Helland) 是反可伸缩模式的，JavaEE中的JTA事务可以支持2PC。因为2PC是反模式，尽量不要使用2PC，使用BASE来回避。



