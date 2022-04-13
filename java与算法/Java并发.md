# Java高并发核心编程

[TOC]

## #1 多线程原理与实战

#### 1.2 无处不在的进程与线程

-   进程
    -   程序是存放在硬盘中的可执行文件，而一个进程是一个程序的一次启动和执行
    -   进程由程序段、数据段和进程控制块三部分组成
        -   程序段：也称代码段
        -   数据段：进程的操作数据在内存中的位置
        -   进程控制块PCB：包含进程的描述信息和控制信息，是进程存在的唯一标志
    -   现代操作系统中，进程是并发执行的
    -   Java程序的进程
        -   Java编写的程序都运行在JVM中，每启动一个Java应用程序时就会启动一个JVM进程
        -   JVM找到程序入口main()，然后执行main()，这样就产生了一个主线程
-   线程
    -   线程的组成主要有三部分
        -   线程描述信息：包括Thread id、线程名称等
        -   程序计数器PC：记录线程下一跳指令的代码段内存地址
        -   栈内存：每个线程独有，不受垃圾回收器管理，分配单位为栈帧
    -   栈帧（方法帧）
        -   Java中执行程序流程的重要单位是方法，而栈内存的分单位为栈帧
        -   方法每一次执行都需要为其分配一个栈帧，栈帧主要保存该方法中的局部变量、方法的返回地址以及其他方法的相关信息
        -   当跳出方法后，该栈帧会被栈内存弹出，此时栈帧的局部变量内存空间会被回收
    -   线程之间共享进程的方法区内存、堆内存、系统资源
-   进程与线程
    -   一个进程至少有一个线程
    -   进程是操作系统分配资源的最小单位，而线程是CPU调度的最小单位
    -   线程的上下文切换速度比进程上下文切换速度快得多，所以有时称线程为轻量级进程

#### 1.3 创建线程的4种方法

##### 1.3.3 线程创建方法一、二：继承Thread类，实现Runnable接口

-   在Java中使用一个Thread实例来描述一个线程
-   线程的`start()`用于线程的启动，`run()`作为用户代码逻辑的执行入口，前者会在适当的时间调用后者
-   注解FunctionalInterface：表示可以使用lambda表达式进行简写，但只是一个声明而已，默认情况下符合要求的接口与方法也能使用lambda表达式
-   Runnable接口方式与继承方式相比的缺点
    -   不能直接访问当前线程的属性，或者是控制当前线程，需要借助`Thread.currentThread()`来获取当前线程实例

##### 1.3.7 线程创建方法三：使用Callable和FutureTask创建线程

-   Callable接口：Java1.5+

    -   相比前两者，能够获取异步执行结果
    -   `call()`有返回值，且返回类型是一个泛型，也有受检异常的异常声明
    -   Callable接口没办法作为Thread线程实例的target来使用，所以需要用到RunnableFuture接口

-   RunnableFuture接口

    -   既接受Runnable，也接受Callable
    -   继承了Runnable接口和Future接口

-   Future接口

    -   功能
        -   能够取消异步执行中的任务
        -   判断异步任务是否执行完成
        -   获取异步任务完成后的执行结果
    -   部分方法
        -   `get()`：获取异步任务的结果。这个方法的调用是阻塞性的，当callable未执行完成时会一直阻塞直到完成。可以设置时限
        -   `cancel()`：取消异步任务的执行

-   FutureTask类

    -   Future接口的实现类，提供了对异步任务的具体实现
    -   实现了RunnableFuture接口，FutureTask类是真正的在Thread和Callable之间搭桥的类（适配器）
    -   成员`Object outcome`：用于保存成员call()方法的异步执行结果，供`get()`方法获取

-   使用步骤

    -   创建一个Callable接口的实例类，并实现其call()方法
    -   使用Callable实现类构造一个FutureTask实例，并将其作为Thread构造器的target入参，构造新的Thread线程实例

    ```java
    public static void main(String[] args) {
        ReturnableTask task = new ReturnableTask();
        FutureTask<Long> futureTask = new FutureTask<>(task);
        new Thread(futureTask).start();
    }
    
    static class ReturnableTask implements Callable<Long>{
        @Override
        public Long call() throws Exception {
            return null;
        }
    }
    ```

##### 1.3.8 线程创建方法四：通过线程池创建

-   线程池的使用
    -   创建：`ExecutorService pool = Executors.newFixedThreadPool(3);` 
    -   向线程池提交任务
        -   `void execute(Runnable command)` 
        -   `Future<?> submit(Runnable task)` 
        -   `<T> Future<T> submit(Callable<T> task)` 



#### 1.4 线程的核心原理

-   Java将线程调度工作委托给操作系统的调度进程去完成
-   线程的调度与时间片（操作系统）
    -   线程只有得到CPU时间片才能执行指令，并处于执行状态
    -   没有得到CPU时间片的线程则处于就绪状态，等待系统分配下一个CPU时间片
    -   CPU时间片是将CPU的时间进行分段，时间片对我们而言非常短，因此线程之间的切换表现出来的特征仍是并发执行
    -   线程的调度模型
        -   分时调度模型：轮流占用CPU
        -   抢占式调度模型：按照线程优先级分配。优先级高的线程获取的CPU时间片会相对多一些。主流的操作系统都使用该模型，所以使得Java中线程存在priority
    -   java线程优先级：最大为10，最小为1，默认为5
-   线程的生命周期（Java）
    -   NEW：创建成功，但是没有调用start()方法启动
    -   RUNNABLE：调用`start()`后，包括了运行状态（得到CPU时间片）和就绪状态（等待CPU时间片）
    -   TERMINATED：线程的代码逻辑执行完成或者异常终止导致run()结束
    -   TIMED_WAITING：线程进入限时等待状态。进入等待的情况如下
        -   `Thread.sleep()` 
        -   `Object.wait()` 
        -   `Thread.join()` 
        -   `LockSupport.parkNanos()` 
        -   `LockSupport.parkUntil()` 
-   使用Jstack查看线程状态
    -   JVM自带，可以导出JVM运行实例当前时刻的线程快照









## #2 Java内置锁的核心原理

## #3 CAS原理与JUC原子类

## #4 可见性与有序性的原理

## #5 JUC显示锁的原理与实战

## #6 AQS抽象同步器的核心原理

## #7 JUC容器类

## #8 高并发设计模式

## #9 高并发核心模式之异步回调模式

## #10 CompletableFuture异步回调

