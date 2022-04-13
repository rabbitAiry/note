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

    -   类型参数的限定：通过extends来表示

        -   上界为某个类、某个接口

        -   限定类型后，就可以使用该类型的方法了

            ```java
            // 上界为某个类
            public class Pair<U extends Number, V extends Number>{
                U first;
                V second;
                public Pair(U first, V second){
                    this.first = first;
                    this.second = second;
                }
                
                // 允许使用Number的方法
                public double sum(){
                    return first.doubleValue()+second.doubleValue()
                }
            }
            ```

    -   类型参数限定 - 上界为其他类型

        -   虽然Integer是Number的子类，但是MyArray\<Integer>不是MyArray\<Number>的子类。所以以下方法1会提示编译出错

        -   通过类型限定来解决

            ```java
            // 以下是两个整个MyArray的元素添加进另一个MyArray的方法
            // 1. 会提示编译错误
            public void addAll(MyArray<E> c){
                // ...
            }
            
            // 2. 正确写法
            public <T extends E> void addAll(MyArray<T> c){
                // ...
            }
            
            // 3. 使用通配符——等价于方法2
            public void addAll(MyArray<? extends E> c){
                // ...
            }
            
            MyArray<Number> numbers = new MyArray<>();
            MyArray<Integer> ints = new MyArray<>();
            numbers.addAll(ints);
            ```

-   泛型的通配符

    -   通配符形式更为简洁，但上面两种通配符都有一个重要的限制：只能读，不能写




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
    -   竞态条件：多个线程同时修改同一个变量
    
    -   内存可见性
    
        -   线程队内存的修改，在另一个线程中看不到
        -   另一个线程没从内存读取
    
        >   计算机中，除了内存，数据还会被缓存在CPU的寄存器以及各级缓存器中。当访问一个变量时，未必会到内存中去获取
    
        ```java
        private static boolean shutdown = false;
        static class HelloThread extends Thread{
            @Override
            public void run() {
                while(!shutdown){
                    // nothing
                }
                System.out.println("exit hello");
            }
        }
        
        // 等不到的“exit hello”
        public static void main(String[] args) throws InterruptedException {
            new HelloThread().start();
            Thread.sleep(1000);
            shutdown = true;
            System.out.println("exit main");
        }
        ```
    
-   线程的优点及成本

    -   优点
        -   充分利用多CPU的计算能力
        -   成分利用硬件资源：CPU、硬盘、网络是可以同时工作的
        -   GUI中保持程序的响应性：让界面不卡顿
        -   简化建模及IO处理：多种用户请求写在一个线程里会显得很复杂

    -   缺点
        -   创建线程需要消耗系统的资源：操作系统会为每个线程创建必要的数据结构、栈、程序计数器等
        -   线程的调度和切换有成本：切换线程时，操作系统需要保存及恢复线程的上下文状态到内存
        -   创建超过CPU数量的线程是没有必要的，并不会加快程序的运行


##### 15.2 理解synchronized

-   synchronized可以用于修饰：
    -   实例方法
    
        -   synchronized保护的是同一个对象的方法调用
        -   不同对象的同一synchronized方法，可被多个线程同时访问
        -   不能获得锁的线程会被假如等待队列，但释放锁后，从中选取一个线程将其唤醒获得锁。不保证公平性（竞争）
        -   synchronized保护的是对象。只要线程访问的是同一个对象的synchronized方法，即使是不同的代码，也会被同步执行
    
        ```java
        // 15.2_a
        // 线程A进入getCount()后，线程B不能够进入同一对象的incr()，但可以进入desc()
        public class Test{
            static int num;
            int count;
        
            // 被同步
            public synchronized int getCount(){
                return count;
            }
        
            // 被同步
            public synchronized void incr(){
                count++;
            }
        
            // 不被同步
            public void decr(){
                count--;
            }
        
            // 不被同步，这是静态方法，synchronized保护的是Test.class
            public static synchronized void incr(){
                num++;
            }
        }
        ```
    
    -   静态方法：保护类对象
    
    -   代码块
    
        -   圆括号内是保护的对象
        -   同步的对象可以是任意对象，因为任意对象都有一个锁和等待队列
    
        ```java
        // 15.2_b
        public class Test{
            static int num;
            Object lock = new Object();
            int count;
        	
            // getCount
            public synchronized int getCount(){
                synchronized(this){
                    return count;
                }
            }
            
            // getCount，这里与前者等价
            public synchronized int getCount(){
                synchronized(lock){
                    return count;
                }
            }
        
            // 保护Test.class
            public static synchronized void incr(){
                synchronized(Test.class){
                    num++;
                }
            }
        }
        ```
    
