---
title: redis实现分布式文件夹锁
categories: redis
tag:
  - 分布式锁
description: 使用redis实现文件夹级别的分布式锁
date: 2019-01-01 13:27:53
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](redis-file-lock/p12.jpg)

## 缘起

最近做一个项目，类似某度云盘，另外附加定制功能，本人负责云盘相关功能实现，这个项目跟云盘不同的是，以项目为分配权限的单位，同一个项目及子目录所有有权限的用户可以同时操作所有文件，这样就很容易出现并发操作，而且表结构设计的时候，定下来文件和文件夹都有个path字段，存储的是所在父级文件夹路径，这样检索方便，重命名和移动比较麻烦。

如下，例如甲同学正在移动项目下C文件夹，而此时乙同学也在操作项目下D下的d.txt文件，这样就会出现问题，所以需要分布式锁控制，甲在操作C文件夹的时候，C文件夹所有子文件和包含C文件夹的父文件夹都被锁住，如图将会被锁定的文件夹和子文件有：A、C、c.txt、D、E、d.txt，其中a.txt和B未被锁定，这个是移动的情况，如下表格列出其他情况.

![](redis-file-lock/图1.jpg)

| 操作对象  | 操作         | 新建 | 上传 | 移动文件夹 | 移动文件 | 复制文件夹 | 复制文件 | 重命名文件夹 | 重命名文件 | 删除文件夹 | 删除文件 | 回收站清除 |
| :-------- | ------------ | :--- | ---- | ---------- | -------- | ---------- | -------- | ------------ | ---------- | ---------- | -------- | ---------- |
| /A        | 新建         | √    | √    | ×          | √        | ×          | √        | ×            | √          | ×          | √        | /          |
| /A        | 上传         | √    | √    | ×          | √        | ×          | √        | ×            | √          | ×          | √        | /          |
| /A->/B    | 移动文件夹   | A×B× | A×B× | A×B×       | A×B×     | A√B√       | A√B√     | A×B×         | A×B√       | A×B×       | A×B√     | /          |
| a.txt->/B | 移动文件     | B√   | B√   | B×         | B√       | B×         | a√B×     | a×B×         | a×B√       | B×         | a×B√     | /          |
| /A->/B    | 复制文件夹   | B√   | B√   | B×         | B√       | B√         | B√       | B×           | B√         | B×         | B√       | /          |
| a.txt->/B | 复制文件     | B√   | B√   | B×         | B√       | B√         | B√       | B×           | B√         | B×         | B√       | /          |
| /A        | 重命名文件夹 | A×   | A×   | A×         | A×       | A√         | A√       | A×           | A×         | A×         | A×       | /          |
| a.txt     | 重命名文件   | /    | /    | ×          | ×        | ×          | ×        | √            | ×          | ×          | ×        | /          |
| /A        | 删除文件夹   | ×    | ×    | ×          | ×        | ×          | ×        | ×            | ×          | ×          | ×        | /          |
| a.txt     | 删除文件     | ×    | ×    | ×          | ×        | ×          | ×        | ×            | ×          | ×          | ×        | /          |
| /A，a.txt | 回收站清除   | /    | /    | /          | /        | /          | /        | /            | /          | /          | /        | A×a×       |

> 符号解释：√：可以操作，×：不可以操作，/：互相不影响
>
> 整体解释：例如第一行，意思是：对于A这个文件夹，当第一个人进行新建操作的时候，其他人同时进行新建、上传、移动文件、复制文件、重命名文件、删除文件是允许的，移动文件夹、复制文件夹、重命名文件夹、删除文件夹是不允许的，回收站清除和新建操作是互不影响的。

## 思考和调研

分布式锁常见的三种实现方式：数据库、zookeeper/etcd（临时有序节点）、redis（setnx/lua脚本），各有千秋。

### 数据库实现分布式锁

#### 实现

原理简单易实现，创建一张lock表，存储锁定的资源、上锁对象、获取锁的资源、获取锁时间等，获取锁时查询该资源是否存在记录，存在且未过失效时间则获取锁失败，不存在则插入一条数据并且获取锁成功；释放锁则更简单，删除锁数据即可。

#### 缺点

- 释放锁删除数据时，会出现死锁情况

#### 优点

- 实现简单

### zookeeper/etcd实现分布式锁

详见[zookeeper总结]()

### redis实现分布式锁 

