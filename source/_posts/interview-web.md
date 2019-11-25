---
title: 面试之---web相关
categories: interview
tags: 
  - 总结
  - web
date: 2018-09-16 21:04:51
description: 包含web等相关面试知识汇总
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](interview-web/interview.png)

> 持续维护java面试题系列博文，发现好的面试题会更新进来。总结的并不只是如何回答面试官，而是这道题涉及到的知识点，明白了相关知识点和面试官要问的点，回答起来就不是问题了。

## servlet

### Servlet的优点:

1，只需要启动一个操作系统进程以及加载一个JVM，大大降低了系统的开销

2，如果多个请求需要做同样处理的时候，这时候只需要加载一个类，这也大大降低了开销

3，所有动态加载的类可以实现对网络协议以及请求解码的共享，大大降低了工作量。

4，Servlet能直接和Web服务器交互，而普通的CGI程序不能。Servlet还能在各个程序之间共享数据，使数据库连接池之类的功能很容易实现。

> 补充：Sun Microsystems公司在1996年发布Servlet技术就是为了和CGI进行竞争，Servlet是一个特殊的Java程序，一个基于Java的Web应用通常包含一个或多个Servlet类。Servlet不能够自行创建并执行，它是在Servlet容器中运行的，容器将用户的请求传递给Servlet程序，并将Servlet的响应回传给用户。通常一个Servlet会关联一个或多个JSP页面。以前CGI经常因为性能开销上的问题被诟病，然而Fast CGI早就已经解决了CGI效率上的问题，所以面试的时候大可不必信口开河的诟病CGI，事实上有很多你熟悉的网站都使用了CGI技术。

### Servlet接口中有哪些方法及Servlet生命周期探秘

Servlet接口定义了5个方法，其中**前三个方法与Servlet生命周期相关**：

- **void init(ServletConfig config) throws ServletException**
- **void service(ServletRequest req, ServletResponse resp) throws ServletException, java.io.IOException**
- **void destory()**
- java.lang.String getServletInfo()
- ServletConfig getServletConfig()

**生命周期：** **Web容器加载Servlet并将其实例化后，Servlet生命周期开始**，容器运行其**init()方法**进行Servlet的初始化；请求到达时调用Servlet的**service()方法**，service()方法会根据需要调用与请求对应的**doGet或doPost**等方法；当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用Servlet的**destroy()方法**。**init方法和destroy方法只会执行一次，service方法客户端每次请求Servlet都会执行**。Servlet中有时会用到一些需要初始化与销毁的资源，因此可以把初始化资源的代码放入init方法中，销毁资源的代码放入destroy方法中，这样就不需要每次处理客户端的请求都要初始化与销毁资源。

### Servlet接口中有哪些方法？

Servlet接口定义了5个方法，其中前三个方法与Servlet生命周期相关： 

```java
void init(ServletConfig config) throws ServletException
void service(ServletRequest req, ServletResponse resp) throws ServletException, java.io.IOException
void destory()
java.lang.String getServletInfo()
ServletConfig getServletConfig() 
```

Web容器加载Servlet并将其实例化后，Servlet生命周期开始，容器运行其`init()`方法进行Servlet的初始化；请求到达时调用Servlet的`service()`方法，`service()`方法会根据需要调用与请求对应的`doGet`或`doPost`等方法；当服务器关闭或项目被卸载时服务器会将Servlet实例销毁，此时会调用Servlet的`destroy()`方法。

### 过滤器有哪些作用和用法？ 

Java Web开发中的过滤器（filter）是从Servlet 2.3规范开始增加的功能，并在Servlet 2.4规范中得到增强。对Web应用来说，过滤器是一个驻留在服务器端的Web组件，它可以截取客户端和服务器之间的请求与响应信息，并对这些信息进行过滤。当Web容器接受到一个对资源的请求时，它将判断是否有过滤器与这个资源相关联。如果有，那么容器将把请求交给过滤器进行处理。在过滤器中，你可以改变请求的内容，或者重新设置请求的报头信息，然后再将请求发送给目标资源。当目标资源对请求作出响应时候，容器同样会将响应先转发给过滤器，在过滤器中你可以对响应的内容进行转换，然后再将响应发送到客户端。

常见的过滤器用途主要包括：对用户请求进行统一认证、对用户的访问请求进行记录和审核、对用户发送的数据进行过滤或替换、转换图象格式、对响应内容进行压缩以减少传输量、对请求或响应进行加解密处理、触发资源访问事件、对XML的输出应用XSLT等。

和过滤器相关的接口主要有：`Filter`、`FilterConfig`和`FilterChain`。

