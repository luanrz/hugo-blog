---
title: Java单例模式中的一个多线程问题
date: 2022-04-15 14:43:37
categories:
- 编程语言
- Java
tags:
- Java
- 并发
- 设计模式
---

合理使用单例模式可以节约内存资源，但错误的使用可能会导致严重的生产问题，如：多线程下，一个线程可能会覆盖上一个线程的单例属性，导致两次不同的请求得到同样的响应。

下面将结合一个例子来分析这种情况。

<!--more-->


## 一、案例描述

考虑以下代码：

```java
/**
 * 单例类
 */
public final class Singleton {
    /** 单例持有的普通变量 */
    private String data;

    /** 单例 */
    private static Singleton singleton;

    /**
     * 私有构造方法
     */
    private Singleton(){ }
    
    /**
     * 获取单例
     * @return Singleton单例
     */
    public static Singleton getInstance(){
        if (singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }

    /**
     * 核心处理方法
     */
    public String handle() {
        // 处理当前单例的data数据
        try {
            //假装在处理
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "响应：" + data;
    }

    /**
     * 设置data
     * @param data data
     * @return Singleton单例
     */
    public Singleton setData(String data) {
        this.data = data;
        return singleton;
    }
}
```

可以使用下面的链式调用来使用上述单例：

```java
Singleton.getInstance().setData("data").handle();
```

1. 调用`Singleton.getInstance()`获取单例
2. 调用`setData()`方法为`data`属性赋值
3. 调用`handle()`方法处理核心逻辑，其中`handle()`使用`data`属性。

在单线程下，上述单例模式没有任何问题。但在多线程下，上述单例至少存在两个问题：

1. 在多个线程同时调用`set()`方法时，单例持有的普通变量`data`中，后一个值可能会覆盖掉前一个值，导致在`handle()`方法中，使用的都是同一个数据，最终出现请求不同但响应相同的现象。
2. 上述单例为懒汉式单例，只有在首次调用`getInstance()`时才会实例化，若“首次”调用是多个线程同时执行时，可能会重复创建多个实例（这违背了单例模式的初衷）。

## 二、问题复现

我们先来看第一个问题，为了复现这种情况，编写以下单元测试代码：

```java
import org.junit.Assert;
import org.junit.Test;

import java.util.*;

/**
 * 单例多线程测试
 */
public class SingletonConcurrencyTest {

    @Test
    public void test() throws InterruptedException {
        // 普通HashSet线程不安全，多个线程对同一个Set执行put操作时可能会丢失数据
        // 使用Collections.synchronizedSet创建线程安全的HashSet
        Set<String> responseSet = Collections.synchronizedSet(new HashSet<>());
        Set<String> threadNameSet = Collections.synchronizedSet(new HashSet<>());

        // 初始化模板渲染线程组
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            Thread t = new Thread(() -> {
                // 核心逻辑
                String data = UUID.randomUUID().toString();
                String result = Singleton.getInstance()
                        .setData(data)
                        .handle();

                // 记录重复响应
                if (responseSet.contains(result)) {
                    result = result + "重复";
                    System.out.println(result);
                }
                responseSet.add(result);
                threadNameSet.add(Thread.currentThread().getName());
            });
            threads.add(t);
        }

        // 批量启动模板渲染线程
        for (Thread t : threads) {
            t.start();
        }
        for (Thread t : threads) {
            t.join();
        }

        // 结果断言
        System.out.println("线程执行完成次数:" + threadNameSet.size());
        System.out.println("非重复响应次数：" + responseSet.size());
        Assert.assertTrue(responseSet.stream().noneMatch(id -> id.contains("重复")));
    }
}
```

运行上述单例模式之后将会打印日志：

```
2c20a55b-aa85-4809-ab09-b2a5c58c38a5重复
...
2c20a55b-aa85-4809-ab09-b2a5c58c38a5重复
线程执行完成次数:100
非重复响应次数：2

java.lang.AssertionError
```

可以看到，同时发了一百个请求，最终却得到了98个重复的响应。

## 三、修改方案

为了避免线程串用单例中的属性，有以下三种修改方案：
1. 弃用单例模式，每次使用时新建对象
2. 去掉单例模式中的普通属性，通过参数的形式将数据传输给对应方法
3. 将单例模式中的普通属性设置为ThreadLocal变量

方案3修改最简单且效率相对较高，下面将使用此方案。

修改单例类如下：

```java
/**
 * 单例类
 */
public final class Singleton {
    /**
     * 单例持有的普通变量
     */
    private ThreadLocal<String> dataThreadLocal = new ThreadLocal<>();

    /**
     * 单例
     */
    private static Singleton singleton;

    /**
     * 私有构造方法
     */
    private Singleton() {
    }

    /**
     * 获取单例
     *
     * @return Singleton单例
     */
    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }

    /**
     * 核心处理方法
     */
    public String handle() {
        String data = dataThreadLocal.get();
        // 处理当前单例的data数据
        try {
            //假装在处理
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 使用完之后移除ThreadLocal以避免内存泄漏
        dataThreadLocal.remove();
        return "响应：" + data;

    }

    /**
     * 设置data
     *
     * @param data data
     * @return Singleton单例
     */
    public Singleton setData(String data) {
        dataThreadLocal.set(data);
        return singleton;
    }
}
```

正常情况下，使用ThreadLocal后，每个线程将持有各自的`data`值，属性串用的问题将得到解决。再次运行`SingletonConcurrencyTest`单元测试，日志如下所示：

```
[响应：null]重复
[响应：null]重复
[响应：null]重复
[响应：null]重复
[响应：null]重复
[响应：null]重复
线程执行完成次数:100
非重复响应次数：95
```

结果并没有和预期一样！可以看到，在调用`ThreadLocal`的`get()`方法时，返回了null。还记得[案例描述](#一案例描述)中说的两个问题吗，我们的单例是懒汉式的，导致多线程下创建了多个实例，同时，我们的ThreadLocal不是static的，每个实例都持有独立的ThreadLocal变量，最终导致返回了null（此处挖坑，后面写一篇ThreadLocal原理的文章）！

通过以下两种方式可以解决这个问题：

1. 为ThreadLocal加上static标识（官方也是这么建议的，详见ThreadLocal源码注释）
2. 为懒汉式单例`getInstance()`方法加锁，或使用饿汉式单例

```java
//Singleton.java

//方式1
private static ThreadLocal<String> dataThreadLocal = new ThreadLocal<>();
//方式2:饿汉式单例
private static Singleton singleton = new Singleton();
```

再次运行单元测试，日志如下所示：
```
线程执行完成次数:100
非重复响应次数：100
```

成功解决！
