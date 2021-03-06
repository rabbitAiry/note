# Java

> Thinking in Java
>
> Head First Java
>
> Java编程的逻辑
>
> Effective Java
>
> 深入理解JVM
>
> 网络内容补充



[TOC]

### #1 控制程序流

-   注释文档Javadoc

    - Javadoc是用于提取注释（`/** ... */`）的工具，输出一份HTML文件

    - Javadoc只为public和protected成员进行文档注释

    - 能够在注释中嵌入html

    - 可以添加一些标签，如`@see`链接到其他文档、`@version`版本信息、`@param`参数

##### 操作符

- 几乎所有操作符都只能操作基本类型，但`=`、`==`、`!=`能够操作所有对象，表示所引用的对象是否相同。此外，String类还支持`+`、`+=`

- 正则表达式  初识

  - 使用`?`表示可能有也可能没有的东西
  - 使用`+`表示有一个或多个要表达的东西

  ```java
  -?\\d+
  // \d表示一个数位,前面加上\表示转义
  // 表示前面可能有或没有负号，后面有一个或多个数位
  ```

- 位的运算：与或非、移位

- 类型转换：扩展转换会自动完成，窄化转换则需要强制

  ```java
  float f1 = 0.13;
  // 上述表达会报错，应该改为下述表达
  
  float f1 = 0.13F;
  float f2 = (float)0.13;
  ```

- 数值表达

  ```java
  1.39e-47f
  // 这表示 1.39 * 10^(-47)
  ```

##### 流程控制

- goto语句：continue和break后面加上标签，类似汇编语言的跳转

  ```java
  label1: 
  outer-iteration {
    inner-iteration {
      //... 
      break; // 1
      //... 
      continue; // 2 
      //...
      continue label1; 
      // 3 中断所有循环，回到label1，准备重新开始循环
      //...
      break label1; 
      // 4 中断所有循环，回到label1，不重新进入循环
    }
  }
  ```

##### 方法

-   java中传递引用数据类型方式是值传递，无论是基本数据类型还是对象（而不是引用传递）

    -   值传递
        -   指在调用函数时将实际函数复制一份到函数中
        -   如果在函数中对参数进行修改，不会影响到实际参数
        -   对象的传递，复制的是其引用（地址值），并不是引用指向的存在于堆内存中的实际对象
        -   可以改变对象的成员，但无法改变对象（如将对象置null，或更换对象、值）
    -   引用传递：将实际参数的地址直接传递到函数中

    ```java
    class MyString{
        String s;
    }
    
    public static void main(String[] args) throws InterruptedException {
        Temp temp = new Temp();
        temp.s.s = "good";
        temp.setNull1(temp.s);
        System.out.println(temp.s.s);
        temp.setNull2(temp.s);
        System.out.println(temp.s.s);
    }
    
    void setNull1(MyString val){
        val = null;
    }
    
    void setNull2(MyString val){
        val.s = null;
    }
    
    /* output:
    good
    
    */
    ```

    



[TOC]

### #2 对象

##### 运行时数据区

- 方法区Method Area/ HotSpot中的永久代，老年代？
    - 被所有线程共享
    - 存放被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
    - 如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常
    - 运行时常量池Runtime Constant Pool

- 堆Heap/ 新生代？
    - JVM所管理的内存中最大的一块，被所有线程共享
    - 生命周期：在虚拟机启动时创建
    - 为所有的实例和数组分配内存
    - 如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常

- 本地方法栈Native Method Stack
    - 与虚拟机栈发挥作用相似，但是为Native方法服务。甚至在有的虚拟机中与虚拟机栈合并了
- 虚拟机栈VM Stack
    - 线程私有
    - 描述的是Java方法执行的内存模型
        - 每个方法被执行的时候都会同时创建一个栈帧
        - 栈帧用于存储局部变量表（栈内存）、操作栈、动态链接、方法出口等信息
        - 每个方法执行完成后，栈帧从虚拟机栈中出栈

    - 该区域的两种异常状况
        - 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常
        - 如果虚拟机栈可以动态扩展，当扩张时无法申请到足够的内存时会抛出OutOfMemoryError

- 程序计数器Program Counter Register
    - 线程私有
    - 当前线程所执行的字节码的行号指示器
    - PCR是为了线程切换后能够恢复到正确的执行位置而设的，各条线程之间的PCR互不影响
    - 只能为Java方法计数，不能为native方法计数
- 直接内存
    - NIO（new Input/Output）类可以使用Native函数库直接分配堆内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作


##### 快来吧，溢出君

-   Java堆溢出

    -   发生异常时分析思路
        -   通过内存映像分析工具堆dump出来的堆存储快照进行内存分析
        -   确认内存中对象是否是必须的
        -   确认是内存泄漏还是内存溢出

    ```java
    static class OOMObject{}
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while(true){
            list.add(new OOMObject());
        }
    }
    ```

-   虚拟机栈和本地方法栈溢出（StackOverflowError）

    ```java
    private int stackLength = 1;
    public void stackLeak(){
        stackLength++;
        stackLeak();
    }
    
    public static void main(String[] args) {
        TestJVM test = new TestJVM();
        try{
            test.stackLeak();
        }catch(Throwable e){
            System.out.println("stack length:"+test.stackLength);
        }
    }
    ```

-   运行时常量池溢出

    ```java
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        int i = 0;
        while(true){
            list.add(String.valueOf(i++).intern());
        }
    }
    ```

-   方法区溢出

-   本机直接内存溢出

##### 引用

-   引用和对象相当于遥控器和电视机，使用引用来操控对象

    - 只需遥控器，就可以改变电视机音量大小
    - 遥控器可以单独存在

    ```java
    String s;
    String s = "abcd";
    ```

-   对象访问与内存

    ```java
    // e.g
    Object obj = new Object()
    ```

    -   `Object obj`的语义会反映到栈内存中，作为一个reference类型数据出现
    -   `new Object()`的语义会反映到堆内存中，形成一个存储Object类型的所有实例数据值的结构化内存
    -   此对象的类型数据（如对象类型、父类）的地址信息则存储在方法区中

-   四种引用Reference（JDK1.2+）

    -   强引用Strong
        -   GC不会回收（如果引用还在的话）

    -   软引用Soft
        -   系统将要发生OOM异常之前，会先回收这些对象

    -   弱引用Weak
        -   GC工作时就会回收掉，无论当前内存是否足够

    -   虚引用Phantom
        -   无法通过虚引用来取得一个对象实例

##### 对象创建过程|@1、2、3、4

- 对象创建过程包括

  - 分配内存
  - 对所有实例变量赋予默认值
  - 执行实例初始化代码

- 在Java中，**初始化**和创建无法分离

  - java会自动初始化，且在使用构造器之前（大部分初始值都为0）

  - 初始化的内容包括对象的初始化

      - 静态变量会在该类的任何对象创建之前就完成初始化
      - 静态变量会在该类的任何静态方法执行之前被初始化
      - 静态子句是多个静态初始化动作的集合
      - 如果用不到包含静态变量的类，则不会被初始化
      - 成员变量会在调用构造器前初始化

      ```java
      class Cups { 
          static Cup c1; 
          static Cup c2; 
      
          // 静态子句
          static { 
              c1 = new Cup(1); 
              c2 = new Cup(2);
          }
          Cups() { 
              System.out.println("Cups()");
          }
      }
      ```

  - 初始化顺序：①调用当前类的父类构造器 ②当前类的成员变量初始化过程 ③当前类的构造器

- 仅类中没有构造器时，编译器会自动创建一个无参构造器

