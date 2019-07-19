# 3 分布式事务解决方案

SpringBoot 集成 Atomikos 实现分布式事务




## LCN
**LCN分布式事务框架**的核心功能是对本地事务的协调控制，框架本身并不创建事务，只是对本地事务做协调控制。因此该框架与其他第三方的框架兼容性强，支持所有的关系型数据库事务，支持多数据源，支持与第三方数据库框架一块使用（例如 sharding-jdbc），在使用框架的时候只需要添加分布式事务的注解即可，对业务的侵入性低。LCN框架主要是为微服务框架提供分布式事务的支持，在微服务框架上做了进一步的事务机制优化，在一些负载场景上LCN事务机制要比本地事务机制的性能更好，4.0以后框架开方了插件机制可以让更多的第三方框架支持进来



**在需要执行的事务上添加注解**

```
@Override
@TxTransaction(isStart = true)
@Transactional
public int save() {
}
```

其中 @TxTransaction(isStart = true) 为lcn 事务控制注解，其中isStart = true 表示该方法是事务的发起方例如，服务A 需要调用服务B,服务B 需要调用服务C，此时 服务A为服务发起方，其余为参与方，参与方只需@TxTransaction 即可

在测试时需要将 事务管理服务启动 txManager, 具体示例参看：https://www.txlcn.org

## ByteTCC

ByteTCC是一个基于TCC（Try/Confirm/Cancel）机制的分布式事务管理器。兼容JTA，可以很好的与EJB、Spring等容器（本文档下文说明中将以Spring容器为例）进行集成。

ByteTCC特性
1、支持Spring容器的声明式事务管理；
2、支持普通事务、TCC事务、业务补偿型事务等事务机制；
3、支持多数据源、跨应用、跨服务器等分布式事务场景；
4、支持长事务；
5、支持dubbo服务框架；
6、支持spring cloud；

该实现方式，需要在业务层编写对应的 tcc（Try/Confirm/Cancel） 方法，开发需要一定的成本，同时某些业务可能无法保证数据可回滚