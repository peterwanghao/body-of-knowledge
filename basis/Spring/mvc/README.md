# 1 Spring MVC

## Spring MVC的优点

1）它是基于组件技术的.全部的应用对象,无论控制器和视图,还是业务对象之类的都是 java组件.并且和Spring提供的其他基础结构紧密集成. 

 2）不依赖于Servlet API(目标虽是如此,但是在实现的时候确实是依赖于Servlet的)  

 3）可以任意使用各种视图技术,而不仅仅局限于JSP  

 4）支持各种请求资源的映射策略  

 5）它应是易于扩展的



## SpringMVC流程

 1）用户发送请求至前端控制器DispatcherServlet。 

 2）DispatcherServlet收到请求调用HandlerMapping处理器映射器。 

 3）处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。  

 4）DispatcherServlet调用HandlerAdapter处理器适配器。  

 5）HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。  

 6）Controller执行完成返回ModelAndView。  

 7）HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。  

 8）DispatcherServlet将ModelAndView传给ViewReslover视图解析器。  

 9）ViewReslover解析后返回具体View。  

 10）DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。  

 11）DispatcherServlet响应用户。



## SpringMvc的控制器是不是单例模式,如果是,有什么问题,怎么解决？

是单例模式。单例的原因有二：

1、为了性能。

2、不需要多例。

所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的。

解决方案是：

1、不要在controller中定义成员变量。

2、万一必须要定义一个非静态成员变量时候，则通过注解@Scope("prototype")，将其设置为多例模式



## 简单介绍下springMVC和struts2的区别有哪些?

1）springmvc的入口是一个servlet即前端控制器，而struts2入口是一个filter过虑器。

 2）springmvc是基于方法开发(一个url对应一个方法)，请求参数传递到方法的形参，可以设计为单例或多例(建议单例)，struts2是基于类开发，传递参数是通过类的属性，只能设计为多例。

 3）Struts采用值栈存储请求和响应的数据，通过OGNL存取数据，springmvc通过参数解析器是将request请求内容解析，并给方法形参赋值，将数据和视图封装成ModelAndView对象，最后又将ModelAndView中的模型数据通过reques域传输到页面。Jsp视图解析器默认使用jstl。



## @RequestMapping注解用在类上面有什么作用？

是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。



## SpringMVC怎么样设定重定向和转发的？

在返回值前面加"forward:"就可以让结果**转发**,譬如"forward:user.do?name=method4" 

在返回值前面加"redirect:"就可以让返回值**重定向**,譬如"redirect:http://www.baidu.com"

转发在服务器端完成的；重定向是在客户端完成的 
转发的速度快；重定向速度慢 
转发的是同一次请求；重定向是两次不同请求 
转发不会执行转发后的代码；重定向会执行重定向之后的代码 
转发地址栏没有变化；重定向地址栏有变化 
转发必须是在同一台服务器下完成；重定向可以在不同的服务器下完成



## 怎么样把ModelMap里面的数据放入Session里面？

可以在类上面加上@SessionAttributes注解,里面包含的字符串就是要放入session里面的key



## SpringMvc怎么和AJAX相互调用的？

通过Jackson框架就可以把Java里面的对象直接转化成Js可以识别的Json对象。



## 当一个方法向AJAX返回特殊对象,譬如Object,List等,需要做什么处理？

要加上@ResponseBody注解