- this表示该对象的引用，只能在方法内部使用

  - 所以静态方法不能用this

- 构造器中使用构造器：

  - 方法名用this代替，而不是构造器名
  - 只能调用一次，且调用必须在最起始处

  ```java
  Flower() {
    this("hi", 47); 
    // 调用其他构造器
    
    System.out.println("default constructor");
  }
  ```
  
- @1：用静态工厂方法代替构造器

  - 静态工厂方法的名称可以将对象特点描述得更清晰
  - 可以实现单例模式，或复用对象
    - 这样的对象是受控的
    - 这样的对象也能够轻易使用==来进行比较

  - 可以抽象地返回其子类型的对象

- @2：遇到多个构造器参数时要考虑用构建器builder模式

  - 可以使用setter替代，但不能保证线程安全
  - 使用builder可读性佳，可把调用串起来
    - 可以看作是抽象工厂


  ```java
  public class Flower{
      private final int size;
      private final int color;
      private final boolean eatable;
      
  	public static class Builder{
          // 必选参数
          private final int size;
          private final int color;
          // 可选参数
          private boolean eatable = false;
          
          public Builder(int size, int color){
              this.size = size;
              this.color = color;
          }
          public Builder eatable(boolean eatable){
              this.eatable = eatable;
              return this;
          }
          public Flower build(){
              return new Flower(this);
          }
      }
      
      private Flower(Builder builder){
          this.size = size;
          this.color = color;
          this.eatable = eatable;
      }
  }
  
  // User part
  Flower flower = new Flower.Builder(24, RED)
      .eatable(false).build();
  ```

- @3：单例模式的多种形态

  - 单例作为public成员、使用静态工厂方法获取单例、使用枚举类声明
- @4：工具类的应该设置private构造器

##### 垃圾收集|@6、7

- java对象不具备和基本类型一样的生命周期。当其引用作用域结束时，其指向对象仍继续占据内存空间

- 垃圾收集器(GC)

  - 如果一个对象失去了对其的引用，则有可能被回收。主要的数据变量（int）也类似
  - GC会在程序内存不足的情况下回收
  - 释放对象的引用③

  ```java
  // 方法结束时释放
  void go(){
  		Life life = new Life();
  }
  
  // 引用被赋值到其他对象
  Life life = new Life();
  life = new Life();
  
  // 直接将引用设定为null
  Life life = new Life();
  life = null;
  ```

- GC标记对象无用的算法

  - 引用计数法
    - 给对象添加一个引用计数器，每当有一个地方引用它时，计数器值加1；引用失效时减1
    - 计数器为0时，对象不可能再被使用
    - 缺点：难以解决相互引用的对象的回收

  - 根搜索法    *Java使用该GC算法
    - 以“GC Roots”对象作为起点，当一个对象无法到达GC Roots时视其为不可用
    - 可作为GC Roots的对象：虚拟机栈中引用的对象、方法区中的类静态属性，常量引用的对象、JNI引用的对象


  ```java
  // 根搜索算法的优势
  class TestJVM{
      public Object instance = null;
      private static final int _1MB = 1024*1024;
      
      // 用于占内存
      private byte[] bigSize = new byte[2*_1MB];
      public static void main(String[] args) {
          TestJVM objA = new TestJVM();
          TestJVM objB = new TestJVM();
          objA.instance = objB;
          objB.instance = objA;
          objA = null;
          objB = null;
          System.gc();
      }
  }
  ```

##### GC算法

- 标记-清除（Mark-Sweep）算法
    - 要清除一个对象，至少需要两次标记过程
    - 第一次标记和筛选
        - 标记方式是根搜索
        - 筛选依据是该对象是否有必要执行`finalize()`方法
        - 没有覆盖`finalize()`或其`finalize()`是否已被虚拟机调用，则没有必要执行
        - 被标记和筛选中的对象会加入F-Queue中，而标记但未被筛选中的对象会被直接回收
    - 第二次标记：发生在F-Queue内
        - 对象由虚拟机自动建立的Finalizer线程去执行`finalize()`
        - 这里的执行仅指触发该方法而不等待其结束。这样做的原因是如果某个对象的`finalize()`执行缓慢甚至发生死循环时，不会使得队列中其他永久处于等待状态
        - `finalize()`方法是对象逃脱死亡命运的最后一次机会。对象可以在该方法中尝试与任一对象进行关联
    - 缺点
        - 标记和清除的效率都不高
        - 标记清除之后会产生大量不连续的内存碎片
        - 空间碎片太多会导致在之后的运行过程中，如若遇到需分配较大对象时无法找到足够的连续内存而不得不提前触发另一次的垃圾收集工作
- 复制算法：更适合新生代
    - 将可用内存按容量大小分为相等的两块，每次只使用其中一块，当内存用完时，将活着的对象复制并整理到另一块内存空间上
    - 优化
        - 在新生代中，每次GC后都有大批对象死去，只有少量存活
        - 不按照1:1的比例来划分空间，而是划分为一块较大的Eden空间和两块较小的Survivor空间。每次使用Eden和其中一块Survivor
        - [ ] Eden有什么用？
        - HotSpot中，Eden和Survivor之比为8:1 
        - 当Survivor空间不足时，可以使用老年代内存
    - 评价
        - 实现简单，也解决了碎片问题
        - 代价是（未优化前）将内存缩小一半
        - 复制操作可能较多，效率低
        - 可以有效回收新生代
- 标记-整理算法：更适合老年代
    - 标记后，将所有存活对象整理好，然后直接清理剩余所有内存
- 分代收集
    - 新生代使用复制算法，因为清理后对象少，复制的工作量不大
    - 老年代因为对象存活率高、没有额外空间对其进行分配担保，故使用标记-清理或标记-整理算法
- GC在方法区的回收
    - 在方法区中回收性价比较低
    - 回收废弃常量：没有被任何对象引用的常量会被回收
    - 回收无用的类判断条件有3个
        - 该类的所有实例已被回收
        - 该类的ClassLoader已被回收
        - 该类对应的Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法
    - 在大量使用反射、动态代理、CGLib等bytecode框架的场景，以及动态生成JSP和OSGi这类频繁自定义ClassLoader的场景都需要虚拟机具备类的卸载功能，以保证永久代不会溢出

- @6：清除过期的对象引用
    -   需要警惕内存泄漏的情形
        -   类自己管理的内存（如，类的成员是数组，持有多个引用）。一旦元素被释放掉，则元素中包含的任意对象也应该被清空掉
        -   缓存。使用弱引用（weak）来解决
        -   监听器和回调
- @7：避免使用终结方法finalize
    - 交给GC就好

##### HotSpot的垃圾收集器

- Serial收集器
    - 单线程收集器，在进行垃圾收集时必须暂停其他所有的工作线程直到收集结束（Stop The World）
    - Serial是新生代收集器，采用复制算法
    - Serial Old是老年代收集器，采用标记-整理算法
    - 评价
        - 因为垃圾收集过程是虚拟机在后台自动发起和自动完成的，所以会在用户毫不知情的情况下暂停工作线程会带来恶劣体验
        - 简单而高效，毕竟没有线程交互开销
- ParNew收集器
    - Serial收集器的多线程版本。能与CMS收集器并行工作
    - 新生代收集器，采用复制算法
