---
title: 面试之---框架相关
categories: interview
tags: 
  - 总结
  - 框架
date: 2018-09-16 21:05:51
description: Hibernate、Mybatis、Zookeeper、Netty等java相关框架面试知识
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](interview-framework/interview.png)

> 持续维护java面试题系列博文，发现好的面试题会更新进来。总结的并不只是如何回答面试官，而是这道题涉及到的知识点，明白了相关知识点和面试官要问的点，回答起来就不是问题了。

## 综合

### 什么是ORM？

对象关系映射（Object-Relational Mapping，简称ORM）是一种为了解决程序的面向对象模型与数据库的关系模型互不匹配问题的技术；简单的说，ORM是通过使用描述对象和数据库之间映射的元数据（在Java中可以用XML或者是注解），将程序中的对象自动持久化到关系数据库中或者将关系数据库表中的行转换成Java对象，其本质上就是将数据从一种形式转换到另外一种形式。

### 持久层设计要考虑的问题有哪些？你用过的持久层框架有哪些？

所谓"持久"就是将数据保存到可掉电式存储设备中以便今后使用，简单的说，就是将内存中的数据保存到关系型数据库、文件系统、消息队列等提供持久化支持的设备中。持久层就是系统中专注于实现数据持久化的相对独立的层面。

持久层设计的目标包括： 

- 数据存储逻辑的分离，提供抽象化的数据访问接口。 
- 数据访问底层实现的分离，可以在不修改代码的情况下切换底层实现。 
- 资源管理和调度的分离，在数据访问层实现统一的资源调度（如缓存机制）。 
- 数据抽象，提供更面向对象的数据操作。

