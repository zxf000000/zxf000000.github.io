---
layout: post
title: Head First Servlet JSP 笔记 (2)
tags:
  - java
---

Head First Servlet JSP 笔记 (2)
==========
## 五 属性和监听
### 在DD中配置一些全局的属性,而不是硬编码到servlet中
解决方法: 初始化参数  
除了把请求参数传递给doGet个doPost中,还可以设置servlet初始化参数

在Servlet配置文件web.xml中

```
<init-param>
	<param-name>name</param-name>
	<param-value>value</param-value>
</init-param>

```

在servlet代码中

```
getServletConfig().getInitParameter("name");
```

**在Serlet初始化之前不能使用初始化参数**  
容器初始化一个servlet时,会为这个servlet创建一个唯一的ServletConfig,容器从DD"读"出Servlet初始化参数,并且将参数传递给ServletConfig,然后把ServletConfig传递给Init()方法  

![](https://upload-images.jianshu.io/upload_images/1769194-911fc0ea18be8345.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Servlet初始化参数只能读取一次,就是在初始化Servlet的时候**  
如果想要JSP拿到初始化参数,那么就要在Servlet中把参数设置成请求属性,这样JSP才能拿到,当然,JSP能拿到Servlet愿意给他的任何数据  

### 如果想不通过设置属性,使JSP能拿到类似于"全局"的参数,就要设置 **上下文初始化参数**

在web.xml中配置

```
<context-param>
	<param-name>name</param-name>
	<param-value>value</param-value>
</context-param>
```

在servlet代码中国

```
getServletContext.getInitParameter("");
```

#### Servlet初始化参数和上下文初始化参数的区别

上下文初始化参数 | servlet初始话参数
-----------------|--------------
部署描述文件  | 
在<web-app>元素内但是不在特定的Servlet元素中| 每个特定的servlet元素中
servlet代码 | 
getServletConfig | getServletContext

每个servlet一个servletConfig元素  
每个web应用一个ServletContext元素

**如果应用是分布式的,那么每一个JVM都有一个ServletContext参数**

#### WEB应用初始化
* 容器读取DD,为每个<context-param>创建一个名/值String对
* 容器创建ServletContext的一个新实例
* 容器为ervletContext提供上下文初始化的ming/值参数对
* 在WEB应用部署的各个serlet和JSP都能访问同样的servletContext


### 如果一个WEB应用想要初始化一个数据库DataSource呢?
------使用ServletContextListener  
一个单独的对象,他能做到  

* 上下文初始化时能得到通知,
	* 从ServletContext上下文得到参数
	* 使用初始化参数名查找简历一个数据库连接
	* 把数据库连接存储为一个属性,使Web应用的各个部分都能访问
* 上下文撤销时能得到通知,
	* 关闭数据库连接

**ServletContextListener类**

```
import javax.servlet.*

public class MyServletContextListener implements ServletContextListener {
	
	public void contextInitialized(ServletContextEvent event) {
		// 初始化数据库连接的代码
		
	}
	
	public void contextDestoryed(ServletContextEvent event) {
		// 关闭数据库连接的代码
	}
}
```

**在DD文件中配置监听类**,这样容器就能拿到它,并且能做一些事情  

```
<listener>
	<listener-class>
		com.example.MyServletContextListener    // 这个监听者配置代码要放在servlet代码外面,因为这个listener要在任何servlet初始化之前进行初始化
	</listener-class>
</listener>
```

### 只要是声明周期中的事件,总有一个对应的监听者  
除了上下文监听者,还有上下文属性监听,servlet请求和属性,以及HTTP会话属性相关的事件  

场景|监听者接口|事件类型
---|---------|-------
你想知道一个web应用上下文是否增加,删除或者替换了一些属性|javax.servlet.ServletContextAttributeListener<br> attributeAdded <br>  attributeRemoved <br> attributeReplaced | ServletContextAttributeEvent
你想知道有多少个并发用户,也就是说,你想跟踪活动会话 | javax.servlet.HttpSessionListener<br>sessionCreated<br>sessionDestroyed|HttpSessionEvent
每次请求你都想知道,以便建立请求日志| javax.servlet.ServletRequestListener<br>requestInitilized<br>requestDestoryed|ServletReqeustListener
你想知道什么时候增加,删除,替换一个请求属性|javax.servlet.servletRequestAttributeListener<br>attributeAdded<br>attributeRemoved<br>attributeReplaced|ServletRequestAttributeListener
你有一个属性类,(这个类表示的对象将被放在一个属性中),而且你希望这个类型的对象绑定到一个会话或从会话中删除时得到通知|javax.servlet.http.HttpSessionBindingAttributeListener<br>attributeAdded<br>attributeRemoved<br>attributeReplaced|HttpSessionBindingListener
你想知道会否创建或撤销了一个上下文|javax.servlet.ServletContextListener<br>contextInitialized<br>contextDestoryed|ServletContextEvent
你有一个属性类,而且希望此类绑定的会话迁移到另外一个JVM时得到通知|javax.servlet.http.HttpSessionActivationListener<br>sessionDidActivite<br>sessionWillPassivate|HttpSessionEvent

#### HttpSessionbindingListener,关于这个监听,可以这么想
假设Dog是一个客户类,每个活动的实例表示一个客户的信息,实际的数据存储在底层数据库中,你使用数据库信息来填充Customer对象的字段,但是问题是,你怎么保持数据库和Customer信息同步呢?另外,什么时候让他同步?你知道,只要有一个Customer增加到会话,就应该用数据库中响应的记录去刷新Customer对象的字段,所以ValueBound()方法就像是敲钟提示"用数据库中的字段来加载我",而对于valueUnbound() 而是说,"用customer中的字段更新数据库"


### 到底什么是属性?
属性就是一个对象,绑定到了另外三个servletApi对象servletContext,servletrequest,或者HttpSession

### 属性不是参数

 | 属性 | 参数
---|---|---
类型|应用/上下文<br>请求<br>会话 | 应用/上下文初始化参数<br>请求参数<br>servlet初始化参数(没有会话参数一说) 
设置方法|setAttribute(String name, Object value) | 不能设置应用和servlet初始化参数 ,他们都在DD中设置
返回类型| Object| String
获取方法| getAttribute(String name) | getInitparameter(String name)


### ServletContext并不是线程安全的
要解决这个问题,要对上下文加锁

```
synchronize(getServletContext()) {
	getServletContext().setAttribute("name",value);
	..........
}  // 因为有了同步锁,所以每次这个上下文只能被一个线程访问,就安全了
```

### HttpSession同样也不是线程安全的!
同样的对session加锁来保护属性

```
synchronize(session) {
	session.setAttribute("name",value);
	..........
}  // 因为有了同步锁,所以每次这个会话只能被一个线程访问,就安全了
```

### 只有请求属性和局部变量是线程安全的!!


 
 
											          