### 监听器有哪些作用和用法？ 

Java Web开发中的监听器（listener）就是`application`、`session`、`request`三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件，如下所示： 

- `ServletContextListener`：对Servlet上下文的创建和销毁进行监听。 
- `ServletContextAttributeListener`：监听Servlet上下文属性的添加、删除和替换。 
- `HttpSessionListener`：对Session的创建和销毁进行监听。

> 补充：session的销毁有两种情况：1). session超时（可以在web.xml中通过<session-config>/<session-timeout>标签配置超时时间）；2). 通过调用session对象的invalidate()方法使session失效。

- `HttpSessionAttributeListener`：对Session对象中属性的添加、删除和替换进行监听。 
- `ServletRequestListener`：对请求对象的初始化和销毁进行监听。 
- `ServletRequestAttributeListener`：对请求对象属性的添加、删除和替换进行监听。

### 服务器收到用户提交的表单数据，到底是调用Servlet的doGet()还是doPost()方法？

HTML的`<form>`元素有一个`method`属性，用来指定提交表单的方式，其值可以是get或post。我们自定义的Servlet一般情况下会重写`doGet()`或`doPost()`两个方法之一或全部，如果是GET请求就调用`doGet()`方法，如果是POST请求就调用`doPost()`方法，那为什么为什么这样呢？我们自定义的Servlet通常继承自`HttpServlet`，`HttpServlet`继承自`GenericServlet`并重写了其中的`service()`方法，这个方法是Servlet接口中定义的。`HttpServlet`重写的`service()`方法会先获取用户请求的方法，然后根据请求方法调用`doGet()`、`doPost()`、doPut()、doDelete()等方法，如果在自定义Servlet中重写了这些方法，那么显然会调用重写过的（自定义的）方法，这显然是对模板方法模式的应用（如果不理解，请参考阎宏博士的《Java与模式》一书的第37章）。当然，自定义Servlet中也可以直接重写service()方法，那么不管是哪种方式的请求，都可以通过自己的代码进行处理，这对于不区分请求方法的场景比较合适。 

### Servlet中如何获取用户提交的查询参数或表单数据？

可以通过请求对象（`HttpServletRequest`）的`getParameter()`方法通过参数名获得参数值。如果有包含多个值的参数（例如复选框），可以通过请求对象的`getParameterValues()`方法获得。当然也可以通过请求对象的`getParameterMap()`获得一个参数名和参数值的映射（Map）。

### Servlet中如何获取用户配置的初始化参数以及服务器上下文参数？

可以通过重写`Servlet`接口的`init(ServletConfig)`方法并通过`ServletConfig`对象的`getInitParameter()`方法来获取`Servlet`的初始化参数。可以通过`ServletConfig`对象的`getServletContext()`方法获取`ServletContext`对象，并通过该对象的`getInitParameter()`方法来获取服务器上下文参数。当然，`ServletContext`对象也在处理用户请求的方法（如`doGet()`方法）中通过请求对象的`getServletContext()`方法来获得。



## jsp

### JSP有哪些内置对象、作用分别是什么

JSP有9个内置对象：

- `request`：封装客户端的请求，其中包含来自GET或POST请求的参数；
- `response`：封装服务器对客户端的响应；
- `pageContext`：通过该对象可以获取其他对象；
- `session`：封装用户会话的对象；
- `application`：封装服务器运行环境的对象；
- `out`：输出服务器响应的输出流对象；
- `config`：Web应用的配置对象；
- `page`：JSP页面本身（相当于Java程序中的this）；
- `exception`：封装页面抛出异常的对象。

### Request对象的主要方法有哪些

- `setAttribute(String name,Object)`：设置名字为name的request 的参数值
- `getAttribute(String name)`：返回由name指定的属性值
- `getAttributeNames()`：返回request 对象所有属性的名字集合，结果是一个枚举的实例
- `getCookies()`：返回客户端的所有 Cookie 对象，结果是一个Cookie 数组
- `getCharacterEncoding()` ：返回请求中的字符编码方式 = getContentLength() ：返回请求的 Body的长度
- `getHeader(String name)` ：获得HTTP协议定义的文件头信息
- `getHeaders(String name)` ：返回指定名字的request Header 的所有值，结果是一个枚举的实例
- `getHeaderNames()` ：返回所以request Header 的名字，结果是一个枚举的实例
- `getInputStream()` ：返回请求的输入流，用于获得请求中的数据
- `getMethod()` ：获得客户端向服务器端传送数据的方法
- `getParameter(String name)` ：获得客户端传送给服务器端的有 name指定的参数值
- `getParameterNames()` ：获得客户端传送给服务器端的所有参数的名字，结果是一个枚举的实例
- `getParameterValues(String name)`：获得有name指定的参数的所有值
- `getProtocol()`：获取客户端向服务器端传送数据所依据的协议名称
- `getQueryString()` ：获得查询字符串
- `getRequestURI()` ：获取发出请求字符串的客户端地址
- `getRemoteAddr()`：获取客户端的 IP 地址
- `getRemoteHost()` ：获取客户端的名字
- `getSession([Boolean create])` ：返回和请求相关 Session
- `getServerName()` ：获取服务器的名字
- `getServletPath()`：获取客户端所请求的脚本文件的路径
- `getServerPort()`：获取服务器的端口号
- `removeAttribute(String name)`：删除请求中的一个属性

