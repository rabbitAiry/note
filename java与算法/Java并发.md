# Java高并发核心编程

>Java高并发核心编程、Java编程的逻辑

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

-   使用线程池的原因
    -   Java线程的创建需要JVM和OS配合完成大量工作
        -   为线程对栈分配和初始化大量内存块，其中包含至少1MB的栈内存
        -   需要进行系统调用，以便在OS中创建和注册本地线程

    -   Java高并发应用频繁创建和销毁线程的操作是非常低效的

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
    -   RUNNABLE
        -   调用`start()`后，包括了运行状态（RUNNING：得到CPU时间片）和就绪状态（READY：等待CPU时间片）
        -   更多时候，把两种CPU状态抽象为RUNNABLE即可
    -   TERMINATED：线程的代码逻辑执行完成或者异常终止导致run()结束
    -   TIMED_WAITING：线程进入限时等待状态。该状态不会被分配CPU时间片。进入等待的情况如下
        -   `Thread.sleep(time)` 
        -   `Object.wait(time)` 
        -   `Thread.join(time)` 
        -   `LockSupport.parkNanos()` 
        -   `LockSupport.parkUntil()` 
    -   BLOCK：不会占用CPU资源。进入阻塞状态的情况如下
        -   线程等待获取锁
        -   IO阻塞
    -   WAITING：无限期等待。进入的情况如下
        -   `Object.wait()` 
        -   `Thread.join()` 
        -   `LockSupport.park()` 
-   使用Jstack查看线程状态
    -   JVM自带，可以导出JVM运行实例当前时刻的线程快照



#### 1.5 线程的基本操作

-   sleep操作

    -   会抛出InterruptedException
    -   线程睡醒后不一定立即得到执行，仍需等待CPU时间片

-   interrupt操作

    -   不建议使用Java提供的`stop()`，因为强行中断线程可能会导致当前持有锁不能释放等问题
    -   `interrupt()`只是将线程设为中断状态，设置后
        -   如果线程属于阻塞状态，就会立马退出阻塞，并抛出InterruptedException。可以通过捕获该异常从而实现让线程退出的操作
        -   `Object.wait()`、`Thread.join()`、`Thread.sleep()`都会阻塞线程
        -   如果InterruptedException被线程捕获后，仍继续调用阻塞方法，则不会触发InterruptedException
        -   如果线程处于运行状态中，则线程不受任何影响
    -   可以通过`isInterrupted()`查看自身是否被标记了中断

-   join操作

    -   使用`join()`的一方为被动方，使用的位置所属线程为主动方

    ```java
    // A是主动方，A会等待B
    class ThreadA extends Thread{
    	void run(){
            Thread threadB = new Thread("thread-b");
            threadB.join();
        }
    }
    ```

    -   执行该方法后，主动方进入TIMED_WAITING，直到被对方线程执行结束或超时
    -   仍没有办法直接取得乙方线程的执行结果
    -   超时或被合并后，线程进入RUNNABLE状态

-   yield操作

    -   让目前正在执行的线程放弃当前的执行，让出CPU的执行权限，使得CPU去执行其他的线程
        -   状态仍是RUNNABLE，但是从CPU层面上，执行状态变为了就绪状态
        -   不会进入阻塞状态
        -   CPU也可能仍选中了当前线程

-   daemon操作

    -   Java中的线程有两类：用户线程与守护线程
        -   守护线程（daemon、后台线程）
        -   专门用于在后台提供某种通用服务的线程，如GC线程
        -   当JVM进程中已无用户线程时，JVM强行终止daemon线程
    -   守护线程的基本操作
        -   通过`setDaemon(true)`设置为守护线程，必须在线程启动前设置
        -   守护线程创建的线程默认也是守护线程



#### 1.6 线程池原理与实战