- Parallel Scavenge收集器
    - 新生代收集器，采用复制算法，并行的多线程收集器
    - 目的是达到一个可控制的吞吐量Throughput。吞吐量是仅运行用户代码和包括GC与运行用户代码的时间之比
    - 与CMS收集器不兼容
    - 最大垃圾收集停顿时间参数MaxGCPauseMillis
        - 该参数降低后，系统会把新生代调小，使得垃圾收集发生更频繁
        - 最终结果是吞吐量也下降了
    - 吞吐量大小参数GCTimeRatio
        - 该值x范围在（0-100）内，默认值为99
        - 垃圾收集时间占比为$\frac{1}{1+x}$ 
    - GC的自适应调节策略（GC Ergonomics）
        - 将参数UseAdaptiveSizePolicy打开后，虚拟机会动态调整停顿时间和吞吐量
- Parallel Old收集器
    - 老年代收集器，使用多线程和标记-整理算法

- CMS收集器（Concurrent Mark Sweep）
    - 以获取最短回收停顿时间为目标的收集器
    - 老年代收集器，使用标记清除算法，整个过程分为4步
        - 初始标记（需Stop The World）：仅标记GC Roots能直接关联的对象，速度很快
        - 并发标记：GC Roots Tracing
        - 重新标记（需Stop The World）
        - 并发清除

    - 缺点
        - 对CPU资源非常敏感
        - 无法处理浮动垃圾（并发过程中产生的垃圾）
        - 如果CMS运行期间预留的内存无法满足程序需要，就会出现Concurrent Mode Failure失败。这是虚拟机将启动后备预案，使用Serial Old收集器来重新进行老年代垃圾收集
        - 为了提高速度，使用的是标记-清除算法而不是标记-整理算法

- 并发与并行
    - 并行：多条GC线程并行工作，但此时用户线程仍处于等待状态
    - 并发：用户线程与GC线程同时执行
- G1收集器
    - 使用标记-整理算法，可以实现精确地控制停顿

##### 内存分配与回收策略

-   



[TOC]

### #3 类

##### static关键字

- 声明一个事物为static时，意味着这个数据或方法可以不创建对象直接访问或使用
- 即使创建同一类的两个对象，其中static成员变量的存储空间一样（只有一份存储空间）
- static方法中不能直接访问非static成员或方法

- java没有东西是全局的，静态类方法最接近全局
- java虚拟机会加载某个类是因为第一次有人要创建该类的新实例，或是使用该类的静态方法或变量。静态变量是在类被加载时初始化的
- 静态的import：`import static java.lang.Math.*`

##### 类加载过程

-   类的加载是指将类的相关信息加载到内存。在java中，类是动态加载的
-   类加载过程
    -   分配内存保存类的信息
    -   给类变量赋默认值
    -   加载父类
    -   设置父子关系
    -   执行类初始化代码

##### Object类|@5、8、9、10、11

-   @8：覆盖equals时的通用约定
    -   自反性reflexivity：自己必须等于自己
    -   对称性symmetry：a=b时，也必须做到b=a。这很容易在多态的比较中出现不符合的情况
    -   传递性transitive：如果a=b，b=c，则a=c
    -   一致性consistency：不会因为多次调用equals方法，其结果会发生改变
    -   非空性Non-nullity：对比双方都不应为空
-   @9：覆盖equals时也要覆盖hashCode

##### 包

- 创建CLASSPATH，存放`.class`文件
- 未给定访问权限：仅包内的该类对象可访问（包访问权限、friendly）
- public：皆可访问
- private：确保访问不了（比如阻止别人直接访问某个特定的构造器）
- protected：提供包访问权限，以及为有继承关系的类提供权限
- 类不可以是private或者protected的

##### 包装类与基础数据类型

-   数据类型

    - boolean、char、byte、short、int、long、float、double、void
    - [ ] 这些变量不大，但存储在堆中不如存储在栈中高效，所以java提供了不用new来创建变量的方法，设计了并非是“引用”的自动”变量“
    - 基本类型具有包装器类，如int类型和Integer类。使用包装器类使得可以在堆中创建一个非基本对象来表示基本类型
    - 6种数据类型包装类都有一个共同的父类Number
    - 高精度数字：BigInteger和BigDecimal

-   装箱、拆箱
    -   将基本类型转换为包装类的过程，一般称为装箱，反之称为拆箱
    -   Java5+引入了自动装拆箱，可以直接相互赋值
    -   自动装拆箱是编译器提供的能力，最后还是会替换为对应的valueOf和intValue方法
    -   使用`valueOf()`而不是new以创建包装类：除部分包装类外，都会缓存包装类对象，以减少需要创建对象的次数，节省空间

-   包装类特性
    -   重写了Object类的方法
        -   `equals(Object obj)`：皆反映了对象间逻辑相等关系
        -   `hashCode()`：逻辑相等的对象的哈希值必须一样。如果equals方法返回true，则hashCode必须一样
        -   `toString() `
    -   实现了Comparable
    -   与String相关：相互转换
    -   提供了常用变量

-   不可变性：包装类的实例对象一旦创建，就没有办法修改了
    -   不可变性的实现
        -   所有包装类都声明为final，不能被继承
        -   内部基本类型值都是private final的
        -   没有定义setter方法
    -   不可变是为了使得程序更为简单安全，不会被意外更改，可以安全地共享数据

-   @5：避免创建不必要的对象

    -   优先使用基本类型，而不是包装类

        ```java
        // 耗时10s，原因是程序构造了2^31g
        Long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum+=i;
        }
        
        // 耗时1s
        long sum2 = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum2+=i;
        }
        ```


##### Integer的二进制算法

>   Long与Integer类似
>
>   位操作能够充分利用CPU

-   位翻转：int的二进制形式种进行左右互换
-   循环移位
-   缓存-享元模式

##### Character

-   Unicode基础
-   检查code point和char
-   code point与char互换
-   按code point处理char数组或序列
-   字符属性
-   字符转换



[TOC]

### #4 泛型与容器

##### 泛型

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



##### ArrayList

-   剖析ArrayList

    -   内部维护一个数组elementData
    -   添加元素
        -   每次添加数据前都会调用`ensureCapacityInternal(size+1)`以确保数组容量是够的
        -   `ensureCapacityInternal(int minCapacity)`会先判断数组是不是空的，如果是，则指定分配大小不小于DEFAULT_CAPACITY
        -   然后上述方法会调用`ensureExplcitCapacity(int minCapacity)`。方法内部，如果需要的长度大于当前数组的长度，则调用`grow()`方法
        -   `grow(int minCapacity)`会先计算oldCapacity的1.5倍是否能够容纳minCapacity，如果能，newCapacity就是它了；如果不能则newCapacity等于minCapacity
        -   然后使用`Arrays.copyOf()`创建新数组
    -   删除元素
        -   使用`System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`将index以后的元素移一位
        -   释放数组最后一位的引用，以便原对象被垃圾回收
    -   基本思维：封装复杂操作，提供简化接口

-   迭代器

    -   foreach语法的背后

        ```java
        // 使用foreach
        for(Integr a: list){
            System.out.println(a);
        }
        
        // 实际上，编译器会将foreach代码转换为类似如下代码
        Iterator<Integer> it = list.iterator();
        while(it.hasNext){
            System.out.println(it.next());
        }
        ```

    -   迭代器接口Iterator

        -   只要对象实现了Iterable接口，就可以使用foreach语法
        -   可以创建Iterator对象的类也不一定需要实现Iterable
        -   Iterator接口提供了三个方法：`hasNext()`、`next()`、`remove()` 

    -   ListIterator

        -   扩展（继承）了Iterator接口，增加了向前、向后遍历的方法

    -   迭代器的删除方法

        ```java
        // 这段代码会抛出ConcurrentModificationException
        for(Integer a: list){
        	if(a<=100)list.remove(a)
        }
        ```

        -   因为迭代器的内部会维护一些索引位置相关的数据，要求在迭代过程中，容器不能发生结构性变化，否则这些索引位置就失效了
        -   结构性变化是指令数据结构长度发生变化的操作
        -   通过使用迭代器的`remove()`来避免这个问题

        ```java
        // 使用迭代器的remove()
        Iterator<Integer> it = list.iterator();
        while(it.hasNext()){
            if(it.next()<=100){
                it.remove();
            }
        }
        ```

