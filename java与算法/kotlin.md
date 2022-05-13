# Kotlin

>   head first kotlin

[TOC]

### #2 基本类型和变量\语法

-   kotlin中的变量可以看作一个杯子。当一个对象被赋值给变量时，变量获得了对象的引用

    -   显式声明变量类型：`var num: Short = 6`使用冒号

-   kotlin数据类型

    -   基本数据类型：Byte、Short、Int和Long，但不支持八进制数

    -   浮点数类型：Float、Double

    -   Char（单引号）、String（双引号）

    -   与Java不同，kotlin中数字类型也是对象

    -   类型转换

        ```kotlin
        // 原理是创建了一个值为5L的Long对象
        // 也就是说x和z指向的不是同一个对象
        var x = 5
        var z:Long = x.toLong()
        ```

-   数组

    -   使用`arrayOf()`创建

        -   数组中只能存放某一特定类型

        ```kotlin
        // 隐式
        var arr = arrayOf(1,2,3)
        println(myArray[0])
        
        // 显式
        var arr2: Array<Byte> = arrayOf(1,2,3)
        ```

-   for循环

    ```kotlin
    // 从1打印到100
    for(x in 1..100)println(x)
    
    // 从1打印到99
    for(x in 1 until 100)println(x)
    ```

    -   使用step来跳过范围数字

        ```kotlin
        for(x in 1..100 step 2)
        ```

    -   使用downTo进行逆序

        ```kotlin
        // 从15打印到1
        for(x in 15 downTo 1)println(x)
        ```

    -   三种访问数组的形式

        ```kotlin
        // 使用foreach
        for(item in array){}
        
        // 使用访问数组下标的方式
        for(index in array.indices){}
        
        // 同时访问下标和元素值
        for((index, item) in array.withIndex()){}
        ```

        



### #4 类和对象

-   定义一个类、创建对象

    -   系统会为新建的对象分配内存，而构造函数负责将对象初始化

    ```kotlin
    // 定义
    class Dog(val name:String, var weight:Int, val breed:String){
        fun bark(){
            println(if(weight<20)"Yip!" else "Woof!")
        }
    }
    
    // 创建
    var myDog = Dog("Fido", 70, "Mixed")
    println(myDog.name)
    myDog.bark()
    ```

    -   使用空构造函数：类名后不带括号
    -   使用初始语句块实现更复杂的操作

    ```kotlin
    init{
    	// ...
    }
    ```

-   kotlin的对象默认必须在构造阶段初始化每一个变量

    -   如果无法在类初始化时为属性分配初始值，则可以使用lateinit作为前缀
    -   lateinit只能使用在var定义的属性上，且不可以用在以下类的属性上：Byte、Short、Int、Long、Double、Float、Char、Boolean。具体原因可以追溯到底层JVM运行代码的机制

-   自定义getter和setter

    -   可以将getter称为acessor，setter称为mutator
    -   编译器会为所有未显式拥有getter和setter的属性定义getter和setter（setter不是必须的，对val而言不会添加）
    -   拥有getter的属性不强制需要初始化

    ```kotlin
    class Dog(val name:String, var weight:Int, val breed:String){
        val weightInKgs: Double
        	get() = weight/2.2
    }
    
    // 调用如下代码时，对象属性的getter会被自动调用
    myDog.weightInKgs
    ```

    -   setter方法中，value（常用的变量名）是准备赋值给属性的值，field是对属性的底层值得引用

    ```kotlin
    class Dog(val name:String, weight_param:Int, breed_param:String){
        val weight: weight_param
        	set(value){
                if(value>0)field = value
                /* 以下写法会陷入无限循环
                if(value>0)weight = value */
            }
    }
    
    // 调用如下代码时，对象属性的getter会被自动调用
    myDog.weightInKgs
    ```




[TOC]

### #5 继承、抽象类、接口

-   使用open关键字声明父类及其能被覆盖的属性和方法

    -   可以使用var覆盖val属性（只需加一个setter即可），但不能使用val覆盖var属性
    -   Java中所有类都是open的，使用final关键字来阻止其被其他类继承或覆盖