-   JUC的线程池架构

    -   JUC（java.util.concurrent）：java并发包，1.5+
    -   在多线程编程中，任务都是一些抽象且离散的工作单元，而线程是使异步任务执行的基本机制。随着任务管理变得复杂，为了简化而使用线程池来统一管理线程及任务分配
    -   Executor：接口
    -   ExecutorService：接口，Executor的子类，对外提供异步任务的接收服务
    -   AbstractExecutorService：实现了ExecutorService接口，为ExecutorService中的接口提供默认实现
    -   ThreadPoolExecutor：线程池实现类
    -   ScheduledExecutorService：接口，实现了ExecutorService接口，是一个可以完成延时和周期性任务的调度线程池接口
    -   ScheduledThreadPoolExecutor：继承于ThreadPoolExecutor并实现了ScheduledExecutorService
    -   Executors：静态工厂类，提供了四种快捷创建线程池的方法

-   4种线程池

    -   SingleThreadExecutor：只有一个线程的线程池

        -   能够保证所有任务按照指定顺序执行
        -   唯一线程存活时间无限，阻塞队列无界

    -   FixedThreadPool：固定数量的线程池

        -   阻塞队列无界，线程数量不超过固定数量
        -   使用场景：需要任务长期密集执行的场景

    -   CachedThreadPool：可缓存线程池

        -   收到任务一定立即执行，线程数量无限制，依赖于操作系统能够创建的最大线程数量大小
        -   空闲线程会在60秒后被回收
        -   使用场景：需要快速处理突发性强、耗时较短的任务场景

    -   ScheduledThreadPool：可调度线程池

        -   `scheduledAtFixedRate(task, time1, time2, unit)`：time1表示首次执行任务的延迟时间，time2表示每次执行任务的间隔时间
        -   使用场景：周期性地执行任务

        ```java
        ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
        scheduled.scheduledAtFixedRate(new TargetTask(), 0, 500, TimeUnit.MILLISECONDS);
        ```

    -   默认提供的线程池都有将资源占用尽的风险，因此尽管使用方便，但应该尽量避免使用

-   线程池参数

    -   corePoolSize：线程数量小于此值时，即使有空闲线程也会创建新线程
    -   maximumPoolSize
    -   keepAliveTime
    -   BlockingQueue\<Runnable>：任务队列
    -   ThreadFactory：控制线程产生方式
    -   RejectedExecutorHandler：拒绝策略

-   向线程池提交任务的两种方式

-   线程池的任务调度流程

    -   如果线程数量小于corePoolSize，直接创建新线程来执行任务
    -   否则，如果任务队列没满，则将任务加入到阻塞队列中
    -   否则，如果线程数量小于maximumPoolSize，则创建新线程来执行任务
    -   否则，执行拒绝策略
    -   当一个线程完成了任务时，会优先从阻塞队列中获取下一个任务
    -   [ ] 非核心线程是否会从阻塞队列中获取任务？
    -   [ ] 假如任务throw了一个exception，怎么办

-   ThreadFactory线程工厂

    -   可以指定线程创建的名称、线程组、优先级、守护进程等属性

-   任务阻塞队列

    -   阻塞队列与普通队列的特点
        -   在阻塞队列为空时，会阻塞当前线程的元素获取操作
        -   当队列中有元素时，被阻塞的线程会被自动被唤醒，唤醒过程不需要用户程序干预
        -   常用实现类：ArrayBlockingQueue（有界）、LinkedBlockingQueue（有界无界皆可）、PriorityBlockingQueue（无界）、DelayQueue（无界）、SynchronousQueue（不存储）

-   调度器的钩子方法（生命周期函数）：ThreadPoolExecutor提供了三种钩子方法

    -   beforeExecute()：任务执行前
    -   afterExecute()：任务执行后
    -   terminated()：线程池终止时

-   线程池的拒绝策略RejectedExecutionHandler接口

    -   任务被拒绝的情况
        -   线程池已关闭
        -   工作队列满了，线程数也够了
    -   RejectedExecutionHandler接口：以下4种常用决策的父类
    -   AbortPolicy拒绝策略
        -   直接抛出RejectedExecutionException
        -   该策略是线程池的默认拒绝策略
    -   DiscardPolicy抛弃策略
        -   直接丢弃该任务，无异常抛出
    -   DiscardOldestPolicy抛弃最老任务策略
        -   将最早进入队列的任务抛弃后，再尝试加入队列
    -   CallerRunsPolicy调用者执行策略
        -   如果新任务添加到线程池时失败，则提交任务线程会自己执行该任务