-   ArrayList中迭代器的实现原理

    -   成员内部类Itr实现了Iterator接口
    -   Itr的成员变量`cursor`：表示下一个要返回的元素位置
    -   Itr的成员变量`lastRet`：表示最后一个（上一次）返回的索引位置
    -   Itr的成员变量`expectedModCount` 
        -   表示期望的修改次数，初始化为外部类的当前修改次数modCount
        -   每次迭代器操作的时候都会检查expectedModCount是否与modCount相同，以检测是否发生了结构性变化
    -   `hasNext()`：比较当前下一个索引是否等于size
    -   `next()`
        -   先调用`checkForModification()`以检查是否发生了结构性变化
        -   然后再更新成员变量的值以保持其语义
        -   返回对应的元素
    -   `remove()` 
        -   删除索引为lastRet的值。因此再调用`remove()`前，应该先调用`next()` 
        -   因为同时更新了expectedModCount的值，所以可以正确删除

-   迭代器设计模式

    -   将数据的实际组织方式与数据的迭代遍历相分离，是一种关注点分离的思想
    -   需要访问容器元素的代码只需要一个Iterator接口的引用，而不需要关注数据的实际组织形式，可以使用一致和统一的方式进行访问
    -   提供了简单而一致的接口

-   ArrayList实现的接口

    -   Collection：表示一个数据集合，数据间没有位置的概念
    -   List：表示有序的数据集合，扩展了Collection
    -   RandomAccess
        -   这是一个没有代码的接口，在Java中被称为标记接口。
        -   RandomAccess表示可以随机访问（指根据索引值直接可以定位到具体元素）
        -   在一些通用的算法代码中，可以根据这个声明而选择效率更高的实现。比如二分查找会根据list是否实现了这个接口而采用不同的实现机制

```java
int binarySearch(List<? extends Comparable<? super T>> list, T key){
    if(list instanceOf RandomAccess || list.size()< BINARYSEARCH_THRESHOLD){
        return Collections.indexedBinarySearch(list,key);
    }else{
        return Collections.iteratorBinarySearch(list,key);
    }
}
```

-   ArrayList的其他方法及相关方法

    -   两个构造方法

        -   指定初始化的数组大小
        -   以一个已有的Collection构建，数据会复制一份

    -   两个返回数组的方法

        -   `Object[] toArray()`返回Object数组
        -   `<T> T[] toArray(T[] a)`：返回对应类型的数组。如果参数数组长度足以容纳所有元素，就使用该数组，否则新建一个

        ```java
        Integer[] arr1 = new Integer[list.size()];
        list.toArray(arr1);
        Integer[] arr2 = list.toArray(new Integer[0]);
        System.out.println(Arrays.equal(arr1, arr2));
        // true
        ```

    -   获得list

        ```java
        new ArrayList<>(Arrays.asList(a));
        ```

    -   控制数组容量

        -   `ensureCapacity(int minCapacity)`：确保数组元素有这么大
        -   `trimToSize()`：重新分配一个数组，大小刚好为实际内容的长度。可以节省数组占用的空间



##### LinkedList

-   LinkedList实现的接口
    -   List、Collection、Iterable
    -   Queue
        -   主要操作有：添加（add、offer），查看（element、peek），删除（remove、poll）。每种操作都有两种形式
        -   在队伍为空时，element和remove会抛出NoSuchElementException异常，而peek和poll返回null
        -   在队列为满时（显然不包括LinkedList），add会抛出IllegalStateException，而offer只是返回false
-   LinkedList实现原理
    -   内部实现是一个双向链表
    -   内部有成员变量：size、first、last、modCount
    -   add方法
        -   直接为last节点后新增一个节点
        -   要注意这是个双向链表，因此添加节点需要更改周围的节点的指针
        -   最后要维护内部三个成员变量的语义

    -   get方法
        -   先判断index是否有效
        -   然后遍历找到对应元素。如果index大于长度/2，就从尾部往前搜，否则从头部开始搜（似乎在这节提到的方法中，只有get方法有优化搜索方向）

    -   indexOf方法：遍历搜索
    -   `add(int index, E element)`在头部或之间插入元素
    -   删除元素
        -   不需要专门让被删除节点变为null，毕竟已经没有node指向它了


-   LinkedList特点
    -   按需分配空间
    -   不可以随机访问

[TOC]

##### HashMap

-   实现的接口

    -   Set接口：表示集合的概念，扩展了Collection但是没有定义新的方法
    -   Map接口

-   HashMap的`keySet()`, `values()`, `entrySet()`方法有一个共同的特点，它们返回的都是视图，而不是复制的值，基于返回值会直接修改Map自身

-   HashMap实现原理

    -   内部组成
        -   `Entry<K, V>[] table `
            -   Entry类型的数组，称为哈希表或哈希桶
            -   其中每个元素指向一个单链表，链表中的每个节点表示一个键值对
            -   Entry是内部类，是链表中的节点，内部有next变量指向下一个Entry节点
            -   这个数组会随着键值对的添加进行扩展，扩展的策略类似于ArrayList
        -   loadFactor：负载因子，表示整体上table被占用的程度。当table占用程度达到该比例时，table考虑进行扩展
        -   threshold：阈值，值为table.length*loadFactor
        -   table.length：总是为2的幂，初始值为16
    -   put方法
        -   如果表为空，则创建
        -   如果找到对应的hash值，且能找到元素也相等的值并添加
        -   否则，先判断空间够不够，size有没有超过阈值，然后在对应hash位置的头部添加节点

-   在Java8+，在哈希冲突比较严重的情况下（大量元素映射到同一个链表），则将该链表转换为一个平衡的排序二叉树

-   对哈希表进行排序：利用EntrySet

    -   由于HashMap不保持顺序，所以没办法直接重映射，或者要使用LinkedHashMap

    ```java
    List<Map.Entry<Integer, Student>> list = new ArrayList<>(map.entrySet());
    Collections.sort(list, new Comparator<Map.Entry<Integer, Student>>() {
    
        @Override
        public int compare(Entry<Integer, Student> o1, Entry<Integer, Student> o2) {
            return o1.getValue().compareTo(o2.getValue());
        }
    
    });
    for (Entry<Integer,Student> entry : list) {
        System.out.print(entry.getKey()+":"+entry.getValue());
    }
    ```

    



##### HashSet

-   实现原理
    -   内部使用HashMap实现，其所有的值皆为常量
    -   构造方法：构造一个新的HashMap
    -   add方法、contains方法、remove方法：调用map的方法
    -   iterator方法：返回map的keySet迭代器













[TOC]

### #5 复用类

- 多个类含有main方法：编译器只会执行当前调用类所属main方法。在每个类中都设置main()使得类的单元测试变得简易，且可以保留

##### 组合

- 组合：新的类由现有类的对象所组成
- toString()方法：每一个非基本类型的对象都有一个toString方法，当编译器需要时会自动调用（比如：直接将对象赋给String类型或者输出）

##### 继承

- 所有类会隐式地从object类进行继承