-   synchronized特性
    -   可重入性：并不是所有锁都可重入
    
        -   如15.2_a示例中，当线程获得了锁访问`incr()`后，可以直接调用`getCount()` 
        -   可重入是通过记录锁的持有线程和持有数量来实现的
        -   当线程调用synchronized方法时，会先检查对象是否被锁，然后检查是否被自己线程锁定，如果是，则增加持有数量
        -   当释放锁时，减少持有数量，当数量变为0时才释整个锁
    
    -   内存可见性
    
        -   在释放锁时，所有写入都会写回内存
        -   在获得锁后，都会从内存中读最新数据
        -   使用synchronized来保证内存可见性的成本有点高。使用volatile是更轻量的方法
    
        ```java
        private volatile boolean on;
        public boolean isOn(){
        	return on;
        }
        ```
    
    -   死锁：互相等待
    
        -   应该避免在持有一个锁的同时去申请另一个锁
        -   如果确实需要多个锁，则应该所有的代码都按照相同的顺序去申请锁
        -   Java自带的jstack命令会报告发现的死锁。但程序中，Java不会主动处理
    
-   同步容器及其注意事项
    
    -   Collection中有一些方法，可以返回线程安全的同步容器
    
    -   通过给容器Collection方法加上synchronized实现安全的同步容器
    
        ```java
        // 装饰类EnhancedMap
        public class EnhancedMap<K, V>{
        	Map<K, V> map;
        	public EnhancedMap(Map<K, V> map){
        		this.map = Collections.synchronizedMap(map);
        	}
        	
            // 1
        	public V putIfAbsent(K key, V value){
        		V old = map.get(key);
        		if(old!=null)return old;
        		return map.put(key, value);
        	}
            
            // 2
        	public synchronized V putIfAbsent(K key, V value){
        		V old = map.get(key);
        		if(old!=null)return old;
        		return map.put(key, value);
        	}
        	
        	public V put(K key, V value){
        		return map.put(key, value);
        	}
        }
        ```
    
    -   同步容器并非绝对安全，需要注意以下情况
    
        -   复合操作：代码段1中，`putIfAbsent()`包含了检查和更新数据的操作，多线程情况下不安全
        -   伪同步：代码段2中，尽管加上了synchronized，但是同步错对象了。`putIfAbsent()`同步的是EnhancedMap对象，而其他方法使用的是map对象的锁。要解决这个问题，则所有方法必须使用相同的锁。
        -   迭代：如果在遍历的同时容器发生了结构性变化，就会抛出ConcurrentModificationException异常
        -   同步容器性能较低，推荐使用并发容器
    

##### 15.3 线程的基本协作机制

-   wait/notify
    -   Object定义了这两类方法
    -   两个`wait()`
        -   `wait()`：无限期等待
        -   `wait(long timeout)`：最长等待时间
        -   调用`wait()`会把当前线程放到条件队列上并阻塞，表示当前线程执行不下去了，需要等待一个条件
        -   这个条件自身无法更改，需要其他线程更改。当其他线程更改了条件后，应该调用`notify()` 
    -   两个`notify()`
        -   `notify()`：从条件队列中选一个线程，将其从队列中移除并唤醒
        -   `notifyAll()`：移除条件队列中所有的线程并全部唤醒
    -   wait/notify方法只能在synchronized代码块中调用这连个方法
        -   否则会抛出异常`java.lang.illegalMonitorStateException`
        -   在调用`wait()`时，线程会释放对象锁

