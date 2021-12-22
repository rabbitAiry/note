[TOC]

# Gradle

### #1 Gradle基础知识

##### GradleWrapper

- Gradle wrapper（gradlew）包含了

  - shell脚本，gradlew是为linux和Mac提供的，而gradlew.bat则是为Windows提供的
  - jar文件

- GradleWrapper是一个指导文件

- GradleWrapper的使用

   - 如果未下载对应gradle，则先执行下载命令
   
   ```shell
   ./gradlew
   ```

   - 可以查看gradle能干什么
   
   ```shell
   ./gradlew tasks
   ```

    - 执行指定任务（示例任务hello）
   
   ```shell
   ./gradlew hello
   ```
   
   - 执行非默认gradle的同名任务（非build.gradle）
   
   ```
    ./gradlew -b solution.gradle hello
   ```
   

##### Gradle

- `.gradle`文件所用的语言是Gradle脚本

- 使用Gradle执行命令（示例任务stringAndTypes）

  ```cmd
  gradle stringAndTypes
  ```

  - 可以使用名字缩写

  ```shell
  gradle sAT
  ```

  - 可以不输出不感兴趣的代码（quiet mode）

  ```sh
  gradle -q sAT
  ```

- gradle语法

  - 可能在闭包内执行

  ```groovy
  task helloWorld {
      doLast{
          println "Hello, World!"        
      }
  }
  ```

  - 也可以直接写

  ```groovy
  task groovy {}
  
  println "Hello Groovy!"
  ```

  - 可以像java代码一样编写

  ```groovy
  task groovy {}
  class JavaGreeter {
      public void sayHello() {
          System.out.println("Hello Java!");
      }
  }
  
  JavaGreeter greeter = new JavaGreeter()
  greeter.sayHello()
  ```

  - 也可以使用def指定类型、直接引用

  ```groovy
  def foo = 6.5
  println "foo has value: $foo"
  println "Let's do some math. 5 + 6 = ${5 + 6}"
  println "foo: ${foo.class} and value: $foo"
  foo = "a string"
  println "foo: ${foo.class} and value: $foo"
  ```

  ```groovy
  def doubleIt(n) {
      n + n // Note we don't need a return statement
  }
  
  foo = 5
  println "doubleIt($foo) = ${doubleIt(foo)}"
  ```

  - 方法的使用甚至可以没有括号，前提是这个方法有参数

  ```groovy
  def noArgs() {
      println "no args function"
  }
  
  def oneArg(x) {
      println "1 arg function with $x"
      x
  }
  
  def twoArgs(x, y) {
      println "2 arg function with $x and $y"
      x + y
  }
  
  oneArg 500 // Look, Ma! No parentheses!
  twoArgs 200, 300
  noArgs()
  //noArgs // Doesn't work
  //twoArgs oneArg 500, 200 // Also doesn't work as it's ambiguous
  twoArgs oneArg(500), 200 // Fixed the ambiguity
  ```

  - 方法可以直接传递

  ```groovy
  def myClosure = {
      println "Hello from a closure"
      println "The value of foo is $foo"
  }
  
  myClosure()
  def bar = myClosure
  def baz = bar
  baz()
  ```

  - lamda

  ```groovy
  def doubleIt = { x -> x + x}
  ```

  - 队列的遍历

  ```groovy
  def myList = ["Gradle", "Groovy", "Android"]
  
  def printItem = {item -> println "List item: $item"}
  myList.each(printItem)
  myList.each{println "Compactly printing each list item: $it"}
  ```

  - 委托

  ```kotlin
  class GroovyGreeter {
      String greeting = "Default greeting"
      def printGreeting(){println "Greeting: $greeting"}
  }
  
  def myGroovyGreeter = new GroovyGreeter()
  
  def greetingClosure = {
      greeting = "Setting the greeting from a closure"
      printGreeting()
  }
  
  // greetingClosure() // This doesn't work, because `greeting` isn't defined
  greetingClosure.delegate = myGroovyGreeter
  greetingClosure() // This works as `greeting` is a property of the delegate
  ```

- Gradle的闭包可以有如下写法

  ```groovy
  project.task("myTask1")
  
  task("myTask2")
  
  task "myTask3"
  
  task myTask4
  ```

