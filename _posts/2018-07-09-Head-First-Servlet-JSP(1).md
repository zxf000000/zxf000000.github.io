---
layout: post
title: Head First Servlet JSP 笔记 (1)
tags:
  - java
---

Head First Servlet JSP 笔记 (1)
==========================
之前做了个小项目上到了Appstore,因为没有后端所以一直很自卑

# 开始正题

## 为什么使用Servlet&JSP
这本书可能年代比较久远了,书上面写的是**WEB应用炙手可热**,原来的静态页面满足不了大家的需求了,现在需要将一个web网站交付成一个web应用

我记得我在上大学的时候(08年左右),那个时候学习制作网站还是用的三剑客:PS+DW+Flash,做出来的网站很炫酷,但是貌似资源配置和管理之类的,都是有一个很复杂的比较难操作的后台管理器在做

现在需要HTML + CSS + Java + JSP + Servlet 来制作一个完整的web应用,来完成更复杂和完整的功能

### 每个人都想有个网站
### WEB服务器做什么
接受客户请求,然后向客户返回一些东西,返回的东西可能是图片,文字,语音,或者pdf文档,如果没有这个资源,那么可能就要返回404页面

**服务器**可能是物理主机(硬件),也可能是web服务应用(软件)
![服务器](https://upload-images.jianshu.io/upload_images/1769194-6bfc0a3b6e49195a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### WEB客户做什么?
**客户**是指人类用户或者浏览器应用
客户和服务器都知道**HTML**和**HTTP**  
**HTTP**是两者通信的协议
服务器使用HTTP向用户发送HTML
### HTML速成指南
```
<!-- --> // 注释
<a>		 // 锚点(通常用来放一个超链接)
<align>  // 内容对齐方式
<body>   // 定义文档的体边界
<br>     // 行分隔
<center> // 内容居中
<form>   // 定义一个表单
<h1>  	  // 一级标题
<head>   // 定义文档首部的边界
<html>   // html文档的边界
<input type> // 表单中定义一个输入组件-----(textfield, checkbox, button...)
<p>      // 一个段落
<title>  // html文档的标题
```
### 什么是HTTP协议?
**HTTP**是**TCP/IP**的上层协议  
书中简易解释:  
**TCP**确保一个网络节点向另一个网络节点发送的文件能作为一个完整的文件到达目的地,尽管在发送过程中这个文件可能分成小块传输  
**IP**是一个底层协议,负责吧数据包沿路移动/路由到目的地,
**HTTP**则是一个网络协议,有一些web的特性,需要依赖TCP/IP完成从一处到另一处的完整请求和相应,HTTP会话结构是一个简单的请求/相应序列: 浏览器发出请求,服务器做出响应

#### 请求的关键要素

* HTTP方法 (完成的动作)
* 要访问的页面 (URL)
* 表单参数 (如方法参数)

#### 响应的关键要素

* 状态码 (表明是否成功)
* 内容类型
* 内容

#### GET请求
![](https://upload-images.jianshu.io/upload_images/1769194-2faf2ea261a4a46d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### POST请求
![](https://upload-images.jianshu.io/upload_images/1769194-a3f983cc3b504002.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### MIME类型
![](https://upload-images.jianshu.io/upload_images/1769194-cff06c0a1a820664.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### URL
即: 统一资源定位符, 包括:  
**协议**: http://  
**服务器**: www.baidu.com  (ip地址或者域名)
**端口**: :80  
**路径**: /beer/home/...  
**资源名**: index.html  
连起来就是一个url
**TCP端口就是一个数字**

1. 0~1023端口已经被系统占用(包括80)
2. 建议用3000以上的端口, tomcat和jenkins等默认用8080端口

![](https://upload-images.jianshu.io/upload_images/1769194-59d8f9982375c5f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Web服务器自己不做的两件事情(就是需要其他辅助程序做的事)

* 动态内容
* 在服务器上面保存数据

##### 如果不按照Java术语来说,Web服务器辅助应用就是CGI程序(Common Gateway Interface 公共网关接口)  
![](https://upload-images.jianshu.io/upload_images/1769194-643ae5c8c1fb5fe3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####Servlet 和 CGI 都是web服务器的辅助应用


##Servlet介绍
建立一个Java项目,目录如下:

* project
	* src
	* classes
	* etc

建立一个MyServlet.java文件,放在src文件夹下面  

	public class MyServlet extends HttpServlet {  
 
		public void doGet(HttpServletRequest request, HttpServletResponse) throws IOException {  
			// 以上是标准的Servlet声明,讲清楚这个东西大约需要400页(书中说的)  
		 
			// ..... 
		}    
	}  
	
创建一个部署文件(DD deployment descriptor), 名字为web.xml 放在etc目录中

```
<web-app ...>
<Servlet>
	<Servlet-name>DemoServlet</Servlet-name>  // 这一行要与下面一行的name一样
	<Servlet-class>MyServlet</Servlet-class>  // 每一个web应用都包含一个web.xml
											
</Servlet>								        // 每一个web.xml可以配置多个Servlet
<Servlet-mapping>                            // servlet-class 是servlet类名
	<Servlet-name>DemoServlet</Servlet-name>
	<url-pattern>/Demo</url-pattern>
</Servlet-mapping>
</web-app>
```
在现有的tomcat目录下创建这个目录

* tomcat
	* webapp
		* Demo1
			* WEB-INF --- web.xml
				* classes   --- MyServlet.class  
				

编译

```
javac -classPath /tomcat/lib/Servlet-api.jar -d classes src/MyServlet.java
```
把MyServlet.class 复制到classes文件夹,把web.xml复制到WEB-INF文件夹

启动tomcat
cd到tomcat根目录  
```
bin/startup.sh
```  

打开浏览器,访问  

http://localhost:8080/Demo1/Demo  

接下来,每次重新部署都要关闭tomcat  
```
bin/shutdown.sh  
```  

**直接在Servlet的doGet方法中返回HTML代码不太好看,那么就需要用到JSP----在HTML中嵌入Java代码**  
JSP开发人员只需要知道HTML的写法,再加上简单的调用一些Java的函数,就可以完成任务  

## Web应用体系
### 什么是容器?
Servlet没有main方法,他是受另外一个Java应用的,那么这个应用就是容器(Container)  
tomcat就是这样的一个容器,Web服务器接受到一个**指向Servlet**的请求的时候,并不会把这个请求直接交给servlet本身,而是交给部署servlet的容器,由容器向servlet提供HTTP请求和响应,而且要由容器调用servlet的方法(doGet和doPost等)  
#### 如果没有容器,只有Java怎么办?
必须在J2SE中实现与servlet相同的效果---  

* 创建与服务器的Socket连接,并为这个Socket创建一个监听者,
* 创建一个线程管理器,实现安全,对日志之类的过滤,JSP支持,内存管理等

### 容器能提供什么?

* 通信支持  
	不需要自己建立连接,(创建Socket,监听端口,创建流等)
* 生命周期管理 (控制servlet的整个声明周期,资源回收等)
* 多线程支持 (自动为每个servlet分配一个线程,在HTTP方法完成之后,回收线程,不需要考虑线程的安全性)
* 声明方式实现安全
* JSP支持

#### 容器处理Servlet请求的流程

1. 用户点击一个链接指向一个servlet  
2. 容器"看出来"这是一个servlet的请求,创建两个对象__HttpServletRequest__和**HttpServletResponse**
3. 容器根据请求中的URL找到对应的servlet,为这个请求创建或者分配一个线程,并把请求个响应对象传递给这个servlet线程
4. doGet等方法生成动态页面,并且把这个页面塞到响应中期<font>
5. 线程结束,容器吧响应对象转换成一个HTTP响应,发回给客户,并且删除请求和响应  


#### Servlet有三个名字---文件路径名/类名(以及包名)/公共url名(用户访问的名字)

### Servlet和JSP中的MVC

* 视图(JSP)
* 控制器(Servlet)
* 模型(实际的业务处理模块,DB等)

任务|Web服务器|容器|servlet
---|---|---|---
创建请求和响应对象| |在开始线程之前创建| 
调用Service方法| | service()方法调用doGet()等方法
开启一个新的线程来处理请求| |开始一个servlet线程| 
把响应对象转换为一个HTTP响应| |容器根据响应对象中的数据生成HTTP响应| 
了解HTTP|通过HTTP与客户(浏览器)对话| | 
把HTML增加到响应对象| | | 这是提供给客户的动态内容
有响应对象的一个引用| |容器把他交给servlet|用他打印响应
在部署描述文件中查找URL| |找到响应对象的适当servlet| 
删除请求和响应对象| |servlet一旦结束就删除请求和响应对象| 
协调生成动态内容| 知道如何转发到容器| 知道调用谁| 
管理声明周期| | 调用服务方法| 
名字与部署描述文件中的相应内容匹配| | | 任何公共类  

#### J2EE如何集成这一切?
![](https://upload-images.jianshu.io/upload_images/1769194-174a48fea6ace881.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## MVC迷你教程
### 大纲

* 构建一个web应用需要部署的目录结构可能包含
	1. 静态内容
	2. JSP页面
	3. servlet类
	4. 部署描述文件(DD)
	5. 标记库
	6. jar文件
	7. java类文件  

	描述如何避免通过HTTP访问资源文件

* 描述以下各个部署文件元素代表的语义:  
	error-page, init-param, mime-mapping, servlet, servlet-class, servlet-mapping, url-pattern, servlet-name和 welcome-file

* 为以下各个部署描述文件元素构造正确的结构
	error-page, init-param, mime-mapping, servlet, servlet-class, servlet-mapping, url-pattern, servlet-name和 welcome-file
	
### 构造一个真正的(小)web应用

1. 分析用户视图
2. 创建用于开发这个项目的开发环境
3. 创建用于部署这个项目的部署环境
4. 对Web应用的各个组件完成迭代的开发和测试  
	* 构建和测试用户最初请求的HTML表单
	* 构建servlet第一个版本,打印接受到的参数
	* 构建一个模型测试类
	* 用模型来生成请求数据
	* 构建JSP,将请求分配给JSP
	
	JSP如下  
	
```
<% page import="java.util.*" %>
<html>
<body>
<h1 align="center">Page Title</h1>
<p>
<%
	List styles = (List)request.getAttributes("styles");
	Iterator it = styles.iterator();
	while (it.hasNext) {
		out.println("...");
	}
%>
</p>
</body>
</html>
```

servlet中的核心代码

```
request.setAttribute("styles", result); // 为request设置一个属性
RequestDispatcher view = request.getRequestDispatcher("result.jsp"); // 为jsp实例化一个请求分派器
view.forward(request, response); // 使用请求分派器要求容器准备好jsp,向jsp发送请求和响应
```

#### 目录结构
	
* MyProject
	* app1
		* etc
			* web.xml
		* lib
		* src
			* com
				* example
					* web
					* model
		* classes
			* com
				* example
					* web
					* model
		* web
			* JSP
			* HTML
	* app2


## 作为Servlet
### servlet技术模型

* 对于每一种HTTP方法,描述该方法的用途,以及该HTTP方法协议的技术特性,并列出客户会由于那些原因使用某方法,明确HTTP方法对应的HttpServlet方法.
* 使用HttpRequest方法,填写代码从众获取HTML表单参数,获取HTTP请求首部信息,或者从请求获取cookie
* 使用HttpServletResponse接口,编写代码设置一个HTTP响应首部,设置响应内容类型,为响应获得一个文本/二进制流,把一个HTTP重定向一个URL,或者响应的cookie
* 描述servlet声明周期行为

### Servlet生命周期
![](https://upload-images.jianshu.io/upload_images/1769194-59a5ab06045f456d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 声明周期的三大重要时刻

序号 | 何时调用 | 作用 | 是否覆盖
-----|-------|-------|------
init()| servlet实例呗创建后,并在为客户提供服务之前 | 使你在为客户提供服务之前有机会进行初始化 | 可能,如果有初始化代码,如得到一个数据库连接,就要覆盖servlet中的init()
service() | 当客户第一个请求到来时,容器就会开始一个新的线程,或者从线程池分配一个线程,调用service方法| 这个方法查看请求方法,并在servlet上调用对应的方法(doGet,doPost等) | 应该覆盖不应该直接覆盖service(),而应该doGet(),doPost()等
doGet()和doPost() | service根据HTTP方法来决定调用哪个| 在这里处理请求,写下自定义代码 | 至少要覆盖其中之一

**service()方法总是在自己的栈中调用**  
__每一个请求都在一个单独的线程中运行__


### ServletConfig对象

* 每一个servlet都有一个servletConfig对象
* 用于向servlet传递部署时信息,而你不想吧这个信息硬编码到servlet类中
* 用于访问servletContext
* 参数在部署描述文件中配置

### ServletContext

* 每一个web应用都有一个ServletContext
* 用于访问Web应用参数
* 相当于应用的一个公告栏,可以在这里放属性,应用的其他部分可以访问这些属性
* 用于得到服务器的信息,如容器的名字,版本等

### 非幂等请求
幂等就是可以一遍一遍的请求,不会出现不同的结果,__非幂等__就是每次请求都有可能出现不同的结果,POST请求就是非幂等请求,因为他会修改服务器的数据

**一个参数可以有多个值**  
如果不确定有几个值,调用 request.getParameterValues("name")  
**除了参数,还能从请求中得到**  

* getHeader() // 获得浏览器的信息
* request.getCookie() // 获取cookie
* request.getSession() // 与客户相关的会话
* request.getMethod() // 请求的方法
* request.getInputStream() // 请求的输入流

### 使用响应完成I/O

```
public class CodeReturn extends HttpServlet {
	public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException,ServletException {
		response.setContentType("application/jar");
		ServletContext ctx = getServletContext();
		InputStream in = ctx.getResourceAsStream("/code.jar"); // 将这一个资源作为一个输入流
		int read = 0;
		byte[] bytes = new byte[1024];
		
		OutputStream out = response.getOutputStream();
		while ((read = is.read(bytes)) != -1) {
			out.write(bytes,0,read);
		}
		
		out.flush;
		out.close;
		
	}
}
```

**对于输出数据,有两种类型,字符/字节**  

#### 响应首部
可以设置-----**setHeader**  
可以增加-----**addHeader**  

#### 不想自己处理响应的时候
**重定向URL**  
	
	response.setRedirect("//重定向url")  // 这个url是相对的
	

请求分派: -----将请求直接转给JSP来处理(另一个web应用)  --浏览器看不到  
重定向: -----直接转发给另外一个URL---浏览器可以看到













