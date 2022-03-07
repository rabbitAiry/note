[TOC]

# Java进阶

### #1 Maven

- 使用第三方库
  - 传统做法
    1. 下载到本地
    2. 找出里面的jar文件，并添加到Java Build Path
    3. 当代码拷贝到其他电脑中时，需确保拥有同样的配置路径
  - 使用maven：能够自动下载、管理jar包、配置path的构建工具
    1. 新建一个maven project（使用默认项目），其中artifactId为项目名，Group Id以`com.`开头
    2. 在中央仓库（mvnrepository.com）中搜索第三方库
    3. 将依赖文本复制粘贴到新建maven项目的`pom.xml`文件中的`<dependencies></dependencies>`
    4. 对整个项目选择`run as maven build`，并在Goals中填入`clean package`
    5. 之后便可以直接运行`run as Java Application`
  - 将普通java项目转maven项目
    1. 右键中的`configure`选项中选择`Convert to Maven Project`
    2. 修改`pom.xml`文件，同上
- 新版eclipse已集成maven
- maven下载jar包后会存储在本地仓库m2
- maven的生命周期：清理-编译-测试-打包-安装-部署



### #2 单元测试和JUnit

- 软件测试

  - 单元测试：对软件中最小可测试单元进行检查和验证。通常是一个函数。属于白盒测试
  - 集成测试：将多个单元相互作用形成一个整体，对整体协调性进行测试。
    - 先单元测试，再集成测试
  - 白盒测试：全面了解程序内部逻辑结构，对所有的逻辑路径都进行测试。一般由程序员完成
  - 黑盒测试：完全不考虑内部结构，按照说明书对函数进行测序，一般由独立使用者完成
  - 自动测试：用程序来测试程序
  - 手动测试：手动输入参数
  - 回归测试：修改旧代码后，重新进行测试以确保没有引入新的错误

- 测试策略

  - 基于main函数的策略
    - 在新的main函数中逐个测试函数功能是否正确运行
  - 基于自动化测试框架的策略

- JUnit 单元测试框架

  - 示例：构成三角形，对方法`judgeEdges(int a,int b,int c)`测试

    - 新建一个测试类，其中有一个测试方法`(test())`，每一个测试方法的头部都需要加上一个注解`@Test`，这样JUnit就会自动执行这些测试方法

    - `import static`导入一个类的所有静态方法，这样子可以省略（相当于python的import，可见下例）

      `import ststic org.junit.Assert.*`

    - `import`只是导入一个类或者几个类

      `import org.junit.Assert`

    - 断言：断言后者输出结果为false（1，2，3不能构成三角形），若不是，则报异常

      `assertEquals(false,new Triangle().judgeEdges(1,2,3))`

      *若未使用import*

      `Assert.assertEquals(false,new Triangle().judgeEdges(1,2,3))`

  - 实战

    - 业务代码放在`src/main`中，测试代码放在`src/test`中
    - 在`pom.xml`导入JUnit的包
    - 右键测试类，选择Run as JUnit Test，每次只能执行一个Junit的类
    - 亦可以右键项目，选择Run as Maven Test，每次能执行所有的junit类

### #3 高级文本处理

- 中文编码：GB18030-GBK-GB2312

- 字符集Unicode：UTF-8，UTF-16，UTF-32

- ANSI编码：Windows上非Unicode的默认编码，不能在兼容使用

- java内部采用UTF-16编码存储所有字符，所以char可以存储一个汉字

- 获取本机相同默认字符编码

  ```java
  Charset c = Charset.defaultCharset();
  System.out.println(c.name());
  ```

- 输出所有支持的字符集

  ```java
  SortedMap<String,Charset> sm = Charset.availableCharset();
  Set<String> keySet = sm.KeySet();
  for(String s:keySet){
  	System.out.println(s);
  }
  ```

- Java写文件：为了保证不会乱码，读入和输出都应该是同一种编码

##### 使用Java完成国际化编程Internationalization（i18n）