详见[Redis总结](/2018/09/12/redis-summary/#分布式锁的实现)

## 分布式文件夹锁实现过程

基于开文处所列情况，要覆盖所有复杂情况很难，但是实现基本的文件夹锁是必须的，故选择了redis+lua脚本，具体代码如下

### java获取锁工具类

```java
/**
 * redis工具类
 */
public class RedisLockUtils {

    static final Long SUCCESS = 1L;
    static final String LOCKED_HASH = "cs:lockedHashKey";
    static final String GET_LOCK_LUA_RESOURCE = "/lua/getFileLock.lua";
    static final String RELEASE_LOCK_LUA_RESOURCE = "/lua/releaseFileLock.lua";
    static final Logger LOG = LoggerFactory.getLogger(RedisLockUtils.class);
    
    /**
     * 获取文件夹锁
     * @param redisTemplate
     * @param lockProjectId
     * @param lockKey
     * @param requestValue
     * @param expireTime 单位：秒
     * @return
     */
    public static boolean getFileLock(RedisTemplate redisTemplate, Long lockProjectId, String lockKey, String requestValue, Integer expireTime) {

        LOG.info("start run lua script，{{}} start request lock",lockKey);
        long start = System.currentTimeMillis();
        DefaultRedisScript<String> luaScript =new DefaultRedisScript<>();
        luaScript.setLocation(new ClassPathResource(GET_LOCK_LUA_RESOURCE));
        luaScript.setResultType(String.class);

        Object result = redisTemplate.execute(
                luaScript,
                Arrays.asList(lockKey, LOCKED_HASH + lockProjectId),
                requestValue,
                String.valueOf(expireTime),
                String.valueOf(System.currentTimeMillis())
        );
        boolean getLockStatus = SUCCESS.equals(result);

        LOG.info("{{}} cost time {} ms，request lock result：{}",lockKey,(System.currentTimeMillis()-start), getLockStatus);
        return getLockStatus;
    }

    /**
     * 释放文件夹锁
     * @param redisTemplate
     * @param lockProjectId
     * @param lockKey
     * @param requestValue
     * @return
     */
    public static boolean releaseFileLock(RedisTemplate redisTemplate, Long lockProjectId, String lockKey, String requestValue) {

        DefaultRedisScript<String> luaScript =new DefaultRedisScript<>();
        luaScript.setLocation(new ClassPathResource(RELEASE_LOCK_LUA_RESOURCE));
        luaScript.setResultType(String.class);

        Object result = redisTemplate.execute(
                luaScript,
                Arrays.asList(lockKey, LOCKED_HASH + lockProjectId),
                requestValue
        );
        boolean releaseLockStatus = SUCCESS.equals(result);

        LOG.info("{{}}release lock result：{}", lockKey, releaseLockStatus);
        return releaseLockStatus;
    }

}
```

### lua脚本

#### 获取文件夹锁

##### 入参说明

`requestKey`为请求锁的路径，`requestValue`为请求锁的value，应为请求锁时生成的`UUID`，确保解锁人只能为上锁人，`lockedKeys`为存放所有锁的`哈希表`的key，这里用常量加项目id的方式，确保一个项目的所有锁存在一个`哈希表`里面，`expireTime`为锁的过期时间，`nowTime`为当前时间，由于lua脚本里面获取当前时间消耗性能且获取的是redis服务器上的当前时间，可能不准确。

##### 思路说明

首先，通过`GET key`判断是否有人正在操作这个文件夹，若有人在操作则直接返回0（获取锁失败），否则获取存放该项目锁的哈希表里面的所有key，遍历所有key，通过lua脚本的`string.find`函数对比该key和请求的key是否存在包含或被包含关系，若存在包含关系且未失效，则返回0（获取锁失败），否则则可获取锁，设置key和过期时间及存入哈希表（哈希表内存放请求锁的key和请求时间），最后返回1（获取锁成功）。

例如请求上图中项目下的C文件夹的锁，请求路径为：`项目/A/C`，当另一个人想操作D文件夹，请求路径为：`项目/A/C/D`，此时查询到存储这个项目所有锁定key的`哈希表`，里面包含`项目/A/C`这个key，这两个key通过lua函数`string.find`发现`项目/A/C/D`包含`项目/A/C`，且未到过期时间，则获取锁失败，否则获取锁成功。

```lua
local requestKey=KEYS[1]
local lockedKeys=KEYS[2]
local requestValue=ARGV[1]
local expireTime=ARGV[2]
local nowTime=ARGV[3]
if redis.call('get',requestKey)
then
    return 0
end
local lockedHash = redis.call('hkeys',lockedKeys)
for i=1, #lockedHash do
    if string.find(requestKey,lockedHash[i]) or string.find(lockedHash[i],requestKey)
    then
        local lockTime = redis.call('hget',lockedKeys,lockedHash[i])
        if (nowTime-lockTime) >= expireTime * 1000
        then
            redis.call('hdel',lockedKeys,lockedHash[i])
        else
            return 0
        end
    end
end
redis.call('set',requestKey,requestValue)
redis.call('expire',requestKey,expireTime)
redis.call('hset',lockedKeys,requestKey,nowTime)
return 1
```

#### 释放文件夹锁

##### 入参说明

`requestKey`为请求锁的路径，`requestValue`为请求锁的value，应为请求锁时生成的`UUID`，确保解锁人只能为上锁人，`lockedKeys`为存放所有锁的`哈希表`的key，这里用常量加项目id的方式，确保一个项目的所有锁存在一个`哈希表`里面。

```lua
local requestKey=KEYS[1]
local lockedKeys=KEYS[2]
local requestValue=ARGV[1]
if redis.call('get', requestKey) == requestValue
then
    redis.call('hdel', lockedKeys,requestKey)
    return redis.call('del',requestKey)
else
    return 0
end
```

### 优点

1. 灵活，锁定的范围可以随`requestKey`变化而变化
2. 性能不错，经测试除了第一次lua脚本未缓存耗时较长，第二次之后则在10ms左右可得到请求结果

### 缺点

1. 可靠性依赖redis
2. 不是可重入锁
3. 维护成本较高，需熟知redis的5种数据结构及lua脚本

## 总结

通过单元自测和测试环境测试基本可以确保多数情况下的多用户并发操作文件只有一人能进行有效操作，保证了数据的安全性。经过这次实践，对分布式锁有了更深入的了解。