持久层框架有： [Hibernate](http://hibernate.org) 、[MyBatis](http://blog.mybatis.org) 、[Spring Data](http://projects.spring.io/spring-data/) 、[TopLink](http://www.oracle.com/technetwork/cn/middleware/toplink/overview/index.html) 、[Guzz](https://code.google.com/p/guzz/) 、[jOOQ](http://www.jooq.org) 、[ActiveJDBC](https://code.google.com/p/activejdbc/)

## Hibernate

### Hibernate中`SessionFactory`是线程安全的吗？`Session`是线程安全的吗（两个线程能够共享同一个Session吗）？

`SessionFactory`对应Hibernate的一个数据存储的概念，它是线程安全的，可以被多个线程并发访问。`SessionFactory`一般只会在启动的时候构建。对于应用程序，最好将`SessionFactory`通过单例模式进行封装以便于访问。`Session`是一个轻量级非线程安全的对象（线程间不能共享session），它表示与数据库进行交互的一个工作单元。`Session`是由`SessionFactory`创建的，在任务完成之后它会被关闭。`Session`是持久层服务对外提供的主要接口。`Session`会延迟获取数据库连接（也就是在需要的时候才会获取）。为了避免创建太多的session，可以使用`ThreadLocal`将session和当前线程绑定在一起，这样可以让同一个线程获得的总是同一个session。Hibernate 3中`SessionFactory`的`getCurrentSession()`方法就可以做到。

### Hibernate中Session的load和get方法的区别是什么？

主要有以下三项区别： 

- 如果没有找到符合条件的记录，get方法返回null，load方法抛出异常。 
- get方法直接返回实体类对象，load方法返回实体类对象的代理。 
- 在Hibernate 3之前，get方法只在一级缓存中进行数据查找，如果没有找到对应的数据则越过二级缓存，直接发出SQL语句完成数据读取；load方法则可以从二级缓存中获取数据；从Hibernate 3开始，get方法不再是对二级缓存只写不读，它也是可以访问二级缓存的。

> **说明：**对于`load()`方法Hibernate认为该数据在数据库中一定存在可以放心的使用代理来实现延迟加载，如果没有数据就抛出异常，而通过`get()`方法获取的数据可以不存在。

### Session的save()、update()、merge()、lock()、saveOrUpdate()和persist()方法分别是做什么的？有什么区别？

Hibernate的对象有三种状态：瞬时态（transient）、持久态（persistent）和游离态（detached），瞬时态的实例可以通过调用`save()`、`persist()`或者`saveOrUpdate()`方法变成持久态；游离态的实例可以通过调用 `update()`、`saveOrUpdate()`、`lock()`或者`replicate()`变成持久态。`save()`和`persist()`将会引发SQL的`INSERT`语句，而`update()`或`merge()`会引发`UPDATE`语句。`save()`和`update()`的区别在于一个是将瞬时态对象变成持久态，一个是将游离态对象变为持久态。`merge()`方法可以完成`save()`和`update()`方法的功能，它的意图是将新的状态合并到已有的持久化对象上或创建新的持久化对象。对于`persist()`方法，按照官方文档的说明：

- `persist()`方法把一个瞬时态的实例持久化，但是并不保证标识符被立刻填入到持久化实例中，标识符的填入可能被推迟到flush的时间；
- `persist()`方法保证当它在一个事务外部被调用的时候并不触发一个`INSERT`语句，当需要封装一个长会话流程的时候，`persist()`方法是很有必要的；
- `save()`方法不保证第二条，它要返回标识符，所以它会立即执行INSERT语句，不管是在事务内部还是外部。

至于`lock()`方法和`update()`方法的区别，`update()`方法是把一个已经更改过的脱管状态的对象变成持久状态；`lock()`方法是把一个没有更改过的脱管状态的对象变成持久状态。

### 阐述Session加载实体对象的过程

`Session`加载实体对象的步骤是： 

- `Session`在调用数据库查询功能之前，首先会在一级缓存中通过实体类型和主键进行查找，如果一级缓存查找命中且数据状态合法，则直接返回； 
- 如果一级缓存没有命中，接下来`Session`会在当前NonExists记录（相当于一个查询黑名单，如果出现重复的无效查询可以迅速做出判断，从而提升性能）中进行查找，如果NonExists中存在同样的查询条件，则返回null； 
- 如果一级缓存查询失败则查询二级缓存，如果二级缓存命中则直接返回；
- 如果之前的查询都未命中，则发出SQL语句，如果查询未发现对应记录则将此次查询添加到Session的NonExists中加以记录，并返回null； 
- 根据映射配置和SQL语句得到`ResultSet`，并创建对应的实体对象；
- 将对象纳入`Session`（一级缓存）的管理；
- 如果有对应的拦截器，则执行拦截器的onLoad方法；
- 如果开启并设置了要使用二级缓存，则将数据对象纳入二级缓存；
- 返回数据对象。

### 锁机制有什么用？简述Hibernate的悲观锁和乐观锁机制。

有些业务逻辑在执行过程中要求对数据进行排他性的访问，于是需要通过一些机制保证在此过程中数据被锁住不会被外界修改，这就是所谓的锁机制。 
Hibernate支持悲观锁和乐观锁两种锁机制。

- 悲观锁，顾名思义悲观的认为在数据处理过程中极有可能存在修改数据的并发事务（包括本系统的其他事务或来自外部系统的事务），于是将处理的数据设置为锁定状态。`悲观锁`必须依赖数据库本身的锁机制才能真正保证数据访问的排他性。
- 乐观锁，顾名思义，对并发事务持乐观态度（认为对数据的并发操作不会经常性的发生），通过更加宽松的锁机制来解决由于悲观锁排他性的数据访问对系统性能造成的严重影响。最常见的乐观锁是通过数据版本标识来实现的，读取数据时获得数据的版本号，更新数据时将此版本号加1，然后和数据库表对应记录的当前版本号进行比较，如果提交的数据版本号大于数据库中此记录的当前版本号则更新数据，否则认为是过期数据无法更新。

Hibernate中通过`Session`的`get()`和`load()`方法从数据库中加载对象时可以通过参数指定使用悲观锁；而乐观锁可以通过给实体类加整型的版本字段再通过XML或`@Version`注解进行配置。

> **提示：**使用乐观锁会增加了一个版本字段，很明显这需要额外的空间来存储这个版本字段，浪费了空间，但是乐观锁会让系统具有更好的并发性，这是对时间的节省。因此乐观锁也是典型的空间换时间的策略。

### 如何理解Hibernate的延迟加载机制？在实际应用中，延迟加载与Session关闭的矛盾是如何处理的？

延迟加载就是并不是在读取的时候就把数据加载进来，而是等到使用时再加载。Hibernate使用了虚拟代理机制实现延迟加载，我们使用Session的`load()`方法加载数据或者一对多关联映射在使用延迟加载的情况下从一的一方加载多的一方，得到的都是虚拟代理，简单的说返回给用户的并不是实体本身，而是实体对象的代理。代理对象在用户调用getter方法时才会去数据库加载数据。但加载数据就需要数据库连接。而当我们把会话关闭时，数据库连接就同时关闭了。

延迟加载与session关闭的矛盾一般可以这样处理： 

- 关闭延迟加载特性。这种方式操作起来比较简单，因为Hibernate的延迟加载特性是可以通过映射文件或者注解进行配置的，但这种解决方案存在明显的缺陷。首先，出现"no session or session was closed"通常说明系统中已经存在主外键关联，如果去掉延迟加载的话，每次查询的开销都会变得很大。 
- 在session关闭之前先获取需要查询的数据，可以使用工具方法`Hibernate.isInitialized()`判断对象是否被加载，如果没有被加载则可以使用Hibernate.initialize()方法加载对象。 
- 使用拦截器或过滤器延长Session的生命周期直到视图获得数据。Spring整合Hibernate提供的`OpenSessionInViewFilter`和`OpenSessionInViewInterceptor`就是这种做法。

### 简述Hibernate常见优化策略。

这个问题应当挑自己使用过的优化策略回答，常用的有：  

- 制定合理的缓存策略（二级缓存、查询缓存）。  
- 采用合理的Session管理机制。 
- 尽量使用延迟加载特性。
- 设定合理的批处理参数。
- 如果可以，选用UUID作为主键生成器。
- 如果可以，选用基于版本号的乐观锁替代悲观锁。
- 在开发过程中, 开启hibernate.show_sql选项查看生成的SQL，从而了解底层的状况；开发完成后关闭此选项。
- 考虑数据库本身的优化，合理的索引、恰当的数据分区策略等都会对持久层的性能带来可观的提升，但这些需要专业的DBA（数据库管理员）提供支持。

### 谈一谈Hibernate的一级缓存、二级缓存和查询缓存。

Hibernate的`Session`提供了一级缓存的功能，默认总是有效的，当应用程序保存持久化实体、修改持久化实体时，`Session`并不会立即把这种改变提交到数据库，而是缓存在当前的`Session`中，除非显示调用了`Session`的`flush()`方法或通过`close()`方法关闭`Session`。通过一级缓存，可以减少程序与数据库的交互，从而提高数据库访问性能。  `SessionFactory`级别的二级缓存是全局性的，所有的`Session`可以共享这个二级缓存。不过二级缓存默认是关闭的，需要显示开启并指定需要使用哪种二级缓存实现类（可以使用第三方提供的实现）。一旦开启了二级缓存并设置了需要使用二级缓存的实体类，`SessionFactory`就会缓存访问过的该实体类的每个对象，除非缓存的数据超出了指定的缓存空间。  一级缓存和二级缓存都是对整个实体进行缓存，不会缓存普通属性，如果希望对普通属性进行缓存，可以使用查询缓存。查询缓存是将`HQL`或`SQL`语句以及它们的查询结果作为键值对进行缓存，对于同样的查询可以直接从缓存中获取数据。查询缓存默认也是关闭的，需要显示开启。

### Hibernate中DetachedCriteria类是做什么的？

`DetachedCriteria`和`Criteria`的用法基本上是一致的，但`Criteria`是由`Session`的`createCriteria()`方法创建的，也就意味着离开创建它的`Session`，`Criteria`就无法使用了。`DetachedCriteria`不需要`Session`就可以创建（使用`DetachedCriteria.forClass()`方法创建），所以通常也称其为离线的`Criteria`，在需要进行查询操作的时候再和`Session`绑定（调用其`getExecutableCriteria(Session)`方法），这也就意味着一个`DetachedCriteria`可以在需要的时候和不同的`Session`进行绑定。

### @OneToMany注解的mappedBy属性有什么作用？

`@OneToMany`用来配置一对多关联映射，但通常情况下，一对多关联映射都由多的一方来维护关联关系，例如学生和班级，应该在学生类中添加班级属性来维持学生和班级的关联关系（在数据库中是由学生表中的外键班级编号来维护学生表和班级表的多对一关系），如果要使用双向关联，在班级类中添加一个容器属性来存放学生，并使用`@OneToMany`注解进行映射，此时mappedBy属性就非常重要。如果使用XML进行配置，可以用<set>标签的`inverse="true"`设置来达到同样的效果。

## Mybatis

### MyBatis 编程步骤

1. 创建 SqlSessionFactory 对象。
2. 通过 SqlSessionFactory 获取 SqlSession 对象。
3. 通过 SqlSession 获得 Mapper 代理对象。
4. 通过 Mapper 代理对象，执行数据库操作。
5. 执行成功，则使用 SqlSession 提交事务。
6. 执行失败，则使用 SqlSession 回滚事务。
7. 最终，关闭会话。

### MyBatis中使用#和$书写占位符有什么区别？

`#`将传入的数据都当成一个字符串，会对传入的数据自动加上引号；`$`将传入的数据直接显示生成在SQL中。注意：使用`$`占位符可能会导致SQL注射攻击，能用`#`的地方就不要使用`$`，写`order by`子句的时候应该用`$`而不是`#`。

### 解释一下MyBatis中命名空间（namespace）的作用。

在大型项目中，可能存在大量的SQL语句，这时候为每个SQL语句起一个唯一的标识（ID）就变得并不容易了。为了解决这个问题，在MyBatis中，可以为每个映射文件起一个唯一的命名空间，这样定义在这个映射文件中的每个SQL语句就成了定义在这个命名空间中的一个ID。只要我们能够保证每个命名空间中这个ID是唯一的，即使在不同映射文件中的语句ID相同，也不会再产生冲突了。

### MyBatis中的动态SQL是什么意思？

对于一些复杂的查询，我们可能会指定多个查询条件，但是这些条件可能存在也可能不存在，例如在58同城上面找房子，我们可能会指定面积、楼层和所在位置来查找房源，也可能会指定面积、价格、户型和所在位置来查找房源，此时就需要根据用户指定的条件动态生成SQL语句。如果不使用持久层框架我们可能需要自己拼装SQL语句，还好MyBatis提供了动态SQL的功能来解决这个问题。MyBatis中用于实现动态SQL的元素主要有： 
- `if` 
- `choose` / `when` / `otherwise` 
- `trim` 
- `where` 
- `set` 
- `foreach`

下面是映射文件的片段。

```XML
    <select id="foo" parameterType="Blog" resultType="Blog">
        select * from t_blog where 1 = 1
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="content != null">
            and content = #{content}
        </if>
        <if test="owner != null">
            and owner = #{owner}
        </if>
   </select>
```

当然也可以像下面这些书写。

```XML
    <select id="foo" parameterType="Blog" resultType="Blog">
        select * from t_blog where 1 = 1 
        <choose>
            <when test="title != null">
                and title = #{title}
            </when>
            <when test="content != null">
                and content = #{content}
            </when>
            <otherwise>
                and owner = "owner1"
            </otherwise>
        </choose>
    </select>
```

再看看下面这个例子。

```XML
    <select id="bar" resultType="Blog">
        select * from t_blog where id in
        <foreach collection="array" index="index" 
            item="item" open="(" separator="," close=")">
            #{item}
        </foreach>
    </select>
```

### 说一下 mybatis 的一级缓存和二级缓存？

一级缓存: 基于 `PerpetualCache` 的 `HashMap` 本地缓存，其存储作用域为 `Session`，当 `Session` flush 或 close 之后，该 `Session` 中的所有 `Cache` 就将清空，默认打开一级缓存。

二级缓存与一级缓存其机制相同，默认也是采用 `PerpetualCache`，`HashMap` 存储，不同在于其存储作用域为 `Mapper(Namespace)`，并且可自定义存储源，如 `Ehcache`。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现`Serializable`序列化接口(可用来保存对象的状态),可在它的映射文件中配置`<cache/>` ；

对于缓存数据更新机制，当某一个作用域(一级缓存 `Session`/二级缓存`Namespaces`)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

### mybatis和Hibernate的区别有哪些？

（1）Mybatis和Hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。

（2）Mybatis直接编写原生态sql，可以严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是灵活的前提是Mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套sql映射文件，工作量大。 

（3）Hibernate对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件，如果用Hibernate开发可以节省很多代码，提高效率。 

### Mybatis 有哪些执行器（Executor）？

Mybatis有三种基本的执行器（Executor）：

`SimpleExecutor`：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。
`ReuseExecutor`：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map内，供下一次使用。简言之，就是重复使用Statement对象。
`BatchExecutor`：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

## Dubbo

todo



>参考链接：
>
>https://blog.csdn.net/jackfrued/article/details/44921941