- 优先从当前类中寻找被指定的方法，可以通过super关键字来调用父类方法

    ```java
    // test5_1
    class Base{
        public Base(){
            test();
        }
        public void test(){}
    }
    
    class Child extends Base{
        private int a = 123;
        public Child(){}
        public void test(){
            System.out.println(a);
        }
        
        public static void main(String[] args){
            Child c = new Child();
            c.test();
        }
    }
    
    /* output:
    0
    123
    
    */
    ```

- java会在子类的构造器中插入对父类构造器的调用。所以调用子类构造器时，会先调用父类构造器，再执行子类构造器的操作。可以使用super指定调用哪一个构造器

- "is-a(是一个)"的关系是用继承来表达的，可以认为子类是对父类的取代；"has-a"的关系使用组合来表达；"is-like-a"的关系则使用接口来表示，这意味着即时向上转型，也能调用子类额外新增的方法

- 向上转型：子类是父类的一种类型

  ```java
  // 正常运行
  Base b = new Child();
  Child c = (Child)b;
  
  // 运行时会抛出异常
  Base b = new Base();
  Child c = (Child)b;
  ```
  
- 变量与继承

  - private变量与方法只能在类内访问。如：子类不能访问分类的private变量
  - 对于public变量，子类能够通过super明确指定访问父类的变量
  - 静态绑定
    - 静态绑定：当通过b（静态类型Base）访问时，访问的是Base的变量和方法，当通过c（静态类型Child）访问时，访问的是Child的变量和方法
    - 静态绑定在程序编译阶段即可决定，而动态绑定发生在程序运行时

  ```java
  // test5_2
  class Base{
      public static String s = "static_base";
      public String m = "base";
      public static void staticTest(){
          System.out.println("base static:"+s);
      }
  }
  
  class Child extends Base{
      public static String s = "child_base";
      public String m = "child";
      public static void staticTest(){
          System.out.println("child static:"+s);
      }
  }
  
  public static void main(String[] args){
      Child c = new Child();
      Base b = c;
      System.out.println(b.s);
      System.out.println(b.m);
      b.staticTest();
      System.out.println(c.s);
      System.out.println(c.m);
      c.staticTest(); 
  }
  
  /* output:
  static_base
  base
  base static:static_base
  child_base
  child
  child static:child_base
  
  */
  ```

  

##### 关键字final

- 对于原始类型，final使数值恒定不变；对于对象引用，一旦引用被初始化指向一个对象，就无法更改（对象自身可以修改）

  ```java
  final int num1 = 10;
  final Value num2 = new Value(10);
  
  // num1++;   //error
  num2.i++;
  ```

- 带有初始值的final static原始类型全用大写字母来命名，并且单词与单词之间用下划线来隔开

- 空白final：声明为final但又未给定初值的数据成员

  - 空白final应该在构造器中得到初始化

- final参数：值或者引用所指向的对象 不能被更改

  - 必须在定义处声明或是初始子句中声明

- final方法

  - 使用的原因：①预防了任何继承类重写（包括重载）它 ②效率
  - final类中标记final方法是多余的

- final和private的关系

  - 类中所有private方法都隐含是final的
  - 如果某方法为private，那么该方法和继承无关，子类使用该方法名*不能* 称之为重载

  ```java
  class OP{
  	private final void f(){}
  	private void g(){}
  }
  
  class OP2 extends OP{
  	public final void f(){}
  	public void g(){}
  	public static void main(String[] args){
  		OP2 op2 = new OP2();
  		op2.f();
  		op2.g();
  		OP op = op2;
  		// op.f();  //可以上转型，但不能调用方法
      // op.g();
  	}
  }
  ```

- final类：该类不能被继承




[TOC]

### #6 多态、抽象

##### 向上转型

> 简述见#5：继承

- 绑定binding：传入子类对象上转型为父类对象时，通过binding得知绑定哪一个子对象

  - 后期绑定（动态绑定）会自动发生

    ```java
    class Shape{
    		void draw()
    		void erase()
    }
    class Circle extends Shape{
    		void draw()
    		void erase()
        public void main(){
          	Shape s = new Circle();
          	s.draw();
          	// 这里被调用的是子类的方法
        }
    }
    ```

- 抽象

  - 
  
- 构造器、清除与多态

  - 当清除类时，应该先从子类开始清除，然后再调用其父类的清除方法
  - 构造器中尽量减少调用方法

##### 向下转型

- 在java中，所有转型都会得到检查。如果类型不符合，则抛出ClassCastException（转型异常）。这种在运行期间对类型进行检查的行为称为RTTI（运行期类型识别）

- 向下转型案例

  ```java
  class Father{
  		public void f();
  }
  
  class Son extends Father{
  		public void f();
  		public void g();
  		public static void main(){
  				Father f1 = new Father();
  				Father f2 = new Son();
        
  				(Son)f1.g();  // Exception
        	f2.g();  			// Exception
  				(Son)f2.g();
  		}
  }
  ```

  

[TOC]

### #7 接口、内部类、枚举

##### 接口

- 可以说是纯粹的抽象类，接口所包含的数据成员隐含都是static和final的，所被定义的方法自动被定义为是public的
  - 接口被用来建立类与类之间的协议protocol
  - 一个类可以实现多个接口从而实现多重继承，其可以上转型为接口
  - 可以implement一个类
  - 抽象类和接口之间做选择：只有在强制必须具有方法定义和成员变量时，才应该选择抽象类
- 接口的继承
  - 使用extend，但子接口不需要实现方法
  - 接口的继承允许继承多个接口，只需用`,`隔开
- 接口的嵌套
  - [ ] 实现private接口的类不能被其他类所使用
  - [ ] 为什么接口可以在类中被调用

##### 内部类

- 内部类与组合是不同的概念
  - 与一般的类不同，内部类可以被声明为private或protected
  - 使用内部类的理由
      - 可以实现对外部的完全隐藏，有更好的封装性
      - 返回引用、创建了不是公用的工具类
  
- 4种内部类

  - 静态内部类：使用static修饰

      -   只可以访问外部类的静态变量和方法，不可以访问实例变量和方法
      -   在当前外部类中，可以直接调用静态内部类的方法；其他类中则只需要先创建一个内部类
      -   使用场景：与外部类关系密切，且不依赖于外部类实例
      -   例子：LinkedList中的私有静态内部类Node

      ```java
      // Outer$StaticInner
      Outer.StaticInner si = new Outer.StaticInner();
      ```

  - 成员内部类

      - 可以访问外部类所有变量
      - 成员内部类中不可以定义静态变量和方法（常量除外，即final变量）
      - 在当前外部类中，需要先创建内部类才能访问；在其他类在，则需要先创建一个外部类才能新建内部类
      - 使用场景：与外部类关系密切，且需要访问外部类的实例变量或方法
      - 例子：LinkedList中返回的Iterator接口

      ```java
      // Outer$Inner
      Outer outer = new Outer();
      Outer.Inner inner = outer.new Inner();
      ```

  - 方法内部类

      - 这个类在方法的作用域内，可以访问的类与方法的变量范围和当前方法一样
      - 访问方法中的参数时可以直接访问，但访问方法中的局部变量时，只能访问被声明了final的局部变量
      - 只能在定义的方法内被使用

  - 匿名类对象

    - 可访问的参数与方法内部类一致
    - 常作为某个类的匿名子类
    - 使用场景：return中
    - 例子：Arrays.sort中的Comparator

    ```java
    Arrays.sort(strs, new Comparator<String>(){
    	@Override
        public int compare(String o1, String o2){
            return o1.compareToIgnoreCase(o2);
        }
    })
    ```

- 内部类的实现

  - 在JVM中无内部类概念，每个内部类最后都会被编译为一个独立的类
  - 使用$表示内部类
  - 访问成员变量或方法的原理：使用外部类的对象来实现内部类的初始化，并为外部类被访问的变量或方法自动生成get方法
  - 对于方法内部类，则把方法中的参数也传递给了内部类，方法中的final变量则直接生成在内部类中