-   线程池的使用建议

    -   线程池启动后建议手动关闭
    
    -   线程池的状态
        -   RUNNING：线程池创建之后的初始状态
        -   SHUTDOWN：该状态下线程池不再接受任务，但会将队列种的任务先完成
        -   STOP：不再接受新任务，也不再处理队列中的剩余任务
        -   TIDYING：该状态下所有任务都已经终止或者处理完成，将会执行terminated()钩子方法
        -   TERMINATED：执行完terminated()钩子方法后的状态
        
    -   `shutdown()`
        -   线程池直到处理完队列中所有任务才退出
        -   状态更改为SHUTDOWN，此时不能够往线程池添加新任务
        
    -   `shutDownNow()`
        -   线程状态更改为STOP，并试图停止所有正在执行的线程，并且不再处理阻塞队列中的任务
        
    -   `awaitTermination()`
        -   让当前线程等待线程池完成关闭
        
    -   线程池的建议
        -   使用有界队列而不是无界队列
        -   使用单例模式，如果代码没有用到线程池，就不会立即创建
        
    -   优雅的关闭方式
        -   执行shutdown()拒绝新任务
        -   执行`awaitTermination()`指定超时时间，判断是否已经关闭所有任务，线程池关闭完成
        -   如果`awaitTermination()`返回false，或者被中断，就调用`shutDownNow()`方法立即关闭线程池所有任务
        -   补充执行`awaitTermination()`判断线程池是否关闭完成。如果超时，就可以进入循环关闭，循环一定的次数，不断关闭线程池，直到关闭或者循环结束
        
    -   在JVM进程关闭前，自动将线程池优雅地关闭，以确保资源正常释放
        ```java
        Runtime.getRuntime().addShutdownHook()
        ```
    
-   使用Executors创建线程池的问题
    -   FixedThreadPool的问题
        -   如果任务提交速度持续大于任务处理速度，就会造成队列中大量的任务等待
        -   如果队列很大，就会导致OOM

    -   SingleThreadExecutor的问题
        -   其工厂方法使用了FinalizableDelegatedExecutorService对该线程池进行包装，以防止线程池的corePoolSize被动态地修改
        -   与FixedThreadPool一样的问题

    -   CachedThreadPool的问题
        -   使用这种线程池需要将maximumPoolSize设置得非常大，从而使得新任务不会被拒绝
        -   如果线程数量不设上限，有可能会导致CPU线程资源耗尽

    -   ScheduledThreadPool的问题
        -   如果线程数量不设上限，有可能会导致CPU线程资源耗尽




#### 1.7 确定线程池的线程数

-   使用线程池的好处：降低资源消耗、提高响应速度、提高线程的可管理性

-   按照三种类型的任务对线程池进行分类

    -   IO密集型任务：由于执行IO操作的时间长，导致CPU的利用率不高。这类任务的CPU常处于空闲状态
    -   CPU密集型任务：执行大量计算任务，CPU的利用率很高
    -   混合型任务：比如HTTP请求操作

-   为IO密集型任务确定线程数

    -   IO处理线程数默认为CPU核数的两倍

    ```java
    SystemPropertyUtil.getInt("io.netty.eventLoopThreads"
    	, Runtime.getRuntime().availableProcessors()*2)
    ```

-   为CPU密集型任务确定线程数：线程数等于CPU数

-   为混合型任务确定线程数：最佳线程数=（线程等待时间与线程得到CPU时间之比+1）*CPU核数



#### 1.8 ThreadLocal原理与实战

-   使得变量在每个线程中都有独立值

-   ThreadLocal基本操作

    ```java
    static ThreadLocal<Value> LOCAL = new ThreadLocal<>();
    
    // main()
    Local.set(new Value(10));
    System.out.println(LOCAL.get().num);
    LOCAL.remove();
    ```

    -   设置初始值：使用工厂方法`ThreadLocal.withInitial()`或者（Java8以下版本）在初始化时重写其`initialValue()`
    -   应该使用static final来修饰ThreadLocal对象以节省内存空间
    -   设置过的ThreadLocal应该要删除绑定。可以在线程池的`afterExecute()`中将线程内部的Entry得以释放

