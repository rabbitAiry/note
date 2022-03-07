# Java进阶

>   Java编程的逻辑

[TOC]

## a.泛型与容器

### #8 泛型

-   泛型

    -   泛型类
        -   类名后面多了一个`<T>`
        -   数据类型也可以为泛型T
    -   T表示类型参数，泛型就是类型参数化，处理的数据类型不是固定的，而是可以作为参数传入
    -   类型参数可以有多个，使用逗号分隔
    -   Java7+可以省略后面的类型参数

    ```java
    Pair<String, Integer> pair = new Pair<>("可乐",3)
    ```

    -   基本原理

        -   通过类型擦除实现，类定义中的类型参数T会被替换为Object。Java编译器会插入必要的强制类型转换
        -   在程序运行过程中，不知道泛型的实际类型参数

    -   好处：更好的安全性、更好的可读性

    -   泛型方法

        -   与泛型类不同，调用时不需要特意指定类型参数的实际类型

        ```java
        public static <T> int indexOf(T[] arr, T elm)
        ```

    -   泛型接口

    -   类型参数的限定

        -   上界为某个类

        -   上界为某个接口、其他类型参数
        
        -   上界为其他参数类型

            ```java
            // 上界为某个类
            public class Pair<U extends Number>
            ```

-   泛型的通配符



[TOC]

## b. 并发

### #15 并发基础知识

##### 15.1 线程基本概念

-   线程
    -   单独的执行流，有自己的程序执行计数器，自己的栈
    -   但线程之间可以共享内存，可以访问和操作相同的对象
    -   创建线程两种方式
        -   继承Thread
        -   实现Runnable接口
    -   启动线程
        -   调用线程的`start()`表示启动该线程
        -   调用线程的`run()`不会启动一条单独的执行流，`run()`的代码仍在main线程中执行
-   线程的基本属性与方法
    -   id和name
    -   优先级priority
    -   状态state
        -   NEW：没有调用start的线程的状态
        -   TERMINATED：线程运行结束后的状态
        -   RUNNABLE：在执行且没有阻塞时的状态
        -   BLOCKED、WAITING、TIMED_WAITING：被阻塞的状态
    -   `isAlive()`：返回线程是否还活着
    -   是否deamon线程
        -   deamon线程：辅助线程，如垃圾回收线程
        -   当主线程退出时，daemon线程已没有意义
        -   程序只会在所有线程都结束时才退出，但daemon线程除外
    -   `sleep()`
    -   `yield()`：表示不急着占用CPU，可以让其他线程运行
    -   `join()`：等待该线程结束后再退出
-   共享内存可能存在的问题
    -   竞态条件
    -   内存可见性
-   线程的优点及成本

##### 15.2 理解synchronized

-   synchronized可以用于修饰：
    -   实例方法
    -   静态方法
    -   代码块
-   synchronized特性
    -   可重入性
    -   内存可见性
    -   死锁
-   同步容器及其注意事项
    -   复合操作
    -   伪同步
    -   迭代
    -   并发容器

##### 15.3 线程的基本协作机制

-   协作的场景
-   wait/notify
-   生产者/消费者模式
-   同时开始
-   等待结束
-   异步结果
-   集合点

##### 15.4 线程的中断

-   场景
-   机制
-   线程的反应
    -   RUNNABLE
    -   WAITING/TIME_WAITING
    -   BLOCKED
    -   NEW/TERMINATE
-   正确操作



[TOC]

### #16 并发包的基石

>   Java并发工具包

##### 16.1 原子变量和CAS

-   AtomicInteger
-   ABA问题

##### 16.2 显式锁

-   接口Lock
-   可重入锁ReentrantLock
    -   基本用法
    -   使用tryLock避免死锁
    -   实现原理
        -   LockSupport
        -   AQS
        -   ReentrantLock
-   与synchronized对比

##### 16.3 显式条件

-   用法
-   生产者/消费者模式
-   原理

[TOC]

### #17 并发容器

##### 17.1 写时复制的List和Set

-   写时复制的List和Set
    -   线程安全的，可被多个线程并发访问
    -   其迭代器不支持修改操作，但也不会抛出ConcurrentModificationException
    -   以原子方式支持一些复合操作
    -   迭代时不需要加锁
-   CopyOnWriteArrayList
-   CopyOnWriteArraySet

##### 17.2 ConcurrentHashMap

-   并发版本的HashMap
    -   并发安全
    -   直接支持一些原子复合操作
    -   支持高并发，读操作完全并行，写操作支持一定程度的并行
    -   与同步容器Collections.synchronizedMap相比，迭代不用加锁，不会抛出ConcurrentModificationException
    -   弱一致性

##### 17.3 基于跳表的Map和Set

-   Java并发包中与TreeMap/TreeSet对应的并发版本是ConcurrentSkipListMap和ConcurrentSkipListSet
-   基于跳表的Map和Set
    -   没有使用锁，所有操作都是无阻塞的，所有操作都可以并行，包括写，多线程可以同时写
    -   弱一致性
    -   支持一些原子复合操作

##### 17.4 并发队列

-   Java并发包提供的几种队列
    -   无锁非阻塞并发队列
    -   普通阻塞队列
    -   优先级阻塞队列
    -   延时阻塞队列
    -   其他

[TOC]

### #18 异步任务执行服务

##### 18.1 基本概念和原理

-   任务执行设计的基本接口
    -   Runnable和Callable：表示要执行的异步任务
    -   Executor和ExecutorService：表示执行服务
    -   Future：表示异步任务的结果

##### 18.2 线程池

-   线程池里面有若干线程，它们的目的就是执行提交给线程池的任务，执行完一个任务后不退出，而是继续等待或执行新任务
    -   主要由两个概念组成
        -   任务队列
        -   工作者线程

##### 18.3 定时任务的陷阱