- Locale类
  - 获取系统默认的语言环境

    `Locale locale = Locale.getDefault()`

  - 根据指定资源加载资源文件(bundle即文件包)

    `ResourceBundle bundle = ResourceBundle.getBundle("message",myLocale)`

    `System.out.println(bundle.getString("hello"))`

  - 更换语言

    `= new Locale("en","US)`

  - 返回所有可用的Locale

    `getAvailableLocales()`

- 语言文件：resource目录下的一个properties文件，包含KV对（键值对，中间为=）

  - 必须是ASCII编码文件

  - ASCII外的文字必须使用Unicode来表示\uxxx

  - 可以使用native2ascii.exe进行转码（这是jdk自带的）（cmd中输入代码）

    `native2ascii 文件名.properties message_zh_CN.properties`

- ResourceBundle类

  - 根据Locale要求加载语言文件
  - 搜索路径：有几行，按顺序逐一搜索（包名msg，当前语言en_US，默认语言zh_CN）
    - msg_en_US.properties
    - msg_en.properties
    - msg_zh_CN.properties
    - msg_zh.properties
    - msg.properties
  - 找不到即报错

- 日期、时间、数字、金额国际化

  - DateTimeFormatter和Locale结合
  - NumberFormat和Locale结合

##### 正则表达式

- 组成一个“规则字符串”

- 作用：替换文本、测试字符串内模式（是否符合要求）、提取文本

- 使用java.util.regex(regular express)包、Pattern类用于匹配、Matcher类用于操作

- 用正则表达式查找匹配
  - 使用String保存正则表达式：`String re = "\\bdog\\b"`
    - `\b`表示边界
    - 这种写法表示的是找出一个独立的词。比如其要找的是叫dog的词，则doggy并不算入其中
    - 仅把词保存进正则表达式也是可以的`String re = "dog"`
  - 其他的正则表达式用法
    - `Srting re = "a*b"`这表示匹配结果允许b前面有0或多个的a
  - 把正则表达式编译成pattern类，并保存`Pattern p = Pattern.compile(re)`
  - 使用matcher来匹配并保存位matcher类`Matcher m = p.matcher(String paragraph)`
  - 不断遍历`m.find()`，当能够找到匹配的词时返回true，否则false
    - 每次遍历中都可以调用`.start()``.end()`，这会返回这个词的起始位置和结束位置
  - 使用`m.lookingAt()`和`m.matches()`则会匹配，前者是部分匹配，后者是完全匹配
  
- 用正则表达式替换

  - 匹配过程同上，但是字符串需改为StringBuffer类

  - 循环中，调用`m.appendReplacement(String target,String replace)`

  - 替换结束后（循环外）需把尾巴加上，否则丢失尾部数据

    `m.appendTail(String target)`

  - 调用`m.replaceAll(String replace)`可调换全部词语

- 用正则表达式判断是否为邮箱

##### 其他字符串操作

- 字符串和List对象互转

  - 借助第三方库Apache Commons Lang(使用StringUtils)或者JDK8以上版本
  - 转字符串`String.join(",",List target)`（jdk写法）
  - 转list`Arrays.asList(String target.split(","))`