-   wait/notify具体过程

    -   把当前线程放入条件等待队列，释放对象锁，阻塞等待
    -   线程状态变为WAITING或TIME_WAITING
    -   等待时间到或被其他线程调用notify/notifyAll，这时要重新竞争对象锁
        -   如果原线程能够获得锁，线程状态变为RUNNABLE，并从wait调用中返回
        -   否则加入对象锁等待队列，线程状态变为BLOCKED


```java
public class Temp extends Thread{
    private volatile boolean fire = false;

    @Override
    public void run() {
        try{
            synchronized(this){
                while(!fire) wait();
            }
            System.out.println("fired");
        }catch(InterruptedException e){}
    }

    public synchronized void fire(){
        this.fire = true;
        notify();
    }

    public static void main(String[] args) throws InterruptedException {
        Temp temp = new Temp();
        temp.start();
        Thread.sleep(1000);
        System.out.println("fire");
        temp.fire();
    }

}

// 只有fire==true时且notify()被调用，才能让线程停止等待
/* output:

fire
fired
*/
```

>   协作的场景

-   生产者/消费者模式
-   同时开始
-   等待结束
-   异步结果
-   集合点：所有线程到齐后，交换数据和计算结果，再进行下一次迭代

##### 15.4 线程的中断

-   场景：程序停止时、用户取消任务、执行任务超时、多渠道执行任务时，其中一个完成了
-   停止一个线程的主要机制：中断
    -   中断给线程传递了一个取消信号，但是由线程来决定何时及如何退出
    -   中断方法`interrupt()` 

-   中断对线程的影响与线程的状态和在进行的IO操作有关
    -   RUNNABLE
        -   调用`interrupt()` 会设置中断标志位
        -   应在代码中通过循环内使用`isInterrupted()`查询是否有中断标志位
    -   WAITING/TIME_WAITING
        -   线程调用join/wait/sleep方法会进入该状态
        -   此时调用`interrupt()` 会抛出中断异常
        -   抛出中断异常后，中断标志位会被清空
    -   BLOCKED：调用`interrupt()` 会设置中断标志位
    -   NEW/TERMINATE：调用后无效果
    -   在使用synchronized关键字的过程中不响应中断请求。使用显式锁Lock以解决问题
-   正确操作



[TOC]

### #16 并发包的基石

>   Java并发工具包

##### 16.1 原子变量和CAS

>   使用原子变量改善使用synchronized的简单任务
>
>   AtomicBoolean \AtomicInteger \AtomicLong \AtomicReference \ ...

-   原子变量，主要介绍AtomicInteger

    -   提供一些原子方式实现组合操作的方法，如：`getAndSet(int newValue)`、`getAndAdd(int delta)`、`addAndGet(int delta)`

    -   组合操作的方法实现依赖于`compareAndSet(int expect, int update)`，简称CAS

        -   expect为当前值，update为当前值更新后的结果
        -   调用CAS方法进行更新，如果更新失败，说明数据被别的线程更改了，则再去取最新值并尝试更新直到成功为止

        ```java
        // 组合操作方法的实现
        private volatile int value;
        
        //...
        
        public final int incrementAndGet(){
        	for(;;){
                int current = get();
                int next = current+1;
                if(compareAndSet(current, next)){
                    return next;
                }
            }
        }
        ```

    -   乐观锁

        -   synchronized是悲观锁，代表一种阻塞式算法，有上下文切换开销
        -   原子变量是乐观锁，使用CAS进行冲突检测，更新逻辑是非阻塞式的

    -   CAS的实现：硬件层次上直接支持CAS指令，可以看作是计算机的基本操作（unsafe类）

-   ABA问题：另一个线程将当前值A修改为B后，又重新修改为A，但是线程的CAS操作无法分辨当前值发生过变化

    -   通过引用时间戳解决


##### 16.2 显式锁

-   显式锁接口Lock

    ```java
    public interface lock(){
    	void lock();		// 获取锁，该方法会阻塞直到成功
    	void lockInterruptibly() throws InterruptedException;
    	boolean tryLock();	// 尝试获取锁，不阻塞
    	boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    	void unlock();		// 释放锁
    	Condition newCondition();
    }
    ```

