# ActiveMQ

## 1. activemq 的几种通信方式

**publish(发布)-subscribe(订阅)(发布-订阅方式)**

发布/订阅方式用于多接收客户端的方式.作为发布订阅的方式，可能存在多个接收客户端，并且接收端客户端与发送客户端存在时间上的依赖。一个接收端只能接收他创建以后发送客户端发送的信息。作为 subscriber ,在接收消息时有两种方法，destination 的 receive 方法，和实现 message listener 接口的onMessage 方法。

**p2p(point-to-point)(点对点)**

p2p 的过程则理解起来比较简单。它好比是两个人打电话，这两个人是独享这一条通信链路的。一方发送消息，另外一方接收，就这么简单。在实际应用中因为有多个用户对使用 p2p 的链路。在 p2p 的场景里，相互通信的双方是通过一个类似于队列的方式来进行交流。和前面 pub-sub 的区别在于一个 topic 有一个发送者和多个接收者，而在 p2p里一个 queue 只有一个发送者和一个接收者。

## 2.**activemq 如果数据提交不成功怎么办(消息丢失)**

**publish(发布)-subscribe(订阅)方式的处理**

发布订阅模式的通信方式， 默认情况下只通知一次， 如果接收不到此消息就没有了。 这种场景只适用于对消息送达率要求不高的情况。 如果要求消息必须送达不可以丢失的话， 需要配置持久订阅。 每个订阅端定义一个 id，**<property** name="clientId" 在订阅是向 activemq 注册。 发布消息**<property** name="subscriptionDurable" value="true"**/>**和接收消息时需要配置发送模式为持久化template.setDeliveryMode(DeliveryMode.**PERSISTENT**);。 此时如果客户端接收不到消息， 消息会持久化到服务端(就是硬盘上)， 直到客户端正常接收后为止。

**p - p(点对点)方式的处理**

点对点模式的话， 如果消息发送不成功此消息默认会保到 activemq 服务端直到有消费者将其消费， 所以此时消息是不会丢失的。

## 3.如何解决消息重复问题

所谓消息重复,就是消费者接收到了重复的消息,一般来说我们对于这个问题的处理要把握下面几点, 

- ①.消息不丢失(上面已经处理了) 

- ②.消息不重复执行

一般来说我们可以在业务段加一张表,用来存放消息是否执行成功,每次业务事物
commit 之后,告知服务端,已经处理过该消息,这样即使你消息重发了,也不会导致重复处理。

大致流程如下:
		业务端的表记录已经处理消息的 id,每次一个消息进来之前先判断该消息
是否执行过,如果执行过就放弃,如果没有执行就开始执行消息,消息执行完成之
后存入这个消息的 id。

## 4.大量的消息每页被消费，能否发生 oom 异常？

可以控制每个消息队列中数据的大小，不允许无线填充数据，避免该队列多大，导致过度消耗系统资源问题； 可以控制队列的内存大小；

## 5.activeMQ 发送消息的方式有哪些？

消息通信的基本方式有两种：

1. 同步方式

两个通信应用服务之间必须要进行同步，两个服务之间必须都是正常运行的。发送程序和接收程序都必须一直处于运行状态，并且随时做好相互通信的准备。

发送程序首先向接收程序发起一个请求，称之为发送消息，发送程序紧接着就会堵塞当前自身的进程，不与其他应用进行任何的通信以及交互，等待接收程序的响应，待发送消息得到接收程序的返回消息之后会继续向下运行，进行下一步的业务处理。

2. 异步方式

两个通信应用之间可以不用同时在线等待，任何一方只需各自处理自己的业务，比如发送方发送消息以后不用登录接收方的响应，可以接着处理其他的任务。也就是说发送方和接收方都是相互独立存在的，发送方只管方，接收方只能接收，无须去等待对方的响应。Java 中 JMS 就是典型的异步消息处理机制，JMS 消息有两种类型：点对点、发布/订阅。

## 6. activeMQ 如何调优

1. 使用非持久化消息;

2. 需要确保消息发送成功时使用事务来将消息分批组合.

```java
public void sendTransacted() throws JMSException {

		ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory()
		Connection connection = cf.createConnection();
		connection.start();
		Session session = connection.createSession(true,Session.SESSION_TRANSACTED);

		Topic topic = session.createTopic("Test.Transactions");

		MessageProducer producer = session.createProducer(topic);

		int count = 0;
		for (int i = 0; i < 1000; i++) {

			Message message = session.createTextMessage("message " + i);

			producer.send(message);

			if (i != 0 && i % 10 == 0) {
				session.commit();

			}

		}

	}

	public void sendNonTransacted() throws JMSException {

		ActiveMQConnectionFactory cf = newActiveMQConnectionFactory();

		Connection connection = cf.createConnection();

		connection.start();

		// create a default session (no transactions)
		Session session = connection.createSession(false, Session.AUTO_ACKNOWELDGE);

		Topic topic = session.createTopic("Test.Transactions");

		MessageProducer producer = session.createProducer(topic);

		int count = 0;
		for (int i = 0; i < 1000; i++) {
			Message message = session.createTextMessage("message " +i);
			producer.send(message);
		}

	}
```

## **7.什么是死信队列？**

如果你想在消息处理失败后，不被服务器删除，还能被其他消费者处理或重试，可以关闭 AUTO_ACKNOWLEDGE，将 ack 交由程序自己处理。那如果使用了 AUTO_ACKNOWLEDGE，消息是什么时候被确认的，还有没有阻止消息确认的方法？有！

消费消息有 2 种方法，一种是调用 consumer.receive()方法，该方法将阻塞直到获得并返回一条消息。这种情况下，消息返回给方法调用者之后就自动被确认了。另一种方法是采用 listener 回调函数，在有消息到达时，会调用listener 接口的 onMessage 方法。在这种情况下，在 onMessage 方法执行完毕后，消息才会被确认，此时只要在方法中抛出异常，该消息就不会被确认。那么问题来了，如果一条消息不能被处理，会被退回服务器重新分配，如果只有一个消费者，该消息又会重新被获取，重新抛异常。就算有多个消费者，往往在一个服务器上不能处理的消息，在另外的服务器上依然不能被处理。难道就这么退回--获取--报错死循环了吗？

在重试 6 次后，ActiveMQ 认为这条消息是“有毒”的，将会把消息丢到死信队列里。如果你的消息不见了，去 ActiveMQ.DLQ 里找找，说不定就躺在那里。