- 继承自内部类

  - 继承内部类只需要在extends中通过点表达式指明哪个类，但是初始化需要传入外围类

    ```java
    class WithInner { 
      class Inner {}
    }
    class InheritInner extends WithInner.Inner { 
        // 不能直接初始化
        // InheritInner(){}
        InheritInner(WithInner wi) {
        	wi.super();
        }
    }
    ```

- 继承自外部类

  - 子类的内部类与父类的内部类是两个独立的实体
  - 可以在子类中的内部类继承自内部类的方法让子类，从而实现重载

- 内部类的作用/应用场景

  - 当需要在一个类中实现两个同样的接口时（比如同时为两个按钮设置监听事件，意味着你可能需要在一个类中实现两次同样的方法，但这是不可能的），使用内部类能够很好的解决问题
  - 使得类能够像成员变量一样被传递
  - 是一个独立的类，但是需要对某些数据保持联系

- 闭包

  - 闭包是一个可调用的对象，包含了创建它的作用域（如：外围类）
  - 通过内部类提供闭包的功能比指针更灵活、安全

##### 枚举

-   使用枚举让代码更为简洁

    -   枚举变量的`toString()`或`name()`会返回其字面值
    -   枚举变量的`ordinal()`或枚举类型的`valueOf()`会返回枚举值在声明式的顺序
        -   枚举类型也实现了Java API中的Comparable接口，可以通过compareTo与其他枚举值进行比较，比较的是`ordinal()`的大小
        -   可以用于switch语句，switch语句内不需要前缀
    -   枚举变量可以使用equals或==进行比较，结果一致
    -   枚举类型的`values()`返回枚举值的数组

    ```java
    public enum Size{
    	SMALL, MEDIUM, LARGE
    }
    ```

    ```
    Size size = Size.MEDIUM
    ```

-   枚举类原理

    -   枚举类是final的，其父类式Enum\<T>
    -   枚举类的构造方法是私有的

-   拓展枚举的用法

    -   枚举值的定义要放在最上面，并于枚举值和代码之间使用分号隔开

    ```java
    public enum Size{
    	SMALL("S","小号"), 
        MEDIUM("M","中号"),
        LARGE("L","大号");
        
        private String abbr;
        private String title;
        private Size(String abbr, String title){
            this.abbr = abbr;
            this.title = title;
        }
        
        public String getAbbr(){
            return abbr;
        }
        
    	public String getTitle(){
            return title;
        }
    }
    ```

    



[TOC]

### #8 异常与错误处理

##### 异常

```
Throwable
- Error
- Exception
    - 受检查异常
    - RuntimeException（非受检异常unchecked exception）
```

-   Error
    -   出现时，程序无法恢复，只能重启（仍然可以使用try-catch语句）
    -   包括了OOMError、StackOverflowError等
-   RuntimeException：不需被检查的异常
    - 其子类包括ClassCastException和NullPointerException等
    - 这些异常大多发生在代码逻辑有误，所以不被编译器检测到（编译器不要求你对抛出这种异常的方法使用try catch）
    - 自定义异常：必须从已有的异常类继承（可以是Exception类）

##### 异常的抛出与捕获

- 异常的处理

  - Java首先创建一个异常对象，然后查看谁能处理这个异常
  - 异常不断查看上一层的函数是否能够处理异常
  - 如果没有代码能够处理异常，则启用默认处理机制，即：打印异常栈信息到屏幕上，并退出程序
  
- 异常的产生

  - 通过new创建一个异常对象，然后通过throw抛出错误（部分抛出异常方法中已包含了new，不需要我们去使用new）
  - throw由jvm实现。throw与return相比，throw是异常退出，throw后执行哪行代码是不确定的
  
- try-catch-finally

  - try区块放入的是可能导致错误的语句。同一异常不会被catch两次

  - finally内的代码总会被执行

      - 如果有异常但是需要抛给上层执行，则在抛出前执行finally内代码块
      - 如果在try或catch代码块内有return语句，则return语句会在finally语句执行结束后才执行。但finally不能够改变返回值
      - 如果finally语句中也有return语句，则try和catch语句中的return会丢失，实际会返回finally代码块中的返回值
      - finally语句中也有return语句，不仅会覆盖try和catch内的返回值，还会掩盖try和catch内的异常，就像异常没有发生过一样
      - finally中抛出异常，则原异常也会丢失
      - 应该尽量避免在finally中使用return或者抛出异常

  - 开发阶段遇到异常时获取异常信息，部署阶段则把异常写入日志文件中

  - 异常链

      - 可以在catch语句中重新（新建）抛出异常。再抛出的异常并不在当前try-catch语句中处理
      - 使用场景：当前代码不能够完全处理该异常，需要调用者进一步处理

      ```java
      try {
          System.out.println(3/0);
      }catch(ArithmeticException e){
          throw new Exception("啊这?",e);
      } catch (Exception e) {
          System.out.println("处理了");
          e.printStackTrace();
          throw e;
      }finally{
          System.out.println("数学真有趣");
      }
      
      /* output:
      数学真有趣
      Exception in thread "main" java.lang.Exception: 啊这?
              at Test.main(Test.java:51)
      Caused by: java.lang.ArithmeticException: / by zero
              at Test.main(Test.java:49)
              
      */
      ```

- 异常说明

  - 将异常告知用户

  - 在方法中使用了关键字throws，后面接一个潜在异常类型的列表（声明有可能抛出的异常）

  - 若当前异常自己不能处理，则应该向上报告

  - 子类方法抛出的异常必须不能够多于父类

      -   如：父类不抛出异常，子类也不能抛出

      ```java
      void makeCake(int size) throws TooBig,TooSmall{
        if(size>100){
          throw new TooBig();
        }
        // ...
      }
      ```

- 避开异常：不使用try-catch语句，在方法中再声明throw，把异常抛给上一个环境中的异常处理程序

- 重新抛出异常：在catch语句中使用throw，把异常抛给上一个环境中的异常处理程序

  - 如果所有方法都选择避开异常，则最后jvm挂了

- 异常链

  - 异常链：在捕获一个异常后抛出另一个异常，并将原始的异常信息记录下来

[TOC]

### #9 字符串、GUI、Swing

##### 字符串

- 输出数字的格式化

  `%[指定参数][flags][最小的字符数][.精确度][数据类型(必填)]`

  ```java
  String.format("I have %,.2f money",476578.09876);
  // I have 476,578.10 money
  ```

- 以日期格式输出（数据类型填入以下，传入的参数是Date类型）

  - %tc 完整的日期与时间
  - %tr 只有时间
  - %tA %tB %tC 周月日
  - %<tB 使用前面用过的参数

- 日期加减法

  ```java
  Calendar c = Calendar.getInstance();
  
  // 日期设置为2004年2月7日15:40
  c.set(2004,1,7,15,40)
  // 返回时间的总微秒数
  c.getTimeInMillis();
  // 加上35天
  c.add(c.DATE, 35);
  // 仅日期自增35
  c.roll(c.DATE, 35);
  // 输出时间
  System.out.println(c.getTime())
  ```
  
- 把字符串拆开：使用`.split(",")`把字符串分割到数组中





[TOC]

### #10 文件、序列化

##### 序列化