-   ThreadLocal的使用场景

    -   线程隔离
        -   既保证了数据安全，又避免了使用Synchronized带来的性能损失
        -   经典案例：为每个线程绑定一个数据库连接，使得这个数据库连接为线程所独享

    -   跨函数传递数据

-   ThreadLocal内部结构

    -   早期Java：每个ThreadLocal拥有一个Map（ThreadLocalMap），key为线程实例，value为绑定值
    -   Java8+：每个线程拥有一个Map，key为ThreadLocal实例，value为绑定值
    -   演进优势
        -   每个ThreadLocalMap存储的键值对数量变少
        -   ThreadLocalMap会随着线程的销毁一起销毁

-   ThreadLocal原理

    -   set方法：若map为空，则创建map
    -   get方法：若map为空，或键值对不存在，则设置并返回初始值
    -   remove方法：若map不为空，则把map给remove
    -   initialValue方法：通过静态工厂方法实现

-   ThreadLocalMap原理

    -   Entry
        -   保存键值对
        -   继承自WeakReference。强引用会导致尽管Entry中指向的某个键值对不能被GC回收，会造成内存泄漏。弱引用则能使未被强引用指向的键值对能够被顺利回收
        -   内存泄漏是不再用到的内存没有及时释放。弱引用指向的对象只能生存到下一次GC回收之前




[TOC]

## #2 Java内置锁的核心原理

-   Java内置锁
    -   互斥锁，意味着只有一个线程能够得到该锁
    -   内置锁，是指每个对象都含有一把锁，都可以用作锁

#### 2.1 线程安全问题

-   线程安全：当多个线程并发访问某个Java对象时，无论系统如何调度这些线程，也无论这些线程将如何交替操作，这些对象都能表现出一致的、正确的行为

-   CountDownLatch倒数闩

    -   直到倒数闩的次数减为0，调用线程才可以继续执行
    -   实现了多并发等待的效果

    ```java
    // fun:testNotSafePlus
    CountDownLatch latch = new CountDownLatch(MAX_THREAD);
    NotSafePlus counter = new NotSafePlus();
    for (int i = 0; i < MAX_THREAD; i++) {
        new Thread(()->{
            for (int j = 0; j < MAX_TURN; j++) {
                counter.selfPlus();
            }
            latch.countDown();
        }).start();
    }
    // 用于等待
    latch.await();
    // 有时有差距，有时没有
    System.out.println("实际结果："+counter.num);
    System.out.println("差距："+(MAX_THREAD*MAX_TURN-counter.num));
    
    // 使用自增运算符的类
    class NotSafePlus{
        int num = 0;
        
        public void selfPlus(){
            num++;
        }
    }
    ```

-   `a++`自增运算符不是线程安全的

    -   自增运算符包含了至少三个JVM指令：内存取值、寄存器增加一、将值保存到内存
    -   这些指令在JVM内部是独立进行的，中间完全可能会出现多个线程并发执行

-   临界区资源与临界区代码段
    -   临界区资源是一种可以被多个线程使用的公共资源或共享对象，但是每一次只能有一个线程在使用它
    -   临界区代码段是每个线程中访问临界资源的代码段，多个线程必须互斥地对临界区资源进行访问。进入代码段前要申请资源，退出后则要释放资源
    -   竞态条件：可能是由于在访问临界区代码段时没有互斥地访问而导致的特殊情况
    -   在上述代码testNotSafePlus中，num是临界区资源，`selfPlus()`则是临界区代码段
    -   使用synchronized关键字解决问题



#### 2.2 synchronized关键字

-   对于小的临界区，可以直接在方法声明中设置synchronized同步关键字