-   可重入锁ReentrantLock
    -   lock/unlock实现了与synchronized一样的语义

        -   可重入、可以解决竞态条件问题（保证公平）、可以保证内存可见性

        -   在ReentrantLock的构造时，通过将fair参数指定为true以保证公平，使得等待锁的队列先到先得

            ```java
            public ReentrantLock()				// 默认不公平
            public ReentrantLock(boolean fair)
            ```

        -   一定要记得调用unlock

    -   使用tryLock避免死锁

        -   案例：账号A、B都有锁，账号A转账给账号B的同时，账号B也在转账给账号A
        -   使用`tryLock()`尝试获取锁，不能获取则放弃本次操作，释放自身所有锁

        ```java
        boolean tryTransfer(Account from,Account to,int val){
        	if(from.tryLock()){
                try{
        			if(to.tryLock()){
                        try{
                        	if(from.getMoney()>=money){
                        		from.reduce(money);
                        		to.add(money);
                    		}else{
                                throw new NoMoneyException()
                            }
                            return true;
                        }finally{
                            to.unlock();
                        }
                    }
                }finally{
                    from.unlock();
                }
            }
            return false;
        }
        ```

        ```java
        void transfer(Account from,Account to,int val) throws NoMoneyException{
            boolean success = false;
            do{
                sucess = tryTransfer(from, to, val);
                if(!success){
                    Thread.yield();
                }
            }while(!success);
        }
        ```

-   显式锁ReentrantLock实现原理
    -   LockSupport

        ```java
        // 基本方法
        park()				// 使得当前线程放弃CPU，进入等待状态
        parkNanos(long Nanos)
        parkUntil(long deadline)
        unpark(Thread thread)	// 恢
        ```

        ```java
        // temp线程示例
        Temp temp = new Temp(){
            @Override
            public void run() {
                LockSupport.park();
                System.out.println("exit");
            }
        };
        temp.start();
        Thread.sleep(1000);
        LockSupport.unpark(temp);
        ```

        -   `park()`与`yield()`不同，后者只是告诉操作系统可以先让其他线程运行，而自己依然是可运行状态，而前者会放弃调度资格，使线程进入WAITING状态
        -   `park()`是响应中断的，当有中断发生时，返回
        -   parkNanos：等待的最长时间
        -   parkUntil：指定最长等待到什么时候
        -   与CAS一样，LockSupport调用了操作系统的API

    -   AbstractQueued-Synchronizer抽象类，简称AQS

        -   简化了并发工具的实现
        -   AQS封装了一个状态，给子类提供了查询和设置状态的方法

-   ReetrantLock的实现，内部使用AQS

    -   3个内部类，1个Sync成员
        -   3个内部类：抽象类Sync、NonfairSync是构造参数fair为false时使用的类、FairSync是fair为true时使用的类
        -   1个Sync成员：用于根据参数fair指向对应的Sync成员
    -   lock的实现
        -   ReentrantLock使用State表示是否被锁和持有数量，如果当前未被锁定，则立即获得锁
        -   否则调用AQS的`acquire(1)`获得锁
        -   如果还是不能获得锁，则调用`acquireQueued(addWaiter(Node.EXCLUSIVE), arg)`。其主体是是循环，如果能获得锁，则返回；否则调用`LockSupport.park`放弃CPU，等待被唤醒，检查自己是否是第一个等待的线程，若否则继续等
    -   保证公平让活跃线程得不到锁，加入等待状态，引起频繁的上下文切换

-   与synchronized对比

    -   synchronized代表一种声明式编程思维，而显式锁代表一种命令式编程思维
    -   synchronized更简单，且能被编译器和JVM优化，满足需求时应该首选


##### 16.3 显式条件

-   用法
-   生产者/消费者模式
-   原理

[TOC]

### #17 并发容器

##### 17.1 写时复制的List和Set

-   CopyOnWriteArrayList/Set写时复制的List和Set
    -   线程安全的，可被多个线程并发访问
    -   其迭代器不支持修改操作，但也不会抛出ConcurrentModificationException
    -   以原子方式支持一些复合操作
    -   迭代时不需要加锁
    
