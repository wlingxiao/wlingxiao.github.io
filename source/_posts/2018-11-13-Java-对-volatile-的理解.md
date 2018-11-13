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
对内存屏障的理解：
> Store Barrier  
> Store屏障，是x86的”sfence“指令，强制所有在store屏障指令之前的store指令，都在该store屏障指令执行之前被执行，并把store缓冲区的数据都刷到CPU缓存。这会使得程序状态对其它CPU可见，这样其它CPU可以根据需要介入。一个实际的好例子是Disruptor中的BatchEventProcessor。当序列Sequence被一个消费者更新时，其它消费者(Consumers)和生产者（Producers）知道该消费者的进度，因此可以采取合适的动作。所以屏障之前发生的内存更新都可见了。

> Load Barrier  
> Load屏障，是x86上的”ifence“指令，强制所有在load屏障指令之后的load指令，都在该load屏障指令执行之后被执行，并且一直等到load缓冲区被该CPU读完才能执行之后的load指令。这使得从其它CPU暴露出来的程序状态对该CPU可见，这之后CPU可以进行后续处理。一个好例子是上面的BatchEventProcessor的sequence对象是放在屏障后被生产者或消费者使用。

> Java 内存模型中 volatile 变量在写操作之后会插入一个 store 屏障，在读操作之前会插入一个 load 屏障。一个类的final字段会在初始化后插入一个 store 屏障，来确保 final 字段在构造函数初始化完成并可被使用时可见。

```java
class Example {
     private volatile boolean i = false;

    private int j = 0;

    /**
     * 线程 A 执行此方法，修改 i 为 true, j = 0
     */
    public void run() {
        j = 1;
        i = true;
    }

    /**
     * 线程 B 执行 此方法，检查 i 的值，在 i 为 true 时打印 j 的值。
     * 由于被 volatile 修饰的变量在写操作之后会插入一个 store 屏障，在读操作之前会插入一个 load 屏障
     * 屏障之前发生的内存更新都可见，所以线程 B 检查到 i 为 true 时 j 的值为 1
     */
    public void doSomething() {
        while (true) {
            if (i) {
                System.out.println(j); // 会打印 1
            }
        }
    }
}
```
[内存屏障](http://ifeve.com/memory-barriers-or-fences/)  