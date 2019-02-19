---
layout: post
title: Head First Servlet JSP 笔记 (3)
tags:
  - java
---

Head First Servlet JSP 笔记 (3)
=======
## 请求属性和请求分派
如果希望应用的其他组件接管全部或者部分请求,可以使用请求属性  
一个简单的例子: 一个应用从Servlet控制器开始,最后以JSP视图结束.控制器与模型通信,得到视图建立相应所需的数据.这些数据不应该在上下文或者会话中,因为这些数据只属于这个请求,所以我们放在请求作用域中.  
**用RequestDispatcher**  

```
request.setAttribute("name",value);
RequestDispatcher view = request.getRequestDispatcher("result.jsp");
view.forward(request,response);
```

* 从servletrequest得到Dispatcher: request.getRequestDispatcher("name.jsp");
	这里需要一个路径作为参数,要把请求转发给哪个资源,就要写下哪个资源的路径,如果前面有一个/,name容器就会再当前的目录下查找

* 从servletContext得到RequestDispatcher: getServletContext.getRequestDiapatcher("/name.jsp");
   这里的路径不能相对于当前资源,也就是说,必须要以/开头
  
* 在RequestDispatcher上调用forward()
   
### 属性作用域
  |  可访问性 | 作用域 | 适用于
---|--------|--------|------
context | Web应用的所有部分(jsp,servlet,Listener等)| 与应用的声明周期等价 | 希望应用共享的资源,包括数据库连接,JNDI查找名等
HttpSession会话| 访问这个特定会话的所有servlet和jsp,一个会话可以扩展到多个servlet | 会话可以通过编程撤销,也可以超时撤销 | 与客户有关的资源或数据
Request(线程安全) | 应用中能直接访问请求对象的部分(所有接受转发请求的servlet和JSP) | service()方法结束 | 模型信息从控制器传递到视图,或者传递给客户

## 跟踪客户的状态(跨多个请求)

* 对于同一个客户,httpsession可以跨多个请求    会话状态
* 换句话说,与一个客户的整个会话期间,httpSession会持久存储

### 客户需要一个唯一的会话ID
容器和客户通过cookie交换会话ID(最简单和常用的方式)  
**容器几乎会做所有的会话工作**  
在响应中发送一个会话cookie:

```
HttpSession session = request.getSession();  // 这个方法不只是创建一个session, 在请求上第一次调用这个方法时,会导致随响应发送一个cookie
```

就这么简单,余下的所有工作,都会自动发生

* 不用自己建立新的Httpsession对象
* 不必生成唯一的会话ID
* 不用自己建立cookie对象
* 不用把会话与cookie关联
* 不用在相应中设置cookie

从请求中得到会话ID  

```
HttpSession session = request.getSession();  // 与上一个方法完全一样
``` 

意味着: 请求一个包含会话id的cookie, 如果没有会话ID,name创建一个新会话,如果有,name就获取  

#### 怎么样确认这个会话是新创建还是已经存在的?

```
if (session.isNew()) {
	// new
} else {
	// old
}
```

另外: 如果实现了与会话相关的监听者接口,也可以从相应的回调中拿到会话  

	getSession(false)  // 这个方法要么是null,要么是已经存在的会话

### 如果客户禁用了cookie,那么**URL重写,是一条后路**  

假设有一个web页面,其中的每个链接都有一些额外的信息,附加到URL最后(会话ID), 当用户点击链接时,达到容器的请求会得到这个信息,从二查找匹配的会话  

#### 如果客户不支持cookie,就要告诉响应要对URL编码,这样才能起作用

```
public void doGet(HttpServletRequest req, HttpServletResponse resp) {
	resp.setContentType("text/html");
	PrintWriter out = response.getWriter();
	HttpSession session = request.getSession();
	out.println("<html><body>");
	out.println("<a href=\"" + response.encodeURL("someUrl") + \">clickme</a>");
	out.println("</body></html>");
}
```

一般来说,想要保证客户与容器之间保持会话,n那么在第一次请求的时候,就要保证两种方法都进行尝试.  

### 使用SendRedirect()的URL重写
吧请求重定向到一个URL,但是还想保持这个会话 ::  response.encodeRedirectURL("");  

>要点
>
>* 在写至响应的HTML中,URL重写会把会话ID增加到URL的后面
>* 会话ID作为请求URL最后的"额外"信息再返回
>* 如果客户不接受cookie,那么重写会自动发生,前提是必须显式的对URL编码
>* 要编码一个URL,需要response.encodeURL("url");
>* 没有办法对静态页面进行重写,如果想依赖会话,那么就必须要动态生成页面

### 关键的HTTPSession接口
  | 做什么| 你用来做什么
---|---|-----
getCreationTime()| 返回第一次创建的时间| 得出这个会话的年龄,可能需要限定这个会话的时限
getLastAccessTime()| 返回容器最后一次得到这个会话的请求的时间| 看看客户有多久不在了,问问他要不要继续,或者直接删除会话(invalidate)
setMaxInActiveIntevel | 指定这个会话的最大请求间隔时间| 如果超过了这个时间,就会被自动撤销,清除无用的资源
getMaxInActiveIntevel| 获取最大请求间隔| 判断一个会话在撤销之前还有多久的寿命
invalidate()| 结束会话|主动删除会话

### 在DD中配置会话超时

```
<web-app>
	<session-config>
		<session-timout>15</session-timeout> // 在这个app中所有的会话超时都是15分钟
	</session-config>
</web-app>
```

设定一个特定的会话超时
	
	session.setMaxInActiveIntevel(20*60); // 这里是秒为单位的,(20)分钟
	
	
### 用cookie做一些其他事情
可以在servlet和客户之间做一些string的键值对的传递

* 创建一个新的cookie
	Cookie cookie = new Cookie("username", user);
* 设置cookie在客户端存货多久
	cookie.serMaxAge(30 * 60); // (秒)
* cookie发送给客户
	response.addCookie(cookie);
* 从客户请求得到cookie
	Cookie[] cookies = request.getCookies();
	
简单的cookie定制
```
public void doPost(HttpServletRequest request, HttpServletResponse response) {
	
	response.setContenttype("text/html");
	String name = request.getParameter("name");
	Cookie cookie = new Cookie("name": name);
	cookie.setMaxAge(30 * 60);
	response.addCookie(cookie);
	RequestDispatcher view = request.getRequestDispatcher("cookieTest.jsp");
	view.forward(request, response);

}

```
	
	
### 会话生命周期

里程碑 | 事件和监听者类型
------|------------
创建会话/撤销会话| HTTPSessionListener
增加/删除/替换一个属性| HttpSessionBindingListener
迁移: 会话准备钝化/已经激活| HttpSessionEvent/HttpSessionActivationListener

## 会话迁移
**只有HttpSesion会从一个VM迁移到另一个VM**  
每个VM有一个ServletContext,每个VM的servlet有一个config
所以在分布式应用中,除了会话意外的其他元素都会在另一个VM上复制  

### 具体的会话迁移

* 容器建立一个新会话 #343,cookie在响应中发回给客户
* 客户发起第二个请求,负载均衡服务器决定吧会话迁移到VM2, VM2的容器得到请求,发现绘画在VM1中
* 会话迁移到VM2中,在VM1中钝化(消失),并且在VM2上面激活
* 容器建立一个新线程,并且把请求与刚才的会话关联

### HttpsessionActivationListener使属性做好准备大搬家
如果属性都是Serializable,name不需要这个监听者,实际上,大部分会话都不需要,但是它就在这里"恭候",随时可用  