-   对于大的临界区，则应该将同步方法分为小的临界区代码段

    -   同步方法是一种粗粒度的并发控制
    -   synchronized代码块则是一种细粒度的并发控制

    ```java
    // 同步方法
    public synchronized void plus(int val1, int val2){
        this.num1 += val1;
        this.num2 += val2;
    }
    
    // 使用同步块进行优化
    // 使用两把锁
    private Integer sum1Lock = new Integer(1);
    private Integer sum2Lock = new Integer(2);
    public void plus(int val1, int val2){
    	synchronized(this.sum1Lock){
            this.sum1 += val1;
        }
        synchronized(this.sum2Lock){
            this.sum2 += val2;
        }
    }
    ```

-   静态的同步方法

    -   使用synchronized关键字修饰static方法时，用的是其对应Class对象的监视锁（类锁）。这是非常粗粒度的同步机制

[TOC]

#### 2.3 生产者-消费者问题

-   生产者-消费者问题，也叫有限缓冲问题，是一个多线程同步问题的经典案例
    -   生产者线程的主要功能是生成一定量的数据放到缓冲区中，然后重复此过程
    -   消费者线程的主要功能是从缓冲区中提取/消耗数据
    -   问题的关键
        -   保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区空时消耗数据
        -   保证在生产者加入过程、消费者消耗过程中，不会产生错误的数据和行为

-   生产者-消费者模式
    -   有若干个生产者线程和消费者线程。生产者与生产者之间、消费者与消费者之间，对数据缓冲区的操作是并发执行的
    -   数据缓冲区是有容量上限的。数据缓冲区满后，生产者不能再加入数据；数据缓冲区空时，消费者不能再取出数据
    -   线程缓冲区是线程安全的。在并发操作数据缓冲区的过程中，不能出现数据不一致的情况；或者在多个线程并发更改共享数据后，不会造成出现脏数据的情况
    -   生产者或消费者线程在空闲时需尽可能阻塞而不是执行无效的空操作，尽量节约CPU资源
    -   默认生产者-消费者模式
    
    ```mermaid
    classDiagram
    DataBuffer <|..Producer
    DataBuffer *--LinkedList
    DataBuffer <|..Consumer
    ```
    
    -   对应动作解耦后的生产者-消费者模式
    
        ```mermaid
        classDiagram
        Producer*--Callable生产动作
        DataBuffer <|..Callable生产动作
        DataBuffer *--LinkedList
        DataBuffer <|..Callable消费动作
        Consumer*--Callable消费动作
        ```
    
-   实现线程安全的生产者-消费者模式

    -   低效的解决方法
        -   为临界区代码加上synchronized关键字即可，具体的临界区代码为DataBuffer的add方法和fetch方法
        -   这样，生产和消费过程都需要抢占同一个锁




#### 2.4 Java对象结构与内置锁

-   Java对象结构：包括了三个部分
    -   对象头：包括了三个字段
        -   Mark Word标记字：用于存储自身运行时的数据，例如CG标志位、哈希码、锁状态等
        -   Class Pointer类对象指针：用于存放方法区Class对象的地址。JVM通过这个指针来确定这个对象是哪个类的实例
        -   Array Length数组长度：如果对象是一个数组，才会有此字段，用于记录数组长度的数据
    -   对象体：包含了成员变量，包括父类的成员变量
    -   对齐字节：占位符，用于保证Java对象所占内存字节数为8的倍数。HotSpot VM的内存管理要求对象的起始地址必须是8字节的整数倍







## #3 CAS原理与JUC原子类

#### 3.1 CAS

-   并发包JUC对操作系统的底层CAS原子操作进行了封装，为上层Java程序提供了CAS操作的API

-   unsafe类：主要提供一些用于执行低级别、不安全的底层操作，如直接访问系统内存资源、自主管理内存资源等。其中大量的方法都是native方法

-   操作系统层面的CAS是一条CPU的原子指令`cmpxchg`，而该指令具备原子性