- 将对象序列化（存储）的方法

  1. 创建出FileOutputStream
  2. 创建ObjectOutputStream
  3. 写入对象
  4. 关闭ObjectOutputStream
  5. 输入输出相关操作都必须在try-catch块中执行

  ```java
  // 1. 如果文件不存在，会自动被创建出来
  FileOutputStream fs = new FileOutputStream("MyGame.ser");
  // 2
  ObjectOutputStream os = new ObjectOutputStream(fs);
  // 3
  os.writeObject(characterOne);
  os.writeObject(characterTwo);
  os.writeObject(characterThree);
  // 4
  os.close();
  ```

- stream用于连接

- 成员变量会被保存，当对象被序列化时，被该对象引用的实例也会被序列化。但是静态变量不会被保存（静态变量属于类）

- 如果要让类能够被序列化，就需要实现序列化

  `implements Serializable`

  - 类中的成员变量会被保存，因此被对象引用的实例与需要实现序列化
  - 如果某实例变量不能（或不应该）被序列化，就把它标记成transient的变量（瞬时的），这意味着序列化的过程中会跳过这个变量
  - 如果父类实现了序列化，则其子类会自动实现，即使没有声明
  - 可以通过继承的方式让方法得到序列化的可能
  - 如果两个对象都有引用实例变量指向相同的对象，则只有一个对象会被存储
  
- 序列化的版本控制

  - 假设1.0的版本将对象序列化了，但在2.0的版本中把double成员变量改为int类型的成员变量
  - 使用serialVersionUID记录类的版本。java会依据结构自动生成该id。还原时，Java会自动对比

##### 解序列化：还原对象

- 步骤

  1. 创建FileInputStream
  2. 创建ObjectInputStream
  3. 读取对象
  4. 转换对象类型
  5. 关闭ObjectInputStream

  ```java
  // 1
  FileInputStream fs = new FileInputStream("MyGame.ser");
  // 2
  ObjectInputStream os = new ObjectInputStream(fs);
  // 3 顺序与写入相同，超出次数则会抛出异常
  Object one = os.readObject();
  Object two = os.readObject();
  Object three = os.readObject();
  // 4
  GameCharacter elf = (GameCharacter) one;
  GameCharacter troll = (GameCharacter) two;
  GameCharacter magician = (GameCharacter) three;
  // 5
  os.close();
  ```

- 解序列过程中，新的对象被配置到堆上，但构造函数不会执行，而是把对象状态抹去又变成全新的

  - 如果对象在继承树上有个不可序列化的祖先类，则不可序列化类以及之上的类的构造函数会执行，即从第一个不可序列化的父类开始全部重新初始化状态
  - transient变量会被赋值null的对象引用或primitive主数据类型的默认为0、false等值
  - 静态变量不会被序列化，对象还原时也只会维持不变

##### 文本文件、文件处理

- 存储文本步骤

  ```java
  // 1
  FileWriter writer = new FileWriter("Foo.txt");
  // 2
  writer.write("hello foo!");
  // 3
  writer.close();
  ```

- File类：代表文件路径而不是文件本身

- 缓冲区

  ```java
  BufferedWriter writer = new BufferedWriter(new FileWriter(aFile));
  ```

  - 缓冲区就像购物车一样，让你一次能够放许多东西，效率高。缓冲区满时就一并写入文件
  - 强制缓冲区立即写入：调用`writer.flush()`

- 读取文件示例

  ```java
  try {
      File file = new File("MyCode.txt");
      FileReader fileReader = new FileReader(file);
      BufferedReader reader = new BufferedReader(fileReader);
  
      String line = null;
      while((line=reader.readLine())!=null){
          System.out.println(line);
      }
      reader.close();
  } catch (Exception e) {
      e.printStackTrace();
  }
  ```



[TOC]

### #11 网络与线程

##### 网络

- 建立Socket连接

  - socket是个代表两台机器之间网络连接的对象，socket连接的建立代表两台机器之间存有对方的信息，包括网络地址和TCP的端口号
  - TCP端口：用于识别服务器上特定程序的数字，共有2^16^个
    - 0-1023的TCP端口号是保留给已知的特定服务使用
    - HTTP 80；HTTPS 443；FTP 20
    - 不同程序不能共用端口
  - 使用BufferedReader从Socket上读取数据：Java的输入输出工作并不在乎stream的另一端是谁
    1. 建立对服务器的Socket连接
    2. 建立连接到Socket上低层输入串流的InputStreamReader
    3. 建立BufferedReader

  ```java
  // 1
  Socket chatSocket = new Socket("127.0.0.1",5000);
  // 2
  InputStreamReader stream = new InputStreamReader(chatSocket.getInputStream());
  // 3
  BufferedReader reader = new BufferedReader(stream);
  String message = reader.readLine();
  ```

  - 使用PrintWriter写数据到Socket上
    1. 对服务器建立Socket连接
    2. 建立连接到Socket的PrintWriter
    3. 写入数据

  ```java
  // 1
  Socket chatSocket = new Socket("127.0.0.1",5000);
  // 2 
  // PrintWriter是字符数据和字节间的转换桥梁，可以衔接String和Socket两端
  PrintWriter.writer = new PrintWriter(chatSocket.getOutputStream());
  // 3
  writer.println("message to send");
  writer.print("another message");
  ```

- 编写服务器端的程序

  1. 服务器应用程序对特定端口创建出ServerSocket
  2. **客户端**对服务器应用程序建立Socket连接
  3. 服务器创建出与客户端通信的新Socket
     - 当用户连上来时此方法会返回一个Socket以便与客户端通信。Socket与ServerSocket的端口不相同，因此ServerSocket可以空出来等待其他的用户

  ```java
  // 示例代码：每日鸡汤客户端
  // 这个程序会创建ServerSocket并等待客户端的请求。
  // 当它收到客户端请求时，服务器会建立与客户端的Socket连接
  // 服务器接着会建立PrintWriter来送出信息给客户端
  try {
    	// 1
      ServerSocket serverSocket = new ServerSocket(4242);
  
      // 服务器进入无穷循环等待服务端的请求
      while(true){
          // 3 这个方法会停下来等待连接
          Socket sock = serverSocket.accept();
  
          PrintWriter writer = new PrintWriter(sock.getOutputStream());
          String advice = getAdvice();
          writer.println(advice);
          writer.close();
          System.out.println(advice);
      }
  } catch (Exception e) {
      e.printStackTrace();
  }
  ```

##### 线程

- 创建线程

  ```java
  Thread t = new Thread();
  t.start();
  ```

  - 新的线程提供了独立的执行空间(stack)
  - 创建线程很简单，但是该线程并没有执行任何程序，因此它几乎是一创建就结束了。因此需要添加线程的任务

- 启动新的线程

  1. 建立Runnable对象（线程的任务）
  2. 建立Thread对象并赋值Runnable
  3. 启动Thread

  ```java
  // 1
  Runnable threadJob = new MyRunnable();
  // 2
  Thread myThread = new Thread(threadJob);
  // 3
  myThread.start();
  ```

  - 线程不能重新启用，完成任务后即结束
  - Runnable函数需要有run()方法，这也是线程中执行的任务

- 每个Java应用程序会启动一个主线程——将main()放在它自己执行空间的最开始处。Java虚拟机会负责主线程的启动以及比如垃圾收集所需的系统用线程

- 线程不是进程，不能够同时执行几件事，只是执行动作可以在stack中非常快速地来回切换

- 线程调度器（java虚拟机）

  - 线程调度器会决定哪个线程从等待状况中被挑出来运行，以及何时把哪个线程送回等待被执行的状态
  - 当线程被踢出时，线程调度器也会指定线程要回去等待下一个机会还是暂时地堵塞
  - 线程调度器不可控制，不能保证哪个线程会被优先执行
  - 可以调用sleep()方法让其他线程都有机会运行

