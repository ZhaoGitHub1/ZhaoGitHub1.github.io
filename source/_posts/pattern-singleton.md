---
title: 设计模式之---单例模式(Singleton Pattern)
categories: 设计模式
tag: 
  - 单例模式
date: 2018-09-10 21:45:44
description: 常用网站清单，记录一下方便日常使用
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](pattern-singleton/it7.png)


## 概述

### 基本概念

　　确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种创建型模式。

### 缘起

　　某些类，我们希望在程序运行期间有且只有一个实例，原因可能是该类的创建需要消耗系统过多的资源、花费很多的时间，或者业务上客观就要求了只能有一个实例。

### 特点
1. 单例类只能有一个实例。
2. 单例类必须自己创建自己的唯一实例。
3. 单例类必须给所有其他对象提供这一实例。

### 应用场景

1. 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源消耗太大而只允许创建一个对象。
2. 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。

### 常见应用
1. 配置文件
2. 日历类
3. IOC容器
4. Windows任务管理器
5. IOC容器级别的单例bean


## 常见的几种实现方式
### 饿汉式单例
``` 
public class EagerSingleton {   
    private static final EagerSingleton instance = new EagerSingleton();   
    private EagerSingleton() { }   
    public static EagerSingleton getInstance() {  
        return instance;   
    }     
}
```
> 优点：
> 线程安全、在类加载的同时已经创建好一个静态对象，调用时反应速度快

> 缺点：
> 资源效率不高，类加载时就初始化，浪费内存。可能getInstance()永远不会执行到，但执行该类的其他静态方法或者加载了该类（class.forName)，那么这个实例仍然初始化 

### 懒汉式单例

```
public class LazySingleton {   
    private volatile static LazySingleton instance = null;
     
    private LazySingleton() { }   
     
    public static LazySingleton getInstance() {   
        //第一重判断  
        if (instance == null) {  
            //锁定代码块  
            synchronized (LazySingleton.class) {  
                //第二重判断  
                if (instance == null) {  
                    instance = new LazySingleton(); //创建单例实例  
                }  
            }  
        }  
        return instance;   
    } 
}
```
> 优点：
> 资源利用率高，不执行getInstance()就不被实例，可以执行该类其他静态方法 

> 缺点 ： 
> 第一次加载时反应不快，由于java内存模型一些原因偶尔失败 

### 内部静态类实现
```
public class Singleton {  
    private Singleton() {  
    }  
      
    private static class SingletonHolder {  
        private final static Singleton instance = new Singleton();  
    }  
      
    public static Singleton getInstance() {  
        return SingletonHolder.instance;  
    }    
}  
```
> 优点：
> 资源利用率高，不执行getInstance()不被实例，可以执行该类其他静态方法 

> 缺点 ：
> 第一次加载时反应不够快 

### 枚举实现
```
public enum EnumSingleton {
  INSTANCE;
}
```
> 优点：
> 代码简洁，线程安全，防反射攻击、防止序列化生成新的实例

> 缺点：
> 不支持java1.5以下

----------

## 单例模式总结

### 一.主要优点
1.  单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。
2.  由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象单例模式无疑可以提高系统的性能。
3.  允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例，既节省系统资源，又解决了单例对象共享过多有损性能的问题。

###  二.主要缺点
1. 由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
2. 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起。
3. 现在很多面向对象语言(如Java、C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失。


>参考文章：[https://blog.csdn.net/lovelion/article/details/17517213/](https://blog.csdn.net/lovelion/article/details/17517213/)