### 为什么热加载技术没有广泛应用：

类变了，里面的变量保存的信息变了，不可用了，差生大量垃圾，高并发场景不适用

### JSP有哪些内置对象？作用分别是什么？

JSP有9个内置对象：  

- `request`：封装客户端的请求，其中包含来自GET或POST请求的参数；  
- `response`：封装服务器对客户端的响应；
- `pageContext`：通过该对象可以获取其他对象；
- `session`：封装用户会话的对象；
- `application`：封装服务器运行环境的对象；
- `out`：输出服务器响应的输出流对象；
- `config`：Web应用的配置对象；
- `page`：JSP页面本身（相当于Java程序中的this）； 
- `exception`：封装页面抛出异常的对象。

> **补充：**如果用`Servlet`来生成网页中的动态内容无疑是非常繁琐的工作，另一方面，所有的文本和HTML标签都是硬编码，即使做出微小的修改，都需要进行重新编译。`JSP`解决了`Servlet`的这些问题，它是`Servlet`很好的补充，可以专门用作为用户呈现视图（View），而`Servlet`作为控制器（Controller）专门负责处理用户请求并转发或重定向到某个页面。基于Java的Web开发很多都同时使用了`Servlet`和`JSP`。`JSP`页面其实是一个`Servlet`，能够运行`Servlet`的服务器（`Servlet`容器）通常也是`JSP`容器，可以提供`JSP`页面的运行环境，Tomcat就是一个`Servlet`/`JSP`容器。第一次请求一个`JSP`页面时，`Servlet`/`JSP`容器首先将`JSP`页面转换成一个`JSP`页面的实现类，这是一个实现了`JspPage`接口或其子接口`HttpJspPage`的Java类。`JspPage`接口是`Servlet`的子接口，因此每个`JSP`页面都是一个`Servlet`。转换成功后，容器会编译`Servlet`类，之后容器加载和实例化Java字节码，并执行它通常对`Servlet`所做的生命周期操作。对同一个`JSP`页面的后续请求，容器会查看这个`JSP`页面是否被修改过，如果修改过就会重新转换并重新编译并执行。如果没有则执行内存中已经存在的`Servlet`实例。我们可以看一段`JSP`代码对应的Java程序就知道一切了，而且9个内置对象的神秘面纱也会被揭开。

### JSP和Servlet是什么关系？ 

Servlet是一个特殊的Java程序，它运行于服务器的JVM中，能够依靠服务器的支持向浏览器提供显示内容。JSP本质上是Servlet的一种简易形式，JSP会被服务器处理成一个类似于Servlet的Java程序，可以简化页面内容的生成。Servlet和JSP最主要的不同点在于，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML分离开来。而JSP的情况是Java和HTML可以组合成一个扩展名为.jsp的文件。有人说，Servlet就是在Java中写HTML，而JSP就是在HTML中写Java代码，当然这个说法是很片面且不够准确的。JSP侧重于视图，Servlet更侧重于控制逻辑，在MVC架构模式中，JSP适合充当视图（view）而Servlet适合充当控制器（controller）。

### 讲解JSP中的四种作用域。

JSP中的四种作用域包括`page`、`request`、`session`和`application`，具体来说：

- `page`代表与一个页面相关的对象和属性。
- `request`代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。
- `session`代表与某个用户与服务器建立的一次会话相关的对象和属性。跟某个用户相关的数据应该放在用户自己的session中。
- `application`代表与整个Web应用程序相关的对象和属性，它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。

### JSP中的静态包含和动态包含有什么区别？