- 线程带来了并发性(concurrency)的问题

  - 可能的状况：两个线程存取单一对象的数据（读取数据和使用数据两个步骤之间，其他线程对该数据进行了操作）
  - 为方法使用synchronized（同步化）关键词。要保护数据，就把作用在数据上的方法给同步化
  
- 同步化》原子性atomic：意思是不可分割的
  - 当对象有一个或多个同步化的方法时，线程只有在取得对象锁的钥匙时才能进入同步化的方法。锁是在对象上或类上（静态变量）的，而不是方法上（尽管声明在方法上）
  - 线程开始调用同步化的方法时，为对象上锁，直到方法完成。上锁期间没有其他线程可以进入该对象的同步化方法

- 同步化的代价

  - 可以只同步化部分行而不必全部同步化

  ```java
  public void go(){
  	doStuff();
  	synchronized(this){
  		doMoreStuff();
  	}
  }
  ```

  - 查询钥匙带来部分损耗
  - 可能导致死锁

- 死锁

  - 两个线程皆拥有钥匙，但皆在等待对方的钥匙，此时死锁发生了
  - java没有处理死锁的机制



### #12 集合与泛型

##### 集合

- 有如下集合

  - ArrayList：无限长数组
  - LinkedList：针对经常插入或删除中间元素所设计的高效率集合
  - TreeSet：以有序状态保持并可防止重复
  - HashMap：使用键值对保存数据
  - HashSet：防止重复的集合
  - LinkedHashMap：类似HashMap，但是可以记住元素插入的顺序

- 排序

  - 倘若是int类型或String类型，可以直接使用TreeSet或Collections.sort()方法
  - 倘若是一个类，则可以
    - 让该类实现`Comparable<T>`，并重写compareTo()方法
    - 或重写Comparator，并传入sort()方法中

  ```java
  // 1
  class Song implements Comparable<Song>{
    // ...
    public int compareTo(Song s){
        return songName.compareTo(s.getName());
    }
  }
  
  // 2
  class SongNameCompare implements Comparator<Song>{
    public int compare(Sone one, Song two){
      return one.getName().compareTo(two.getName());
    }
  }
  SongNameCompare snc = new SongNameCompare();
  Collections.sort(songList, snc);
  ```

- HashSet查重：会使用对象的hashcode作对比

  - 在类中重写两个方法：equals()方法、获取hashCode的hashCode()方法（返回文字的hashCode）
  - 不同的对象可能会有相同的hashCode

##### 泛型

- 泛型意味着集合中的类型得到了限制，意味着更好的类型安全性

- 多态参数与泛型

  - 数组与使用泛型的集合类不同

  ```java
  public void takeAnimals(Animal[] animals){
    	for(Animal a: animals){
        	a.eat();
      }
  }
  public void takeAnimals(ArrayList<Animal> animals){
      for(Animal a: animals){
        	a.eat();
      }
  }
  
  // 数组
  Dog[] dogs = {new Dog(), new Dog(), new Dog()};
  takeAnimals(dogs);
  
  // ArrayList
  ArrayList<Dog> dogs = new ArrayList<Dog>();
  // takeAnimals(dogs);			error
  ```

  - 报错的原因是因为存在Cat类对象也被加入集合的可能存在。数组没有报错只是因为数组的类型是在运行期间检查的，而集合的类型检查只会发生在编译期间（上述操作在数组版本中是不可以通过运行的）

- 使用万用字符wildcard 或 泛型类型

  - 创造出接受animal子型参数的办法

  ```java
  // 万用字符
  void takeAnimals(ArrayList<? extends Animal> animals){
      // ...
  }
  
  // 泛型类型
  <T extends Animal> void takeThing(ArrayList<T> list)
  ```

  - 在方法参数中使用万用字符时，编译器会阻止可能破坏引用参数所指集合的行为：你能够调用list中任何元素的方法，但不能加入元素（若代码中包含加入Cat类对象，则不能通过编译）



### #13 包、jar存档文件、部署

- 将源代码和类文件分离：确保编译过的类不会放在源代码目录中

  - 把源代码存储在source目录下
  - 编译时加上`-d`：编译时让`.class`文件产生在classes目录下

  ```bash
  javac -d ../classes com/example/*.java
  ```

- 把程序包放进JAR（需要先分离）

  - 把一组类文件包装起来，并创建出可执行的JAR
  - JAR（大写）：集合起来的文件；jar（小写）：用来整理文件的工具

- 可执行的JAR：让用户不必把文件抽出来即可运行

  - 秘诀：创建出manifest.txt文件，它会带有JAR的信息

  - 步骤

    1. 在classes目录下创建manifest.txt，指出哪个类带有main()方法
    2. 执行jar工具来创建带有所有类以及manifest的JAR文件

    ```bash
    # 1 txt里面》不需要加.class，此行后面需要换行
    Main-Class: MyApp
    
    
    # 2 bash里面
    jar -cvmf manifest.txt app1.jar *.class
    
    # 3 执行JAR
    java -jar app1.jar
    ```

- 防止包命名冲突：把类放进包中

- 如果用户没有安装Java，就不能执行JAR

- jws应用（Java Web Start）



### #14 远程部署、RMI

- RMI: Remote Method Invocation远程程序调用

- 调用不同机器上的对象方法

  - 使用Socket通信，使用辅助设施（helper）
    - 客户端调用helper的方法，就好像客户端就是服务器一样
    - 客户端helper假装成服务，实际上会连接服务器，并将调用的方法和参数传给服务器，然后等待响应。客户端helper是代理
    - 服务器helper会通过socket连接来自客户端的要求，解析发来的信息并调用真正的服务（服务器），并且发回去
    - 客户端helper会解析这些数据
  - RMI会提供客户端和服务器端的helper
    - 协议：JRMP（RMI原生）、IIOP
    - RMI中，称客户端helper为stub，服务器端helper为skeleton

- 创建远程服务

  1. **创建远程接口**：定义了客户端可以远程调用的方法，stub和服务都会实现这个接口
     
     1. 继承java.rmi.Remote
     2. 声明所有的方法都会抛出RemoteException
     3. 确定参数和返回值都是primitive主数据类型或者Serializable的
     
  2. **实现远程接口**：真正的服务
     
     1. 你的服务需要实现你的远程接口
     2. 继承UnicastRemoteObject：使得成为远程服务对象
     3. 编写声明RemoteException的无参数构造函数（跟着throw）
     4. 向RMI registry注册服务
     
     ```java
     try{
       	MyRemote service = new MyRemoteImpl();
     		Naming.rebind("Remote Hello（服务名）",service);
     }
     ```
     
  3. **用rmic产生stub与skeleton**：JDK所附的rmic工具。对真正实现的类执行rmic，会产生出两个helper的`.class`文件
     
  4. **执行rmiregistry**：就像是电话簿，用户（stub）会从此处获得代理

     - 客户端取得stub对象(查询远程服务)

     ```java
     MyRemote service = (MyRemote)Naming.lookup("rmi://127.0.0.1/Remote Hello（服务名）");
     ```

  5. **启动服务**：实现服务的类会初始化服务的实例并向RMI registry注册

- servlet：在http web服务器上运行的Java程序

  1. 下载servlets.jar并添加到classpath上

  2. 通过继承HttpServlet来编写servlet的类

  3. 编写HTML来调用servlet

     ```html
     <a href="servlets/MyServletA">click me</a>
     ```

  4. 给服务器设定HTML网页和servlet

- JSP：使用java写出网页，可以更容易地编写HTML部分



[TOC]

