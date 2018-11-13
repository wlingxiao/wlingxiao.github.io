---
title: Java-对 volatile 的理解
date: 2018-11-13 16:32:52
tags: [Java]
categories: Java
---
当一个变量被定义为 `volatile` 之后，它将具备两种特性：
1. 保证此变量对所有线程的可见性，但是不能保证原子性
2. 禁止指令重排序优化
<!-- more -->

当一个变量被定义为 `volatile` 之后，它将具备两种特性：
1. 保证此变量对所有线程的可见性，但是不能保证原子性，如果满足以下规则则可以考虑使用 `volatile` 变量
+ 运算结果不依赖变量的当前值或能够保证只有单一的线程修改变量的值

```java
/**
 * volatile 的典型用法：
 * 检查某个状态标记来判断是否停止任务
 */
class Example {
    private volatile boolean exit = false;

    public void run() {
        while (!exit) {
            // doSomething()
        }
    }

    /**
     * 在另一个线程调用该方法来结束当前线程的循环
     */
    public void stop() {
        exit = true;
    }
}
```

```java
/**
 * volatile 的典型错误用法：
 * i++
 */
class Example {
    private volatile boolean i = 0;

    private AtomicInteger j = new AtomicInteger(0);

    /**
     * 多个线程同时执行该方法时，由于 volatile 不能保证可见性，每一个线程
     * 对 i 的修改不能及时的被其他线程观察到会导致 i 的累加值出现错误
     * 可以使用 java.util.concurrent.atomic.AtomicInteger 替代 volatile 变量实现原子递增操作
     */
    public void increase() {
        i++; // 非线程安全

        j.incrementAndGet(); // 线程安全
    }
}
```

+ 变量不需要与其他的状态变量共同参与不变约束  

```java
/**
 * 例子来自：漫画：什么是 volatile 关键字？
 * https://blog.csdn.net/bjweimengshu/article/details/78769623
 *
 * 我们的需求是由线程 B 来对 start 和 end 加 3 且线程 A 不能退出循环操作
 */
class Example {

    private volatile static int start = 3;

    private volatile static int end = 6;

    // 线程A执行如下代码：
    while (start < end){
        //do something
    }

    /**
     * 线程 B 执行如下代码：
     * 线程 B 执行 start += 3 时，start 可能会先被修改为 6 导致线程 A 的判断 while (start < end) 为 false 退出循环
     * 将 start 和 end 替换为 AtomicInteger 也不能解决问题，因为 start += 3;end += 3; 这两个操作合在一起不是原子操作，
     * 线程 A 有可能会看见 start 被修改但 end 没被修改的中间状态，可以使用加锁的方式将这个两个操作进行同步
     */
    start += 3;
    end += 3;
}
```

[并发关键字volatile（重排序和内存屏障）](https://www.jianshu.com/p/ef8de88b1343)  
[漫画：什么是 volatile 关键字？](https://blog.csdn.net/bjweimengshu/article/details/78769623)  
[Java 并发编程实战 3.1.4](https://book.douban.com/subject/10484692/)  
[深入理解Java虚拟机（第2版） 12.3.3](https://book.douban.com/subject/24722612/)  

2. 禁止指令重排序优化
