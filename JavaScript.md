# JavaScript

[TOC]

### #3、4 函数

##### 函数声明

- 所有的函数都有一个name属性

- 作用域

    - 作用域由function进行声明，而不是代码块
    - 即使声明的作用域创建于代码块，但不是终结于代码块

    ```js
    if(window){
      	var x = 2;
    }
    alert(x);
    // 2
    ```

    - 作用域内声明的方法可以提前引用，而变量不行
    
- 匿名函数

    - 注意：属性可以指向方法，但是属性名称不是方法名称


##### 函数调用

-   当函数传入参数数量与函数声明的形参数量不一致时，不会报错

    -   多于时：多余的参数不会被赋值
    -   少于时：不足的形参会被赋值为undefined

-   隐式（implicit）参数

    -   所有的函数都会有两个隐式参数arguments和this
    -   arguments：传递给函数的所有参数的一个集合，类似一个数组
    -   this：this参数引用了与该函数调用进行隐式关联的一个对象，被称之为函数上下文（function context）

-   函数4种调用方式

    -   作为函数

        ```js
        function ninja(){};
        ninja();
        ```

        -   此时，函数的上下文是全局上下文——window对象

    -   作为方法：函数被赋值给对象的一个属性

        ```js
        var o = {};
        o.whatever = function(){};
        o.whatever();
        ```
    
        -   此时，该对象变成了函数上下文，在函数内部可以以this参数的形式进行访问
    
    -   作为构造器
    
        ```js
        function Ninja(){}
        var ninja1 = new Ninja();
        ```
    
        -   构造器函数一般首字母以大写开始
    
    -   使用apply()和call()进行调用
    
        ```js
        function juggle(){
            var result = 0;
            for(var n = 0; n<arguments.length; n++){
                result += arguments[n];
            }
            this.result = result;
        }
        
        var ninja1 = {};
        // 以下两句等效
        juggle.apply(ninja1, [1,2,3,4]);
        juggle.apply(ninja1,1,2,3,4);
        ```
    
        -   可以显式指定任何一个对象作为其函数上下文

##### 将函数视为对象

-   函数和对象一样，可以赋值给变量，也可以拥有属性

    ```js
    var obj = {};
    var fn = function(){};
    obj.prop = "apple";
    fn.prop = "apple";
    ```

-   函数存储

-   缓存记忆



### #5、6 面向对象

##### 构造器

-   
