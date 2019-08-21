# 3.2 TX-LCN分布式事务框架

## 1. 原理介绍

### 1.1. 事务控制原理

1. TX-LCN由两大模块组成, **TxClient、TxManager**，TxClient作为模块的依赖框架，提供TX-LCN的标准支持，TxManager作为分布式事务的控制方。事务发起方或者参与方都由TxClient端来控制。（**简单来说就是单独部署一套TxManager模块来实现事务管理，TxClient就是我们自己的服务系统**）
2. 原理图如下：

![](./static/yuanli.png)

### 1.2. LCN事务模式

#### 1.2.1. 原理介绍

1. LCN模式是通过**代理Connection**的方式实现对**本地事务**的操作，然后在由**TxManager统一协调控制事务**。当本地事务提交回滚或者关闭连接时将会执行假操作，该代理的连接将由LCN连接池管理。

#### 1.2.2. 模式特点

1. 该模式对代码的**嵌入性为低**。
2. 该模式仅限于**本地存在连接对象**且可通过连接对象控制事务的模块。
3. 该模式下的事务提交与回滚是由本地事务方控制，对于**数据一致性上有较高的保障**。
4. 该模式缺陷在于**代理的连接需要随事务发起方一共释放连接**，增加了连接占用的时间

### 1.3. TCC事务模式

#### 1.3.1. 原理介绍

1. TCC事务机制相对于传统事务机制（X/Open XA Two-Phase-Commit），其特征在于它不依赖资源管理器(RM)对XA的支持，而是通过对（由业务系统提供的）业务逻辑的调度来实现分布式事务。主要由三步操作，**Try: 尝试执行业务、 Confirm:确认执行业务、 Cancel: 取消执行业务**。

#### 1.3.2. 模式特点

1. 该模式对代码的**嵌入性高**，要求**每个业务需要写三种步骤**的操作。
2. 该模式对**有无本地事务控制都可以支持**使用面广。
3. 数据一致性控制几乎**完全由开发者控制**，对业务开发**难度要求高**。

### 1.4. TXC事务模式

#### 1.4.1. 原理介绍

1. TXC模式命名来源于阿里云的GTS，实现原理是在执行SQL之前，先查询SQL的**影响数据保存起来**然后再执行业务。当需要回滚的时候就采用这些**记录数据回滚事务**。

#### 1.4.2. 模式特点

1. 该模式同样对代码的**嵌入性低**。
2. 该模式仅限于对**支持SQL方式的模块**支持。
3. 该模式由于每次执行SQL之前**需要先查询影响数据**，因此相比LCN模式**消耗资源与时间要多**。
4. 该模式**不会占用数据库的连接资源**。

## 2. 快速开始

### 2.1. 吐槽

1. 坑点一：不要相信官方网站上的的快速开始和示例，不知道是多久以前的了，我捣鼓了很久，下下来的代码缺斤少两的，打包了源代码也满足不了这示例代码的需求
2. 坑点二：不要直接使用源代码的tx-manager，肯定仍旧缺少配置，或者和示例代码版本不一致
3. 让我来个完整的能运行的示例Demo。 ps：也都是从官方github拉的
4. 我演示的是4.0的demo，目前也够用了，**5.0的demo我是运行不起来，缺少jar包，有路过的大神知道怎么搞，求教**

### 2.2. tx-manager

> https://pan.baidu.com/s/1cLKAeE#list/path=%2Fsharelink974324822-625872931897976%2Ftx-manager&parentPath=%2Fsharelink974324822-625872931897976

1. 这个tx-manager直接从官方提供的网盘下载，我下的4.1版本的
2. 修改配置文件，eureka和redis都整成自己的
3. 直接java -jar tx-manager.jar 运行起来就可以了

### 2.3. SpringCloud Demo

> https://github.com/codingapi/springcloud-lcn-demo

1. 上述地址为4.0版本的demo，经试验可以使用
2. 我用mybatis-demo这个包做的试验，里面两个module，分别修改application.properties，只需要修改数据库mysql和eureka地址就行
3. 当然数据库别忘了建，建个test库，新建下列表用于测试

```
USE test;

DROP TABLE IF EXISTS `t_test`;

CREATE TABLE `t_test` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