-   CopyOnWriteArrayList

    -   两个原子方法

        ```java
        // 不存在才添加，如果添加了，返回true
        public boolean addIfAbsent(E e)
        // 批量添加c中的非重复元素，不存在才添加，返回实际添加的个数
        public int addAllAbsent(Collection<? extends E> c)
        ```

    -   CopyOnWriteArrayList内部也是一个数组，但这个数组是以原子方式被整体更新的

        -   每次修改操作，都会新建一个数组，复制原数组的内容到新数组，在新数组上进行修改，然后以原子方式设置内部的数组引用
        -   数组内容是只读的
        -   性能低，不适用与数组很大且修改频繁的场景。适用于写操作发生次数少的场景

-   CopyOnWriteArraySet：内部通过CopyOnWriteArrayList实现

##### 17.2 ConcurrentHashMap

-   并发版本的HashMap
    -   并发安全
    -   直接支持一些原子复合操作
    -   支持高并发，读操作完全并行，写操作支持一定程度的并行
    -   与同步容器Collections.synchronizedMap相比，迭代不用加锁，不会抛出ConcurrentModificationException
    -   弱一致性
-   并发安全
-   原子复合操作
-   高并发的基本机制
-   迭代安全
-   弱一致性：在遍历过程中，map内部元素可能发生变化。如果在未遍历过的部分，迭代器不会反映出来

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

-   Runnable和Callable：表示要执行的异步任务

    -   Runnable没有返回结果，而Callable有
    -   Ruunable不会抛出异常，而Callable会

-   Executor和ExecutorService：表示执行服务

    -   Executors是其工厂类
    -   Executor表示最简单的执行任务

    ```java
    public interface Executor{
    	void execute(Runnable command);
    }
    ```

    -   ExecutorService扩展了Executor，定义了更多服务
        -   返回Future后，表示任务已提交，不代表已执行
        -   通过Future可以查询异步任务状态、获取最终结果、取消任务等等

    ```java
    public interface ExecutorService extends Executor{
        <T> Future<T> submit(Callable<T> task);
        <T> Future<T> submit(Runnable task, T result);
        Future<?> submit(Runnable task);
        // ...其他方法
    }
    ```

-   Future：表示异步任务的结果

    -   get方法用于返回异步任务最终的结果。如果任务还未执行完成，会阻塞等待
    -   get方法2可以限定阻塞等待的时间，若超时任务还未结束，则抛出TimeoutException
    -   cancel方法用于取消异步任务，如果任务已完成/取消/不能被取消，则返回false

    ```java
    public interface Future<V>{
    	boolean cancel(boolean mayInterruptIfRunning);
        boolean isCancelled();
        boolean isDone();
        V get() throws InterruptedException, ExecutionException;
        V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    }
    ```

-   基本实现原理

    -   AbstractExecutorService：ExecutorService的抽象实现类
    -   FutureTask

##### 18.2 线程池

-   线程池里面有若干线程，它们的目的就是执行提交给线程池的任务，执行完一个任务后不退出，而是继续等待或执行新任务
    -   优点
        -   重用线程，避免线程创建的开销
        -   任务过多时，通过排队避免创建过多线程，减少系统资源消耗和竞争，确保任务有序完成

    -   实现类：ThreadPoolExecutor
    -   任务队列：保存待执行任务
    -   工作者线程：主体是一个循环，从任务队列中接受任务并执行
-   线程池大小
    -   参数
        -   corePoolSize：核心线程个数
        -   maximumPoolSize：最大线程个数
        -   keepAliveTime和unit：空闲线程存活时间
    -   线程的数量会动态变化，但不会超过maximumPoolSize
    -   创建一个线程池后，不会立即创建任何线程
    -   有新任务到来时，如果当前线程个数小于corePoolSize，则会创建一个新线程来执行任务。即使其他线程是空闲的
    -   如果线程数量大于corePoolSize，则会尝试排队（尝试，而不是阻塞等待）
        -   如果队列满了，则先检查线程个数是否达到了maximumPoolSize，并创建线程
    -   keepAliveTime：超时后线程被终止