- gredle配置

  ```groovy
  myTask4.description = "This is what's shown in the task list"
  myTask4.group = "This is the heading for this task in the task list,"
  myTask4.doLast {println "Do this last"}
  
  task myTask7 {
      description("Description") 
      group = "Some group" 
      doLast {
          println "Here's the action"
      }
  }
  ```

- 运行顺序依赖

  - dependsOn：需要先运行指定任务
  - finalizedBy：需要在本项任务完成后执行指定任务
  - shouldRunAfter：本项任务与指定任务并不互相依赖，但如果要执行both，则应该先执行指定任务

- Gradle文件操作：需指定属性

  - 复制文件

  ```groovy
  task copyImages(type: Copy) {
      from 'images'
      into 'build'
  }
  ```

  - 复制文件，选择格式

  ```groovy
  task copyJpegs(type: Copy) {
      from 'images'
      include '*.jpg'
      into 'build'
  }
  ```

  - 复杂文件操作：将jpeg文件夹和gif文件夹一起放入build文件夹中

  ```groovy
  task copyImageFolders(type: Copy) {
      from('images') {
          include '*.jpg'
          into 'jpeg'
      }
  
      from('images') {
          include '*.gif'
          into 'gif'
      }
  
      into 'build'
  }
  ```

  - 压缩文件：将jpeg文件夹和gif文件夹一起放入build文件夹下的images.zip中

  ```groovy
  task zipImageFolders(type: Zip) {
      baseName = 'images'
      destinationDir = file('build')
  
      from('images') {
          include '*.jpg'
          into 'jpeg'
      }
  
      from('images') {
          include '*.gif'
          into 'gif'
      }
  }
  ```

  - 删除build文件夹

  ```groovy
  task deleteBuild(type: Delete) {
      delete 'build'
  }
  ```

  - 增量构建：Gradle会根据源文件是否有改动（如增加、删除）来判断是否需要执行任务。若无需要，则会显示`UP-TO-DATE`

- 为Gradle Task的执行提供参数（此处示例参数为greeting，任务为printGreeting）

  - 在gradle.properties中提供
    - 此方式下提供文件路径不需要任何引号


  ```groovy
  greeting = "Hello from a properties file"
  ```

  - 在`.gradle`中提供
    - 此方式下提供文件路径不需要任何引号

  ```groovy
  ext {
      greeting = "Hello from inside the build script"
  }
  ```

  - 运行时提供（-P）
    - 此方式下提供文件路径需要任何引号

  ```shell
  gradle -Pgreeting="Hello from the command line" pG
  ```

- 通过继承自定义Task（helloName）

  - 可提供属性

  ```groovy
  class HelloNameTask extends DefaultTask {
      String firstName
  
      @TaskAction
      void doAction() {
          println "Hello, $firstName"
      }
  }
  
  task helloName(type: HelloNameTask) {
      firstName = 'Jeremy'
  }
  ```

- 日志

  - Debug》Info》Lifecycle》Warning》Quiet》Error
  - 当在执行时使用`-q`参数，则只会显示最后两个级别的日志
  - `-d`参数会显示全部日志，`-i`会显示Info及以上级别的日志
  - 默认会显示Lifecycle及以上级别的日志
  - `println`是Quiet级别的日志

- Gradle构建的生命周期

  - 初始化
  - 配置：可以决定任务运行的顺序
  - 执行

- Gradle执行什么

  - 执行`gradle help`时，除了doLast中的语句，其他都会被执行
  - 执行`gradle first`时，执行完`gradle help`中会出现的语句后，还会执行task first中的doLast语句

  ```groovy
  println 'First top level script element'
  
  task first {
      println 'First task: Configuration'
  
      doLast {
          println 'First task: Action'
      }
  }
  
  task second(dependsOn: first) {
      println 'Second task: Configuration'
  
      doLast {
          println 'Second task: Action'
      }
  }
  
  println 'Second top level script element'
  ```

  



[TOC]

### #2 使用Gradle构建Java

[TOC]

### #3 使用Gradle构建Android

[TOC]

### #4 高级Android构建

[TOC]

### #5 最终项目