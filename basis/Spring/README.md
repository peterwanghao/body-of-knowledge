# 第6节 Spring

**在Java中依赖注入有以下三种实现方式：**

1. 构造器注入
2. Setter方法注入
3. 接口注入



**BeanFactory和ApplicationContext有什么区别？**

BeanFactory 可以理解为含有bean集合的工厂类。BeanFactory 包含了种bean的定义，以便在接收到客户端请求时将对应的bean实例化。

BeanFactory还能在实例化对象的时生成协作类之间的关系。此举将bean自身与bean客户端的配置中解放出来。BeanFactory还包含 了bean生命周期的控制，调用客户端的初始化方法（initialization methods）和销毁方法（destruction methods）。

从表面上看，application context如同bean factory一样具有bean定义、bean关联关系的设置，根据请求分发bean的功能。但applicationcontext在此基础上还提供了其他的功能。

1. 提供了支持国际化的文本消息
2. 统一的资源文件读取方式
3. 已在监听器中注册的bean的事件

以下是四种较常见的 ApplicationContext 实现方式：

1、ClassPathXmlApplicationContext：从classpath的XML配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中

```java
ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);   
```


2、FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。
```java
ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”); 
```

3、XmlWebApplicationContext：由Web应用的XML文件读取上下文。

4、AnnotationConfigApplicationContext(基于Java配置启动容器)



**Spring Bean的作用域之间有什么区别？**

Spring容器中的bean可以分为5个范围。所有范围的名称都是自说明的，但是为了避免混淆，还是让我们来解释一下：

1. singleton：这种bean范围是默认的，这种范围确保不管接受到多少个请求，每个容器中只有一个bean的实例，单例的模式由bean factory自身来维护。
2. prototype：原形范围与单例范围相反，为每一个bean请求提供一个实例。
3. request：在请求bean范围内会每一个来自客户端的网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
4. Session：与请求范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
5. global- session：global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果 你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。

全局作用域与Servlet中的session作用域效果相同。



**Spring 框架中都用到了哪些设计模式？**

Spring框架中使用到了大量的设计模式，下面列举了比较有代表性的：

- 代理模式—在AOP和remoting中被用的比较多。
- 单例模式—在spring配置文件中定义的bean默认为单例模式。
- 模板方法—用来解决代码重复的问题。比如. [RestTemplate](http://howtodoinjava.com/2015/02/20/spring-restful-client-resttemplate-example/), JmsTemplate, JpaTemplate。
- 前端控制器—Spring提供了DispatcherServlet来对请求进行分发。
- 视图帮助(View Helper )—Spring提供了一系列的JSP标签，高效宏来辅助将分散的代码整合在视图里。
- 依赖注入—贯穿于BeanFactory / ApplicationContext接口的核心理念。
- 工厂模式—BeanFactory用来创建对象的实例