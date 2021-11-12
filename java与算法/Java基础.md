# Java基础

> Thinking in Java  x  Head First Java 笔记

[TOC]

### #1 对象

##### 用引用操纵对象

- 引用和对象相当于遥控器和电视机，使用引用来操控对象

  - 只需遥控器，就可以改变电视机音量大小
  - 遥控器可以单独存在

  ```java
  String s;
  String s = "abcd";
  ```

- new表示创建对象

##### 存储数据的地方

- 寄存器：数量极其有限
- 堆栈：位于RAM中，用于存储方法
- 堆：一种通用的内存池，位于RAM中，用于存放所有的JAVA对象
- 静态存储：在固定位置的RAM中，存放程序运行时一直存在的数据
- 常量存储：存放在代码内部。嵌入式系统下可以选择存放ROM中
- 非RAM存储：数据存活于程序之外

##### 数据类型

- 基本类型
  - boolean、char、byte、short、int、long、float、double、void
  - 这些变量不大，但存储在堆中不如存储在堆栈中高效，所以java提供了不用new来创建变量的方法创建了并非是“引用”的自动”变量“
  - 基本类型具有包装器类，如int类型和Integer类。使用包装器类使得可以在堆中创建一个非基本对象来表示基本类型
- 高精度数字
  - BigInteger和BigDecimal
  - 必须以方法调用的方式取代运算符方式来实现，这是以速度换取了精度（运算慢）
- 数组初始化

##### 对象内部

- 作用域scope
  - 作用域决定了在其内定义的变量名的可见性和生命周期
  - 对象作用域
    - java对象不具备和基本类型一样的生命周期。当其引用作用域结束时，其指向对象仍继续占据内存空间
    - java有一个“垃圾回收器”，用于监视new创建的所有对象，并辨别那些不会再被引用的对象，随后会释放这些内存空间，以便供其他对象的使用
  
- 成员变量和方法（域和方法 field and method）
  - 如果成员变量是一个对象的引用，则有必要使用构造器
  
- 基本成员默认值

  - 若类的某个成员是基本类型，即时没有进行初始化，java也会确保其获得一个默认值（仅 成员变量 拥有默认值）

    | 基本类型 | 默认值   |
    | -------- | -------- |
    | boolean  | false    |
    | char     | null     |
    | byte     | (byte)0  |
    | short    | (short)0 |
    | int      | 0        |
    | long     | 0L       |
    | float    | 0.0f     |
    | double   | 0.0d     |

##### 第一个java程序

- 名字可视化

  ```
  如 com.example.TODO, java.util.ArrayList
  ```

- 注释文档Javadoc

  - Javadoc是用于提取注释（`/** ... */`）的工具，输出一份HTML文件
  - Javadoc只为public和protected成员进行文档注释
  - 能够在注释中嵌入html
  - 可以添加一些标签，如`@see`链接到其他文档、`@version`版本信息、`@param`参数

[TOC]

### #2 控制程序流

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

[TOC]

### #3 初始化和清除

##### static关键字

- 声明一个事物为static时，意味着这个数据或方法可以不创建对象直接访问或使用
- 即使创建同一类的两个对象，其中static成员变量的存储空间一样（只有一份存储空间）
- static方法中不能直接访问非static成员或方法

- java没有东西是全局的，静态类方法最接近全局
- java虚拟机会加载某个类是因为第一次有人要创建该类的新实例，或是使用该类的静态方法或变量。静态变量是在类被加载时初始化的
- 静态的import`import static java.lang.Math.*`

##### 构造器

- 在Java中，**初始化**和创建无法分离

  - java会自动初始化，且在使用构造器之前（大部分初始值都为0）

  - 初始化的内容包括对象的初始化，调用他们的构造器

  - 静态变量会在该类的任何对象创建之前就完成初始化。静态变量会在该类的任何静态方法执行之前被初始化。int类型默认赋值0

  - 初始化子句、静态子句：多个静态初始化动作的集合，只会在其他类使用该类之前就执行初始化，且只执行一次

    - 静态子句（静态初始化程序）前会有static

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

##### 方法重载

- 方法就是给某个动作起名字，为了让方法名字相同而形式参数不同的构造器存在，必须用到方法重载

  - 允许有不同的参数列表，不同的返回值

- 若输入的类型没有对应方法，则会自动提升再寻找

  ```java
  void f(long x) { 
  	System.out.println("long"); 
  } 
  void f(float x) {
  	System.out.println("float"); 
  }
  void main(){
  	f(1) // long
  }
  ```

##### 清除

- 生命周期与作用域

  - 只要变量的堆栈块还在堆栈上，局部变量就还活着

    ```java
    void do(){
    		int cnt = 0;
    		doHomework();
    }
    
    void doHomeWork(){
    	// 尽管cnt不在作用域中，但是do方法仍在堆栈上
    }
    ```

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

- 



[TOC]

### #4 访问权限

##### 包

- 创建CLASSPATH，存放`.class`文件
- 未给定访问权限：仅包内的该类对象可访问（包访问权限、friendly）
- public：皆可访问
- private：确保访问不了（比如阻止别人直接访问某个特定的构造器）
- protected：提供包访问权限，以及为有继承关系的类提供权限
- 类不可以是private或者protected的