-   任务队列：类型为BlockingQueue
    -   如果使用无界队列，则线程个数最多只能达到corePoolSize，参数maximumPoolSize失去意义
    -   如果使用SynchronousQueue时，因为其没有实际存储元素的空间，所以当尝试排队时有空闲线程才能入队成功
-   任务拒绝策略
    -   队列已满、线程个数达到最大值时提交任务，会触发任务拒绝策略
    -   默认情况下，此时提交任务会抛出异常RejectedExecutionException
    -   ThreadPoolExecutor实现了4种处理方式
        -   ThreadPoolExecutor.AbortPolicy：默认方式
        -   ThreadPoolExecutor.DiscardPolicy：静默处理，忽略新任务，不抛出异常，也不执行
        -   ThreadPoolExecutor.DiscardOldestPolicy：将等待时间最长的任务扔掉，然后自己排队
        -   ThreadPoolExecutor.CallerRunsPolicy：在任务提交者线程中执行任务，而不是交给线程池中的线程执行

    -   拒绝策略只有在队列有界，且maximumPoolSize有限的情况下才会触发
-   线程工厂ThreadFactory
-   关于核心线程的特殊配置
    -   线程个数小于等于corePoolSize时，称这些线程为核心线程
    -   核心线程不会因为空闲而终止，keepAliveTime参数默认不适用于核心线程
-   工厂类Executors可以方便地创建一些预配置的线程池
    -   FixedThreadPool
        - 线程数量固定的线程池，只有核心线程
        - 任务队列也无大小限制
        - 因为线程不会被回收，所以能够更快地响应外界请求
    -   CachedThreadPool
        - 线程数量不定的线程池。只有非核心线程，且最大线程数为Integer.MAX_VALUE（相对于无限大）
        - 所有的空闲线程都会有超时机制，默认为60秒
        - 其队列相对于一个空集合，这会导致所有的任务都会被立即执行（SynchronousQueue）
        - 适用于执行大量的耗时较少的任务
    -   ScheduledThreadPool
        - 核心线程数量是固定的，而非核心线程数是没有限制的
        - 当非核心线程闲置时会被立即回收
        - 适用于执行定时任务和具有固定周期的重复任务
    -   SingleThreadExecutor
        - 只有一个核心线程，确保所有的任务都在该同一个线程中按顺序执行，使得在这些任务之间不需要处理线程同步的问题

-   线程池中的死锁

##### 18.3 定时任务及其陷阱

-   Java中两种实现定时任务的方式
    -   Timer和TimerTask
    -   Java并发包的ScheduledExecutorService
-   Timer和TimerTask
    -   使用
    -   原理
        -   Timer内部主要由任务队列和Timer线程两部分组成。一个Timer对象只有一个Timer线程
        -   Timer线程主体是一个循环，不断从队伍中获取或等待任务
        -   只有完成一个任务后才执行下一个
    -   异常处理
        -   在执行任何任务中遇到抛出异常，Timer线程就会退出，从而所有定时任务都会被取消
        -   Timer会因此抛出异常
-   ScheduledExecutorService
    -   原理
        -   基于线程池实现，有多个线程可以执行定时任务
        -   任务队列是一个无界的优先级队列
    -   异常处理
        -   TaskB抛出异常并不影响TaskA，但ScheduledExecutorSerivce不会因此抛出异常





[TOC]

### #19 同步和协作工具类

>   19.1-19.4皆是基于AQS实现

##### 19.1 读写锁ReentrantReadWriteLock

-   在读操作中可以并行，只要操作中包括写就不可以并行
-   分为读锁和写锁

##### 19.2 信号量Semaphore

>   使用场景：对不同等级的账户，限制不同的最大并发访问数

-   可以限制对资源的并发访问数

##### 19.3 倒计时门栓CountDownLatch

-   倒计时变为0后，门栓打开，等待的所有线程都可以通过
-   打开后就不能关上了，所以CountDownLatch是一次性的

##### 19.4 循环栅栏CyclicBarrier

-   适用于并行迭代计算，每个线程负责一部分计算，然后再栅栏处等待其他线程完成。所有线程到齐后，交换数据和计算结果，再进行下一次迭代

##### 19.5 理解ThreadLocal