-   完成Java应用层CAS操作的步骤原理

    -   获取Unsafe实例

        -   Unsafe类是一个final修饰的最终类，且其构造函数是private的
        -   只能通过反射的方式获取Unsafe实例

        ```java
        public static Unsafe getUnsafe(){
            try {
                Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
                theUnsafe.setAccessible(true);
                return (Unsafe)theUnsafe.get(null);
            } catch (Exception e) {
                throw new AssertionError();
            }
        }
        ```

    -   调用Unsafe提供的CAS方法

        -   提供的三个CAS方法包括了`compareAndSwapObject()`、`compareAndSwapInt()`、`compareAndSwapLong()` 
        -   上述三个方法包括了四个参数：字段所在的对象、字段内存位置（偏移量）、预期原值、新值

    -   调用Unsafe提供的字段偏移量方法

        -   提供的方法`staticFieldOffset()`、`objectFieldOffset()`，前者用于获取静态属性Field在Class对象中的偏移量，后者则是成员属性中的Field

-   使用CAS进行无锁编程

    -   步骤

        -   获得字段的期望值
        -   计算出需要替换的新值
        -   通过CAS将值放在字段上，如果失败就重复前两步直到成功。这种重复叫做CAS自旋

        ```java
        public synchronized void selfPlus(){
            // num++;
            int oldValue;
            do{
                oldValue = num;
            }while(!unSafeCompareAndSet(oldValue, oldValue+1));
        }
        ```

-   字段偏移量的计算：对象头共占了12个字节，因此第一个属性的字段偏移量是12



#### 3.2 JUC原子类

-   与synchronized相比，JUC原子类是基于CAS轻量级原子操作的实现，使得程序运行效率更高
-   4种原子类
    -   基本原子类：通过原子方式更新Java基础类型的值
    -   数组原子类：通过原子方式更新数组中某个元素的值
    -   引用原子类：通过原子方式更新引用类型。使用AtomicMarkableReference\AtomicStampledReference类甚至可以解决AtomicBoolean\AtomicInteger进行原子更新时可能出现的ABA问题
    -   字段更新原子类：可以原子更新Integer、Long、Refernence中的值\引用
-   AtomicInteger线程安全原理
    -   通过CAS自旋+volatile的方案实现，既保障了变量操作的线程安全性，又避免了synchronized重量级锁的高开销




#### 3.3 对象操作的原子性

-   基础的原子类型只能保证一个变量的原子操作，而引用原子类AtomicReference可以保证对象引用的原子性

    -   只能保障User引用的原子操作，User成员值被修改时不能保证原子性

    ```java
    AtomicReference<User> ref = new AtomicReference<User>();
    User user = new User("1", "张三");
    ref.set(user);
    
    User next = new User("2", "李四");
    ref.compareAndSet(user, next);
    ```

-   要使对象成员更新也具有原子性，则应该使用字段更新原子类

    ```java
    AtomicIntegerFieldUpdater<User> updater = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
    User user = new User("1", "张三", 0);
    updater.getAndAdd(user, 100); // 100
    ```

    

#### 3.4 ABA问题

-   ABA问题
    -   当前线程的活被其他线程干了，但因为其结果和没干之前一样，所以当前线程并不知道活已被干了，于是又做了一遍
-   使用版本号解决ABA问题
    -   版本号每次都会成功执行后都会增加
-   JDK提供了AtomicStampledReference、AtomicMarkableReference来解决ABA问题





## #4 可见性与有序性的原理

#### 4.1 CPU物理缓存结构

-   由于CPU运行速度比主存（物理内存）的存取速度快很多，为了提高处理速度，现代CPU不直接和主存进行通信，而是在CPU和主存之间设计了多层的Cache（高速缓存）
-   CPU高速缓存有L1和L2、L3...高速缓存
    -   每一块高速缓存中所存储的数据都是下一级高速缓存的一部分
    -   越靠近CPU的高速缓存读取越快，容量也越小。L1最靠近CPU
-   CPU内核读取数据时，先从L1高速缓存中读取，如果没有命中，再到L2、L3中读取，最后才到物理内存中读取所需数据
-   CPU通过高速缓存进行数据读取的优势
    -   写缓存区可以保证指令流水线持续运行，可以避免由于CPU停顿下来等待向内存写入数据而产生的延迟
    -   通过批量处理的方式进行刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，减少对内存总线的占用



