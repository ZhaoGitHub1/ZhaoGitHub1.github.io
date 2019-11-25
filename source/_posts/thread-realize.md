---
title: 创建线程的几种方式
categories: 基础
tag: 
  - 多线程
date: 2018-09-07 13:45:44
description: 实现线程的六种方式demo
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](thread-realize/words14.jpg)

## 继承Thread类

```java
public class ExtendsThreadTest extends Thread {
    @Override
    public void run() {
        System.out.println("thread is running！");
    }
    public static void main(String[] args) {
        ExtendsThreadTest et1 = new ExtendsThreadTest();
        et1.start();
    }
}
```

---
## 实现Runnable接口
```java
public class RunnableTest implements Runnable{
    @Override
    public void run() {
        System.out.println("thread is running！");
    }
    public static void main(String[] args) {
        Thread t1 = new Thread(new RunnableTest());
        t1.start();
    }
}
```

---
## 匿名内部类的两种写法
```java
public class App {
    public static void main(String[] args){
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("thread1 is running！");
            }
        }){}.start();

        new Thread(){
            @Override
            public void run(){
                System.out.println("thread2 is running！");
            }
        }.start();
    }
}
```

---
## 基于java.util.concurrent.Callable工具类的实现，带返回值
```java
public class CallableTest {
    public static void main(String[] args) throws Exception {
        Callable<Integer> call = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                System.out.println("thread is running！");
                return 1;
            }
        };
        FutureTask<Integer> task = new FutureTask<>(call);
        Thread t =  new Thread(task);
        t.start();
    }
}
```

---
## 基于java.util.Timer工具类的实现
```java
public class TimerTest {
    public static void main(String[] args) throws Exception {
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("thread is running!");
            }
        }, new Date());
    }
}
```
---
## 基于java.util.concurrent.Executors工具类，基于线程池的实现
```java
public class ThreadPoolTest {
    public static void main(String[] args) {
        // 创建线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        while(true) {
            threadPool.execute(new Runnable() { // 提交多个线程任务，并执行
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + " is running ..");
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```