静态包含是通过JSP的`include`指令包含页面，动态包含是通过JSP标准动作`<jsp:forward>`包含页面。静态包含是编译时包含，如果包含的页面不存在则会产生编译错误，而且两个页面的"`contentType`"属性应保持一致，因为两个页面会合二为一，只产生一个class文件，因此被包含页面发生的变动再包含它的页面更新前不会得到更新。动态包含是运行时包含，可以向被包含的页面传递参数，包含页面和被包含页面是独立的，会编译出两个class文件，如果被包含的页面不存在，不会产生编译错误，也不影响页面其他部分的执行。代码如下所示：

```JSP
<%-- 静态包含 --%>
<%@ include file="..." %>

<%-- 动态包含 --%>
<jsp:include page="...">
    <jsp:param name="..." value="..." />
</jsp:include>
```

## Session

### session 和 cookie 有什么区别？

1. 存储位置不同：session 存储在服务器端；cookie 存储在浏览器端。
2. 安全性不同：cookie 安全性一般，在浏览器存储，可以被伪造和修改。
3. 容量和个数限制：cookie 有容量限制，每个站点下的 cookie 也有个数限制。
4. 存储的多样性：session 可以存储在 Redis 中、数据库中、应用程序中；而 cookie 只能存储在浏览器中。

## Web

### web.xml文件中可以配置哪些内容？ 

web.xml用于配置Web应用的相关信息，如：监听器（listener）、过滤器（filter）、 Servlet、相关参数、会话超时时间、安全验证方式、错误页面等，下面是一些开发中常见的配置：

1. 配置Spring上下文加载监听器加载Spring配置文件并创建IoC容器：

   ```XML
   <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:applicationContext.xml</param-value>
   </context-param>
   
   <listener>
       <listener-class>
           org.springframework.web.context.ContextLoaderListener
       </listener-class>
   </listener>
   ```

