---
title: 设计模式之---汇总
categories: 
  - 设计模式
tag: 
  - 总结
date: 2018-09-07 13:45:44
description: 设计模式总结及导读
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](pattern-all/words4.jpg)

> 说明：本系列所有实现方式都是基于`java`语言实现，由于设计模式种类众多此处`并未能包含所有设计模式`。
## 定义
**模式**：模式是在特定环境下人们解决某类重复出现问题的一套成功或有效的解决方案。【A pattern is a successful or efficient solution to a recurring  problem within a context】

**设计模式**：设计模式设计模式(Design Pattern)是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结，使用设计模式是为了可重用代码、让代码更容易被他人理解并且保证代码可靠性。

## 七大设计原则
- `单一职责原则`(Single Responsibility Principle, SRP)
  - 一个类只负责一个功能领域中的相应职责，或者可以定义为：就一个类而言，应该只有一个引起它变化的原因。
- `开闭原则`(Open-Closed Principle, OCP)
  - 一个软件实体应当对扩展开放，对修改关闭。即软件实体应尽量在不修改原有代码的情况下进行扩展。
- `里氏代换原则`(Liskov Substitution Principle, LSP)
  - 所有引用基类（父类）的地方必须能透明地使用其子类的对象。
- `依赖倒转原则`(Dependence  Inversion Principle, DIP)
  - 抽象不应该依赖于细节，细节应当依赖于抽象。换言之，要针对接口编程，而不是针对实现编程。
- `接口隔离原则`(Interface Segregation Principle, ISP)
  - 使用多个专门的接口，而不使用单一的总接口，即客户端不应该依赖那些它不需要的接口。
- `合成复用原则`(Composite Reuse Principle, CRP)
  - 尽量使用对象组合，而不是继承来达到复用的目的。
- `迪米特法则`(Law of Demeter, LoD)
  - 一个软件实体应当尽可能少地与其他实体发生相互作用。

## GoF 23种设计模式
### 创建型模式
- `工厂方法模式`(Factory Method)
- `抽象工厂模式`(Abstract Factory)
- `建造者模式`(Builder)
- `原型模式`(Prototype)
- [`单例模式`(Singleton)](https://www.yizhuxiaozhan.site/2018/09/10/pattern-singleton/)
### 结构型模式
- `代理模式`(Proxy)
- `适配器模式`(Adapter)
- `桥接模式`(Bridge)
- `组合模式`(Composite)
- `装饰者模式`(Decorator)
- `门面模式`/`外观模式`(Facade)
- `享元模式`(Flyweight)
### 行为型模式
- `解释器模式`(Interpreter)
- `模板方法模式`(Template Method)
- `责任链模式`/`职责链模式`(Chain of Responsibility)
- `命令模式`(Command)
- `迭代器模式`(Iterator)
- `调解者模式`/`中介者模式`(Mediator)
- `备忘录模式`(Memento)
- `观察者模式`(Observer)
- `状态模式`(State)
- `策略模式`(Strategy)
- `访问者模式`(Visitor)
### GoF 23种设计模式之间的关系图
**![](pattern-all/patterns.jpg)**

## 其他常见设计模式
- `简单工厂模式`（Simple Factory ）
- `委派模式`（Delegate）
- `过滤器模式`/`标准模式`（Filter、Criteria）
- `空对象模式`（Null Object）
### J2EE 模式
- `MVC 模式`（MVC）
- `业务代表模式`（Business Delegate）
- `组合实体模式`（Composite Entity）
- `数据访问对象模式`（Data Access Object）
- `前端控制器模式`（Front Controller）
- `拦截过滤器模式`（Intercepting Filter）
- `服务定位器模式`（Service Locator）
- `传输对象模式`（Transfer Object）
## 参考文章：
[https://blog.csdn.net/lovelion/article/details/17517213/](https://blog.csdn.net/lovelion/article/details/17517213/)