- 字符串转义

  - 在字符串内表示双引号：使用`\`转义
  - 借助第三方库Apache Commons Text
  - `StringEscapeUtils.escapeJava(String target)`调用则可以把反斜杠也输出。使用`.unescape(...)`则可以取消

- 自动变量驼峰命名

  - 借助第三方库Google Guava

  - List的快速添加

    `=Lists.newArrayList(1,2,3,...)`

  - 驼峰命名

    `=CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL,String target)`

- 字符串转成字符流

  - 借助第三方库Apache Commons IO

    ```java
  //将字符串作为默认的输入流    
    InputStream in = IOUtils.toInputStream(String target,Charsets.toCharset("UTF-8"));
    //重置系统的输入流
    System.setIn(in);
  //模拟输入
    Scanner sc = new Scanner(System.in);
  //分割
    sc.useDelimiter(",");
    //取出
    while(sc.hasNext()){
      System.out.println(sc.next());
    }
    ```
  



### #4 高级文件处理

##### XML

- 可扩展的标记语言：意义+数据
- 特点

  - 每个xml文件都要有一个根元素
  - 大小写敏感
  - 转义字符
- 定义XML文档结构：DTD、XML Schema
- xml的解析方法选择

  - 树结构（DOM）适用于小规模读写
    - 直观易用
    - 将整个树结构读入内存
    - 解析大量的xml文件时，会遇到内存泄漏、程序崩溃的风险
    - 适用解析类DocumentBuilder
  - 流结构（SAX、Stax）适用于读

##### 使用dom来读写xml

- NodeList节点列表
- Node节点

  - Document文档根节点
  - Element标签节点元素
  - Text文本（内容）节点
  - Attr属性节点
  - 空白格：也算是一个节点，且会

- 获得xml

  - 新建`DocumentBuilderFactory`实例

    `= DocumentBuilderFactory.newInstance()`

  - 从`Factory`获取并保存`DocumentBuilder`实例

    `.newDocumentBuilder()`

  - 使用`DocumentBuilder`将解析内容保存给`Document`类型

    `.parse("xml文件名.xml")`

    - 若操作为写，则应这么调用

      `.newDocument()`

- 解析方法1：自上而下进行访问

  - 使用`document`获取一级子节点为头的列表并保存为`NodeList类型`

    `.getChildNodes()`

  - 此时调用`NodeList`的`.getLength()`会返回1，因为根节点只有一个（此处不会有空白格）

  - 获取具体节点：调用`NodeList`的`.item(int Index)`，保存为`Node`类型

  - 获取某个节点之下的节点列表：调用`Node`的`.getChildNodes()`

  - 判断是否为空白：调用`Node`的方法与空白格判断

    `.getNodeType()==Node.ELEMENT_NODE`

  - 获取节点内容：调用`Node`的`.getNodeName()`获得键名，调用其`.getTextContent()`获得值

- 解析方法2：根据标签名（键）搜索

  - 调用`Document`的方法获取根元素，通过`Element`并保存：

    ` .getDocumentElement()`

  - 直接获得所有节点并保存在`NodeList`中，通过使用`Element`的方法

    `.getElementsByTagName("标签名称")`

- 编写方法

  - 调用`Document`的方法创建节点，并保存为`Element`元素

    `.createElement("标签名称")`

  - 调用`Element`的方法

    - 设置属性

      `.setAttribute("属性名称","属性")`

    - 增加节点

      `.appendChild(document.crateTextNode("内容"))`

      `.appendChild(Element node)`

  - 写入xml文件中

    - 使用`TransformerFactory`类。先构造一个新实例

      `.newInstance()`

    - 调用`TransformerFactory`类方法新建`Transformer`并保存

      `.newTransFormer()`

    - 传递根节点（`Document`类）以构建一个`DOMSource`

      `= new DOMSource(Document head)`

    - 使用`File`类新建文件

      `= new File("文件名.xml")`

    - 通过`StreamResult`获得结果

      `= new StreamResult(File target)`

    - 使用`Transformer`将xml内容写入文件中

      `.transform(DOMSource d,StreamResult s)`

  - 此时产生了一个未经整理的（全部在一行）xml文件

##### 使用流模型解析xml

- 采用流模型来解析，更快更更轻量

- 一次性读取，不存储（流水般），故不能同时访问文档中多处数据

- SAX方法

  - 构造一个`XMLReader`

    `XMLReader x = XMLReaderFactory.createXMLReader()`

  - 构造一个处理器：使用一个继承自`DefaultHandler`的类以（BookHandler）达到个人筛选目的

  - （BookHandler）中方法：

    - 成员变量：list、flag
    - 当文档加载时：新建一个List
    - 当文档结束时
    - 访问某一个元素：应该确定是否为搜索目标标签（更改flag）
    - 结束访问元素：继续寻找（重置flag）
    - 访问元素正文（内容）：若找到目标，则增加到list中
    - 获得list方法`getNameList()`

  - 为`XMLReader`设上一个处理器

    `.setContentHandler(DefaultHandler d)`

  - 告知`XMLReader`读什么

    `.parse("文件.xml")`

  - 输出：调用`.getNameList()`

- Stax模型

  - 比SAX更灵活，代码量更少，JDK6+

- Stax:readByStream流模式

- Stax:readByEvent迭代器方法

- 使用第三方库

##### JSON