2. 配置Spring的OpenSessionInView过滤器来解决延迟加载和Hibernate会话关闭的矛盾：

   ```XML
   <filter>
       <filter-name>openSessionInView</filter-name>
       <filter-class>
           org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
       </filter-class>
   </filter>
   
   <filter-mapping>
       <filter-name>openSessionInView</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

3. 配置会话超时时间为10分钟：

   ```xml
   <session-config>
       <session-timeout>10</session-timeout>
   </session-config>
   ```

4. 配置404和Exception的错误页面：

   ```XML
   <error-page>
       <error-code>404</error-code>
       <location>/error.jsp</location>
   </error-page>
   
   <error-page>
       <exception-type>java.lang.Exception</exception-type>
       <location>/error.jsp</location>
   </error-page>
   ```

5. 配置安全认证方式：

   ```XML
   <security-constraint>
       <web-resource-collection>
           <web-resource-name>ProtectedArea</web-resource-name>
           <url-pattern>/admin/*</url-pattern>
           <http-method>GET</http-method>
           <http-method>POST</http-method>
       </web-resource-collection>
       <auth-constraint>
           <role-name>admin</role-name>
       </auth-constraint>
   </security-constraint>
   
   <login-config>
       <auth-method>BASIC</auth-method>
   </login-config>
   
   <security-role>
       <role-name>admin</role-name>
   </security-role>
   ```

   > **说明：**对`Servlet`（小服务）、`Listener`（监听器）和`Filter`（过滤器）等Web组件的配置，Servlet 3规范提供了基于注解的配置方式，可以分别使用@`WebServlet`、@`WebListener`、@`WebFilter`注解进行配置。 
   >
   > ------
   >
   > 补充：
   >
   > 如果Web提供了有价值的商业信息或者是敏感数据，那么站点的安全性就是必须考虑的问题。安全认证是实现安全性的重要手段，认证就是要解决“Are you who you say you are?”的问题。认证的方式非常多，简单说来可以分为三类： 
   >
   > - What you know?  — 口令 
   > - What you have? — 数字证书（U盾、密保卡） 
   > - Who you are? —  指纹识别、虹膜识别 
   >
   > 在Tomcat中可以通过建立安全套接字层（Secure Socket Layer, SSL）以及通过基本验证或表单验证来实现对安全性的支持。

### get和post请求的区别？

- get请求用来从服务器上获得资源，而post是用来向服务器提交数据；  
- get将表单中数据按照name=value的形式，添加到action 所指向的URL 后面，并且两者使用"?"连接，而各个变量之间使用"&"连接；post是将表单中的数据放在HTTP协议的请求头或消息体中，传递到action所指向URL；  
- get传输的数据要受到URL长度限制（1024字节）；而post可以传输大量的数据，上传文件通常要使用post方式；  
- 使用get时参数会显示在地址栏上，如果这些数据不是敏感数据，那么可以使用get；对于敏感数据还是应用使用post；  
- get使用MIME类型application/x-www-form-urlencoded的URL编码（也叫百分号编码）文本的格式传递参数，保证被传送的参数由遵循规范的文本组成，例如一个空格的编码是"%20"。

### 实现会话跟踪的技术有哪些？

由于`HTTP`协议本身是无状态的，服务器为了区分不同的用户，就需要对用户会话进行跟踪，简单的说就是为用户进行登记，为用户分配唯一的ID，下一次用户在请求中包含此ID，服务器据此判断到底是哪一个用户。 

- URL 重写：在URL中添加用户会话的信息作为请求的参数，或者将唯一的会话ID添加到URL结尾以标识一个会话。 
- 设置表单隐藏域：将和会话跟踪相关的字段添加到隐式表单域中，这些信息不会在浏览器中显示但是提交表单时会提交给服务器。 

这两种方式很难处理跨越多个页面的信息传递，因为如果每次都要修改URL或在页面中添加隐式表单域来存储用户会话相关信息，事情将变得非常麻烦。 

- `cookie`：cookie有两种，一种是基于窗口的，浏览器窗口关闭后，cookie就没有了；另一种是将信息存储在一个临时文件中，并设置存在的时间。当用户通过浏览器和服务器建立一次会话后，会话ID就会随响应信息返回存储在基于窗口的cookie中，那就意味着只要浏览器没有关闭，会话没有超时，下一次请求时这个会话ID又会提交给服务器让服务器识别用户身份。会话中可以为用户保存信息。会话对象是在服务器内存中的，而基于窗口的cookie是在客户端内存中的。如果浏览器禁用了cookie，那么就需要通过下面两种方式进行会话跟踪。当然，在使用cookie时要注意几点：首先不要在cookie中存放敏感信息；其次cookie存储的数据量有限（4k），不能将过多的内容存储cookie中；再者浏览器通常只允许一个站点最多存放20个cookie。当然，和用户会话相关的其他信息（除了会话ID）也可以存在cookie方便进行会话跟踪。 
- `HttpSession`：在所有会话跟踪技术中，`HttpSession`对象是最强大也是功能最多的。当一个用户第一次访问某个网站时会自动创建`HttpSession`，每个用户可以访问他自己的`HttpSession`。可以通过`HttpServletRequest`对象的`getSession`方法获得`HttpSession`，通过`HttpSession`的`setAttribute`方法可以将一个值放在`HttpSession`中，通过调用`HttpSession`对象的`getAttribute`方法，同时传入属性名就可以获取保存在`HttpSession`中的对象。与上面三种方式不同的是，`HttpSession`放在服务器的内存中，因此不要将过大的对象放在里面，即使目前的`Servlet`容器可以在内存将满时将`HttpSession`中的对象移到其他存储设备中，但是这样势必影响性能。添加到`HttpSession`中的值可以是任意Java对象，这个对象最好实现了`Serializable`接口，这样`Servlet`容器在必要的时候可以将其序列化到文件中，否则在序列化时就会出现异常。

> **补充：**HTML5中可以使用Web Storage技术通过JavaScript来保存数据，例如可以使用localStorage和sessionStorage来保存用户会话的信息，也能够实现会话跟踪。

### 解释一下网络应用的模式及其特点。

典型的网络应用模式大致有三类：B/S、C/S、P2P。其中B代表浏览器（Browser）、C代表客户端（Client）、S代表服务器（Server），P2P是对等模式，不区分客户端和服务器。B/S应用模式中可以视为特殊的C/S应用模式，只是将C/S应用模式中的特殊的客户端换成了浏览器，因为几乎所有的系统上都有浏览器，那么只要打开浏览器就可以使用应用，没有安装、配置、升级客户端所带来的各种开销。P2P应用模式中，成千上万台彼此连接的计算机都处于对等的地位，整个网络一般来说不依赖专用的集中服务器。网络中的每一台计算机既能充当网络服务的请求者，又对其它计算机的请求作出响应，提供资源和服务。通常这些资源和服务包括：信息的共享和交换、计算资源（如CPU的共享）、存储共享（如缓存和磁盘空间的使用）等，这种应用模式最大的阻力安全性、版本等问题，目前有很多应用都混合使用了多种应用模型，最常见的网络视频应用，它几乎把三种模式都用上了。

> **补充：**此题要跟"电子商务模式"区分开，因为有很多人被问到这个问题的时候马上想到的是B2B（如阿里巴巴）、B2C（如当当、亚马逊、京东）、C2C（如淘宝、拍拍）、C2B（如威客）、O2O（如美团、饿了么）。