#### 4.2 并发编程的三大问题

-   由于需要尽可能地释放CPU的能力，因此在CPU上不断增加内核和缓存。随着CPU内核和缓存的增加，导致了并发编程的可见性和有序性问题

-   并发编程的三大问题：原子性、可见性、有序性

-   原子性：不可中断的一系列操作

    -   使用javap命令解析class文件可以得到其汇编指令信息
        -   可以发现`++`操作实际上是4个操作（取、获取常数1、加、放）
        -   这4个操作之间是可以发生线程切换的，所以`++`操作不是原子操作，在并发场景中会发生原子性问题

-   可见性：一个线程对共享变量的修改，使得另一个对象能够立刻看见

    -   Java内存模型JMM（Java Memory Model）规定，所有的变量都存放在公共主存中，当线程使用变量时会将主存中的变量复制到自己的私有内存中，线程对变量的读写操作，是自己工作内存中的变量副本
    -   使用volatile关键字令所有线程都必须将共享变量刷新到主内存
    -   可见性问题只出现在共享变量上
        -   局部变量、方法参数不会在线程之间共享
        -   Object实例、Class实例和数组元素都存储在JVM堆内存上，堆内存在线程之间共享，所以存在可见性问题

    ```java
    private static boolean shutdown = false;
    static class HelloThread extends Thread{
        @Override
        public void run() {
            while(!shutdown){
                // nothing
            }
            System.out.println("再见");
        }
    }
    
    // 可见性问题  - 之 -  讲不出“再见”
    public static void main(String[] args) throws InterruptedException {
        new HelloThread().start();
        Thread.sleep(1000);
        shutdown = true;
        System.out.println("exit main");
    }
    ```

-   有序性：程序按照代码的先后顺序执行，如果程序执行的顺序与代码的先后顺序不同，并导致了错误，即发生了有序性问题

    -   指令重排序：CPU为了提高程序运行效率，可能会对输入代码进行优化，这不保证各个语句的执行顺序同代码中的先后顺序一致，但是会保证最终的执行结果和代码顺序执行的结果是一致的
    -   重排序是在单核时代下非常优秀的优化手段。在多核时代，如果工作线程之间不共享数据或仅共享不可变数据，重排序也是性能优化利器；但会影响多线程并发执行的正确性



#### 4.3 硬件层的MESI协议原理

-   volatile的底层原理：在汇编指令中的操作多了一个lock前缀指令lock addl。该前缀指令有三个功能
    -   将当前CPU缓存行的数据立即写回系统内存
    -   l引起在其他CPU中缓存了该内存地址的数据无效
    -   禁止指令重排：作为内存屏障使用



#### 4.4 有序性与内存屏障

-   内存屏障（内存栅栏Memory Fences）：是一系列CPU指令，作用主要是保证特定操作的执行顺序

-   两类重排序
    -   编译器重排序：代码编译阶段进行指令重排
        -   场景：A操作需要获取其他资源而进入等待状态，而A操作后面的代码跟A操作没有多大关系，所以编译器可以先编译后面的代码以提升编译速度
    -   CPU重排序
        -   流水线（Pipeline）和乱序执行（Out-of-Order Execution）是现代CPU基本都具有的特性
        -   为了CPU的执行效率，流水线都是并行处理的。在不影响语义的情况下，处理次序（CPU执行机器指令的顺序）和程序次序（程序代码的逻辑执行顺序）是允许不一致的，只要满足As-if-Serial规则即可
        -   指令集重排序和内存系统重排序



#### 4.7 volatile不具备原子性

-   volatile的两个重要语义
    -   使用volatile修饰的变量在变量值发生变化时，会立刻同步到内存，并使其他线程的变量副本失效
    -   禁止指令重排序：硬件层面上在指令前后加入内存屏障来实现





## #5 JUC显示锁的原理与实战

## #6 AQS抽象同步器的核心原理

## #7 JUC容器类

## #8 高并发设计模式

## #9 高并发核心模式之异步回调模式

## #10 CompletableFuture异步回调