-   子类的继承

    -   调用父类构造器

    ```kotlin
    open class Car(val make:String, val model:String){
        // ...
    }
    
    class ConvertibleCar(make_param:String, model_param:String):Car(make_param, model_param){
        // ...
    }
    ```

    -   被覆盖的方法和属性是open的，可以通过使用final关键字来阻止这一行为

    ```kotlin
    open class Vehicle{
    	open fun lowerTemperature(){
            // ...
        }
    }
    
    open class Car:Vehicle(){
        final override fun lowerTemperature(){
            // ...
        }
    }
    ```

-   




[TOC]

### #6 ...







[TOC]

### #7 数据类

-   在kotlin中

    -   Any是所有类的父类

    -   ==操作符会调用equals方法

        -   在Java中，未重写equals时，equals结果与==相同
        -   在Java中，==操作是用于判断基本数据类型值是否相等以及引用地址是否相等，不能被重写

    -   ===操作符不依赖于equals方法，与java的\==相同

        -   在尝试中发现kotlin的基本类型使用===比较的结果相等的，可能是因为版本更新而导致的

        ```kotlin
        var num1 = 10
        var num2 = 10
        println(num1==num2)
        println(num1===num2)
        // z
        var num3 = Integer(10)
        var num4 = Integer(10)
        println(num3==num4)
        println(num3===num4)
        // true true true false
        ```

-   数据类

    -   覆盖率父类的equals方法、hashCode方法、toString方法、copy方法

    -   equals方法中，会自动比较构造器中包括的所有参数。这也意味着在类主体中定义的参数不会被比较

    -   数据类定义了componentN方法，其中N表示被访问属性的编号，按照声明排序
    
        ```kotlin
        val r = Recipe("Chicken Banana", false)
        
        // 1.
        // 等价于 val title = r.title
        val title = r.component1()
        
        // 2.
        // 解构数据对象：将r的属性值依次赋给这两个变量
        val(title, vegetarian) = r
        ```
    
-   辅助构造函数constructor

    ```kotlin
    class Mushroom(val size:Int, val isMagic:Boolean){
        constructor(isMagic_param:Boolean):this(0, isMagic_param){
            // ...
        }
    }
    ```

-   Java代码使用带有默认参数值得kotlin方法

    ```kotlin
    // 方法
    @JvmOverloads fun myFun(str: String = ""){}
    
    // 构造器
    class Foo @JvmOverloads constructor(i:Int = 0){}
    ```

    

[TOC]

### #8 空值和异常

-   在类型后面加上问好表示该类型可以为空

    ```kotlin
    var w: Wolf? = Wolf()
    w = null
    ```

    ```kotlin
    var myArray: Array<String?> = arrayOf("Hi", "Hello", null)
    ```

    -   不应该未声明类型信息就存放null，否则编译器会把该变量视为只能存放空值得变量
    -   Any类型也不包括空值

    ```kotlin
    // 错误示范
    var x = null
    ```

-   确保非空

    -   var属性的变量直接使用if判断非空判断编译失败

        -   因为编译器无法保证别的代码不会在判断不为空和调用方法的间隙更新引用

        ```kotlin
        // 错误示范
        var w: Wolf? = Wolf()
        
        fun myFun(){
        	if(w!=null){
                w.eat()
            }
        }
        ```

    -   解决方法1：使用问号安全访问（链式）

        -   只有在w不为空的情况下才调用eat方法，否则返回null

        ```kotlin
        w?.eat()
        ```

    -   解决方法2：使用let

        -   只有在值不为空时执行代码块中的代码
        -   花括号不可省略

        ```kotlin
        w?.let{
        	it.eat()
        }
        ```

    -   原“使用if表达式令值若空值，则执行其他代码”的解决方法

        -   使用Elvis表达式
        -   表达式因为长得像Elvis猫王而被命名

        ```kotlin
        // 错误示范
        if(w!=null) w.hunger else -1
        
        // 使用Elvis表达式
        w?.hunger ?:-1
        ```

-   断言非空：值为空时抛出NullPointerException异常

    ```kotlin
    var x = w!!.hunger
    ```

-   使用try-catch-finally捕获异常

    -   kotlin不区别检查受控异常和非受控异常

-   类型转换

    ```kotlin
    val r: Roamable = Wolf()
    if(r is Wolf){
        val wolf = r as Wolf
    	wolf.eat()
    }
    ```



[TOC]

### #9 集合

-   list是不可变的，如果不希望如此，则应该使用MutableList