-   线程本地变量ThreadLocal
    -   线程本地变量：每个线程都有一个变量的独有拷贝，这个变量不会被其他线程所更改
    -   initialValue：初始值。threadLocal设置值前或删除值后都会为当前值
-   使用场景
    -   日期处理
        -   日期和时间的操作不是线程安全的
        -   通过ThreadLocal，每个线程使用自己的DateFormat，这样解决了线程安全问题
    -   随机数
        -   使用ThreadLocal减少竞争
        -   Random是线程安全的，但是如果并发访问激烈的话，性能会下降。所以Java提供了类ThreadLocalRandom，通过静态方法current获取对象
    -   上下文信息：保存线程执行过程的全局信息
-   原理
    -   每个线程都有一个ThreadLocalMap，对于每个ThreadLocal对象，调用其get/set实际上就是以ThreadLocal对象为键读写当前线程的Map
    -   ThreadLocalMap的键类型为是WeakReference\<ThreadLocal>

### #20 并发总结

## c. 动态与函数式编程

### #21 反射

-   反射是Java的一种机制，在运行时动态获取类型的信息，比如接口信息、成员信息、方法信息、构造方法信息等
-   根据这些动态获取到的信息创建对象、访问/修改成员、调用方法等
-   反射的入口是Class类

##### 21.1 Class类

-   每个已加载的类在内存中都有一份类信息，每个对象都有指向它所属类信息的引用

-   Object类提供了一个获取对象/类的Class对象的方法：`getClass()` 

-   Class类的方法能够提供类的：

    -   名称信息：类名

    -   字段信息：类中定义的静态和实例变量

        -   `getDeclaredFields()`：返回本类声明的所有字段，包括非public的，但不包括父类的
        -   `Field.setAccessible()`：忽略Java的访问检查机制，以允许读写非public的字段

    -   方法信息：类中定义的静态和实例方法

    -   创建对象和构造方法

        -   `newInstance()`：调用类的默认构造方法（即无参public构造方法），如果无该构造方法，则抛出异常InstantiationException

    -   类型检查和转换

        -   `isInstance(Object obj)`与instanceOf等价

            ```java
            System.out.println(list instanceOf ArrayList);
            System.out.println(ArrayList.isInstance(list));
            ```

        -   `cast()`与强制类型转换等价

    -   Class的类型信息

    -   类的声明信息

    -   类的加载

    -   反射与数组

    -   反射与枚举

    -   反射与泛型

##### 21.2 应用实例

-   类的序列化及反序列化

    -   通过获取Class类方法对象的字段信息

    ```java
    // 演示
    public static String toString(Object obj) throws IOException{
        Class<?> cls = obj.getClass();
        StringBuilder sb = new StringBuilder();
        sb.append(cls.getName()+"");
        for(Field f: cls.getDeclaredFields()){
            if(!f.isAccessible()){
                f.setAccseeible(true);
            }
            sb.append(f.getName()+"="+f.get(obj).toString()+"\n");
        }
        return sb.toString();
    }
    ```

[TOC]

### #22 注解

##### 22.1 内置注解

-   `@Override`：表示该类重写了该方法。用于标记，告知编译器这是重写方法
-   `@Deprecated`：表示对应的代码已经过时了，程序员不应该使用它。表示一种警告
-   `@SuppressWarnings`：表示抑制Java的编译器警告，其参数表示要抑制的警告类型

##### 22.2 各种框架和库的注解

-   包括了Jackson、依赖注入容器DI、Servlet、Web应用框架等
-   声明式编程风格：诸如简单声明Serializable，java就能够自动处理很多复杂的操作，亦或是使用注解或synchronized关键字降低了编程的难度，这就是声明式编程风格

##### 22.3 创建注解

-   以`@Override`的定义为例

    -   使用`@interface`，以及两个元注解`@Target`，`@Retension`定义

        -   `@Target`表示注解目标，可以使用`{...}`表示有多个目标，若无声明，则默认为适用于所有类型
        -   `@Retention`表示注解信息保留到什么时候，取值只能有一个，包括了SOURCE只在源代码中保留、（默认）CLASS保留到字节码文件中、RUNTIME一直保留到运行时

        ```java
        @Target(Element.METHOD)
        @Retention(RetentionPolicy.SOURCE)
        public @interface Override{
        }
        ```

    -   可以为注解定义一些参数，甚至是提供非null的默认值

    -   更多元注解

        -   `@Documented`：表示将注解信息包含到生成的文档中
        -   `@Inherited`：让使用该注解的目标被继承后同样得到该注解

    -   注解不能继承

