---
title: 面试之---其他细小知识点汇总
categories: interview
tags: 
  - 总结
date: 2018-09-16 21:00:45
description: 包含Xml、表达式及其他细小java相关的面试知识点
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](interview-others/interview.png)

> 持续维护java面试题系列博文，发现好的面试题会更新进来。总结的并不只是如何回答面试官，而是这道题涉及到的知识点，明白了相关知识点和面试官要问的点，回答起来就不是问题了。

## XML

###  `XML`文档定义有几种形式？它们之间有何本质区别？解析`XML`文档有哪几种方式？

`XML`文档定义分为`DTD`和`Schema`两种形式，二者都是对`XML`语法的约束，其本质区别在于`Schema`本身也是一个`XML`文件，可以被`XML`解析器解析，而且可以为`XML`承载的数据定义类型，约束能力较之`DTD`更强大。对`XML`的解析主要有`DOM`（文档对象模型，`Document Object Model`）、`SAX`（`Simple API for XML`）和`StAX`（Java 6中引入的新的解析XML的方式，`Streaming API for XML`），其中`DOM`处理大型文件时其性能下降的非常厉害，这个问题是由`DOM`树结构占用的内存较多造成的，而且`DOM`解析方式必须在解析文件之前把整个文档装入内存，适合对`XML`的随机访问（典型的用空间换取时间的策略）；`SAX`是事件驱动型的`XML`解析方式，它顺序读取`XML`文件，不需要一次全部装载整个文件。当遇到像文件开头，文档结束，或者标签开头与标签结束时，它会触发一个事件，用户通过事件回调代码来处理`XML`文件，适合对`XML`的顺序访问；顾名思义，`StAX`把重点放在流上，实际上`StAX`与其他解析方式的本质区别就在于应用程序能够把`XML`作为一个事件流来处理。将`XML`作为一组事件来处理的想法并不新颖（`SAX`就是这样做的），但不同之处在于`StAX`允许应用程序代码把这些事件逐个拉出来，而不用提供在解析器方便时从解析器中接收事件的处理程序。

## Others

### 谈一谈测试驱动开发（TDD）的好处以及你的理解。

TDD是指在编写真正的功能实现代码之前先写测试代码，然后根据需要重构实现代码。在JUnit的作者Kent Beck的大作《测试驱动开发：实战与模式解析》（Test-Driven Development: by Example）一书中有这么一段内容：“消除恐惧和不确定性是编写测试驱动代码的重要原因”。因为编写代码时的恐惧会让你小心试探，让你回避沟通，让你羞于得到反馈，让你变得焦躁不安，而TDD是消除恐惧、让Java开发者更加自信更加乐于沟通的重要手段。TDD会带来的好处可能不会马上呈现，但是你在某个时候一定会发现，这些好处包括： 

- 更清晰的代码 — 只写需要的代码 
- 更好的设计 
- 更出色的灵活性 — 鼓励程序员面向接口编程 
- 更快速的反馈 — 不会到系统上线时才知道bug的存在

> **补充：**敏捷软件开发的概念已经有很多年了，而且也部分的改变了软件开发这个行业，TDD也是敏捷开发所倡导的。

TDD可以在多个层级上应用，包括单元测试（测试一个类中的代码）、集成测试（测试类之间的交互）、系统测试（测试运行的系统）和系统集成测试（测试运行的系统包括使用的第三方组件）。TDD的实施步骤是：红（失败测试）- 绿（通过测试） - 重构。关于实施TDD的详细步骤请参考另一篇文章[《测试驱动开发之初窥门径》](http://blog.csdn.net/jackfrued/article/details/44433249)。 
在使用TDD开发时，经常会遇到需要被测对象需要依赖其他子系统的情况，但是你希望将测试代码跟依赖项隔离，以保证测试代码仅仅针对当前被测对象或方法展开，这时候你需要的是测试替身。测试替身可以分为四类： 

- 虚设替身：只传递但是不会使用到的对象，一般用于填充方法的参数列表 
- 存根替身：总是返回相同的预设响应，其中可能包括一些虚设状态 
- 伪装替身：可以取代真实版本的可用版本（比真实版本还是会差很多） 
- 模拟替身：可以表示一系列期望值的对象，并且可以提供预设响应 

Java世界中实现模拟替身的第三方工具非常多，包括`EasyMock`、`Mockito`、`jMock`等。

### 说一下表达式语言（EL）的隐式对象及其作用。

EL的隐式对象包括：`pageContext`、`initParam`（访问上下文参数）、`param`（访问请求参数）、`paramValues`、`header`（访问请求头）、`headerValues`、`cookie`（访问cookie）、`applicationScope`（访问application作用域）、`sessionScope`（访问session作用域）、`requestScope`（访问request作用域）、`pageScope`（访问page作用域）。

用法如下所示：

```JSP
${pageContext.request.method}
${pageContext["request"]["method"]}
${pageContext.request["method"]}
${pageContext["request"].method}
${initParam.defaultEncoding}
${header["accept-language"]}
${headerValues["accept-language"][0]}
${cookie.jsessionid.value}
${sessionScope.loginUser.username}123456789
```

> **补充：**表达式语言的.和[]运算作用是一致的，唯一的差别在于如果访问的属性名不符合Java标识符命名规则，例如上面的accept-language就不是一个有效的Java标识符，那么这时候就只能用[]运算符而不能使用.运算符获取它的值

### 如何设置请求的编码以及响应内容的类型？

通过请求对象（`ServletRequest`）的`setCharacterEncoding(String)`方法可以设置请求的编码，其实要彻底解决乱码问题就应该让页面、服务器、请求和响应、Java程序都使用统一的编码，最好的选择当然是`UTF-8`；通过响应对象（`ServletResponse`）的`setContentType(String)`方法可以设置响应内容的类型，当然也可以通过`HttpServletResponsed`对象的`setHeader(String, String)`方法来设置。

> **说明：**现在如果还有公司在面试的时候问JSP的声明标记、表达式标记、小脚本标记这些内容的话，这样的公司也不用去了，其实JSP内置对象、JSP指令这些东西基本上都可以忘却了。

### 遇到过什么比较难的问题，怎么解决的？



### 在哪个技术点研究的比较深入？







> 参考链接：
>
> https://blog.csdn.net/jackfrued/article/details/44921941