### #5 复用类

- 多个类含有main方法：编译器只会执行当前调用类所属main方法。在每个类中都设置main()使得类的单元测试变得简易，且可以保留

##### 组合

- 组合：新的类由现有类的对象所组成
- toString()方法：每一个非基本类型的对象都有一个toString方法，当编译器需要时会自动调用（比如：直接将对象赋给String类型或者输出）

##### 继承

- 所有类会隐式地从object类进行继承
- 优先从当前类中寻找被指定的方法，可以通过super关键字来调用父类方法
- java会在子类的构造器中插入对父类构造器的调用。所以调用子类构造器时，会先调用父类构造器，再执行子类构造器的操作。可以使用super指定调用哪一个构造器
- "is-a(是一个)"的关系是用继承来表达的，可以认为子类是对父类的取代；"has-a"的关系使用组合来表达；"is-like-a"的关系则使用接口来表示，这意味着即时向上转型，也能调用子类额外新增的方法

- 向上转型：子类是父类的一种类型

  ```java
  class Instrument{
  	static void tune(Instrument i){
  		// ...
  	}
  }
  
  public class Wind extends Instrument{
  	public static void main(String[] args){
  		Instrument.tune(new Wind());
  	}
  }
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

  - 使用抽象类：有的类只应该其子类可以被初始化
  - 如果一个类包含抽象方法，则该类必须被限制为是抽象的。抽象类可以没有抽象方法，这时可以阻止产生这个类的对象

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

### #7 接口与内部类

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
  - 实现private接口的类不能被其他类所使用

##### 内部类

- 内部类与组合是不同的概念
  - 与一般的类不同，内部类可以被声明为private或protected
  - 使用内部类的理由：返回引用、创建了不是公用的工具类
  
- 几种内部类

  - 局部内部类：这个类在方法的作用域内，可以访问的常量和当前方法一样
  - 匿名类对象：常用于return中，可以是一个抽象类，在该抽象类的return中，直接new一个子类并向上转型，并补上参数和方法
    - 匿名内部类若使用外部变量，则要求参数是final的（不包括传递参数）
    - 和匿名内部类相比，局部内部类允许有自己的构造器，但是局部内部类的方法在外是不可见的

- 内部类可以访问外围类（即包含着这个内部类的类）的成员，即内部类能够与外部类联系

  - 通过一个特殊的this引用，可以连接到外部类对象

- 嵌套类

  - 若不需要联系，可以将内部类声明为static，此时内部类称为嵌套类。不能从嵌套类对象中访问非static的外围对象
  - 嵌套类也可以置于接口内部，因为它是static的
  - 可以使用嵌套类来放置测试代码（书中建议为每个类都写一个main()方法，提到了使用嵌套类更省事）

- 必须使用外部类的实例来生成内部类（嵌套类除外）

  ```java
  Outer outer = new Outer();
  Outer.Inner inner = outer.new Inner();
  ```

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

- 内部类标识符

  - 每个类都会产生一个.class文件，其中包含了如何创建该类对象的全部信息。接口、内部类也必须生成一个.class文件
  - 内部类的文件命名
    - 外围类的名字加上`$`，再加上数字类标识符，以及内部类的名字

- 内部类的作用/应用场景

  - 当需要在一个类中实现两个同样的接口时（比如同时为两个按钮设置监听事件，意味着你可能需要在一个类中实现两次同样的方法，但这是不可能的），使用内部类能够很好的解决问题
  - 使得类能够像成员变量一样被传递
  - 是一个独立的类，但是需要对某些数据保持联系

- 闭包

  - 闭包是一个可调用的对象，包含了创建它的作用域（如：外围类）
  - 通过内部类提供闭包的功能比指针更灵活、安全
  
- 控制框架与内部类



[TOC]

### #8 异常与错误处理

##### 异常的抛出与捕获

- try-catch

  - try区块放入的是可能导致错误的语句
  - catch关键字后的就是异常处理程序Exception handler，也是上述“恰当的地点”。当异常被抛出时，异常处理机制将负责搜索参数与异常类型相匹配的第一个处理程序，然后加入catch子句执行
  - 异常也可以是多态的

- RuntimeException：不需被检查的异常

  - 其子类包括ClassCastException和NullPointerException等
  - 这些异常大多发生在代码逻辑有误，所以不被编译器检测到（编译器不要求你对抛出这种异常的方法使用try catch）

- 抛出异常后：Java在堆上创建异常对象》当前执行路径终止（跳出）》把问题提交给上一级别的环境（弹出异常对象的引用）》异常处理机制接管程序，并找一个恰当的地点来继续执行程序

- 自定义异常：必须从已有的异常类继承（可以是Exception类）

- 异常说明

  - 将异常告知用户

  - 在方法中使用了关键字throws，后面接一个潜在异常类型的列表（声明有可能抛出的异常）

  - 当被调用的方法中含有throws关键字时，该方法必须使用try-catch块中（编译器要求）

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
  - 使用



p326- p350 特别快

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
  Socket charSocket = new Socket("127.0.0.1",5000);
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