##### 22.4 查看注解信息

-   要使得注解能够影响程序，则需要查看注解信息

    -   主要考虑`@Retention`为RetentionPolicy.RUNTIME的注解

    -   利用反射机制在运行时查看和利用这些信息

        `Annotation[][] annts = method.getParameterAnnotations();`

##### 22.5 注解的应用：定制序列化

>   在序列化的过程中，定制输出格式

-   通过注解来修饰要序列化的类字段

    ```java
    static class Student{
    	@Label("姓名")
    	String name;
    }
    
    main(){
        new Student("张三");
    }
    
    /* output
    姓名：张三
    */
    ```

##### 22.6 注解的应用：DI容器

-   通过注解，可以自动获取该类型的实例，以及其所依赖的实例

[TOC]

### #23 动态代理

### #24 类加载机制

-   类加载器ClassLoader，用于加载其他类的类
    -   负责将字节码文件加载到内存中、创建Class对象
    -   ClassLoader由系统提供
    -   通过创建自定义的ClassLoader，可以实现
        -   热部署：在不重启Java程序的情况下，动态替换类的实现
        -   应用的模块化和相互隔离：不同的ClassLoader可以加载相同的类但互相隔离，互不影响
        -   从不同地方灵活加载：可以从网络中加载字节码文件

##### 24.1 类加载的基本机制和过程

-   三个类加载器
    -   启动类加载器Bootstrap ClassLoader，由C++实现
    -   扩展类加载器Extension ClassLoader
    -   应用程序类加载器/系统类加载器Application ClassLoader/System ClassLoader
-   双亲委派模型：优先让父ClassLoader去加载
    -   这三个ClassLoader可以认为是父子关系。不是父子继承关系，而是父子委派关系
    -   加载一个类的基本流程
        -   判断是否已经加载过了，加载过了则直接返回Class对象。一个类只会被一个ClassLoader加载一次
        -   如果没有被加载，则先让父ClassLoader去加载
        -   如果父ClassLoader没有加载成功，则自己尝试加载

##### 24.2 理解ClassLoader

-   获取ClassLoader
    -   每个Class对象都有一个方法，可以获取实际加载它的ClassLoader`getClassLoader()` 
    -   ClassLoader中
        -   可以获取它的父ClassLoader`getParent()` 
        -   可以获取默认的系统类加载器`getSystemClassLoader()`
        -   有一个加载类的主要方法`loadClass(String name)`
    -   如果ClassLoader是Bootstrap ClassLoader，则返回值为null，因为它不是由Java实现的，没有对应的类
-   ClassLoader的loadClass与Class的forName都可以加载类
    -   ClassLoader的loadClass不会执行初始类的初始化代码

##### 24.3 类加载的应用：可配置的策略

-   策略模式：通过更改配置，不用改变代码，就可以改变程序的行为

##### 24.4 自定义ClassLoader

-   自定义方式：继承ClassLoader，重写findClass

-   实现隔离：

    ```java
    MyClassLoader cl1 = new MyClassLoader();
    MyClassLoader cl2 = new MyClassLoader();
    String className = "com.example.test";
    Class<?> class1 = cl1.loadClass(className);
    Class<?> class2 = cl2.loadClass(className);
    if(class1 != class2){
    	System.out.println("different classes")
    }
    ```

##### 24.5 自定义ClassLoader的应用：热部署

-   实现动态更新
    -   使用同一个ClassLoader，类只会加载一次，加载后，即使class文件已经变了，在此加载，得到的也还是原来的Class对象
    -   使用自定义的ClassLoader加载发生了变化的Class能够得到新的Class对象，从而实现动态更新
-   案例：服务器端的java程序不通过重启就能更新服务

[TOC]

### #25 正则表达式

### #26 函数式编程

