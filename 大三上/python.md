# Python基础

> 考勤5% + 平时20% + 在线阶段测试20% + 期末50%

[TOC]

### 初识Python

#### #1、2 程序设计基本方法、实例

- IPO：程序编写方法(Input, Process, Output)

- 同步赋值

  ```python
  x,y = y,x
  
  '''
  t = x
  x = y
  y = t
  '''
  ```

- 字符串：

  - 两种序号体系：正向、反向

  ```python
  str = "hello"
  print(str[-1]) 
  #o
  
  print(str[0:-1])
  #hell
  ```

  

- input()：从控制台获得用户输入，括号内为提示性文字

  - 仅以字符串类型返回结果

- eval()：将字符串变成Python语句，执行该语句并输出

  ```python
  eval("1.1+2.2")
  # 3.3
  
  str = "102C"
  eval(str[0:-1])
  # 102
  
  eval("hello")
  # error: eval函数得到的结果是hello，但是没有这个变量，所以报错
  
  eval("'hello'")
  # 'hello'
  
  value = eval(input("请输入要计算的值："))
  ```

- python允许在表达式内部标记之间增加空格

- print()：槽格式和format()方法

  ```python
  C1, C2 = 10, 10.24024
  print("tempreature:{:.2f}C".format(C1))
  # tempreature10.00C
  
  print("tempreature:{:.2f}C".format(C2))
  # tempreature10.24C
  ```

  - center()方法

  ```python
  print("start".center(scale//2,'-'))
  ```

- 循环

  ```python
  while i<4:
  		# ...
  		
  for i in range(4):
  		#...
  ```

- 函数：用def来定义，缩进关系表示在函数块中

- 库：两种导入方式

  ```python
  import turtle
  # turtle.penup
  
  from turtle import *
  # penup
  # 导入常数或方法，可以一个个导入
  ```

##### turtle库

- 坐标体系：屏幕正中央为(0, 0)
- setup()设置窗体大小及其位置
  - width, height窗口大小
  - startx, starty 窗口相对于屏幕的位置
- 画笔控制函数
  - penup(), pendown()画笔挂起、下笔
  - pensize(), pencolor()
- 形状绘制函数
  - fd()：画笔按照当前方向前进距离(forward)
  - seth():改变画笔前进角度(setHeading)
  - circle():根据半径和角度绘制弧形
  - write():写字
  
  ```python
  turtle.write('年'，font=("Arial",18,"normal"))
  ```
  
  

[TOC]

### 深入Python

#### #3 基本数据类型

- 数字类型：整数、浮点数、复数

  - 整数类型：二进制、八进制、十进制、十六进制。前面加上引导符号（如：0x）。整数取值范围理论无穷大，实际受计算机影响

  - 浮点数类型：十进制表示和科学计数法。精度未必可靠

    ```python
    # 科学计数法
    
    4.3e-3
    # 0.004.3
    
    9.6E5
    9.6E+5
    # 960 000.0
    ```

  - 复数类型：虚数部分通过后缀j来表示。使用`.real`或`.imag`以获取这个数的实数部分和虚数部分

- 数字类型的操作

  - 内置9个数值运算操作符。除正负外，运算符右边加上等号皆能直接赋值(如：+=)

    ```
    + - * /	%				加减乘除求余
    + -				正负
    //				整数商（不大于商的最大整数）
    **				幂
    ```

  - 内置的数值运算函数

    - abs() 绝对值
    - divmod() 同时输出整数商和余数
    - pow()幂，可以幂运算与模运算一起进行
    - round() 四舍五入，可以表示四舍五入到某一位小数
    - max()、min() 可以放入无穷多个数一起比大小

    ```python
    abs(-3+4j)
    # 5.0
    # 复数长度
    
    pow(3,4,5)
    # (3^4)%5
    # 第三个参数可以省略，则不会执行模运算
    ```

  - 内置的数字类型转换函数：int(), float(), complex()

    - 复数不能直接转换成其他数字类型，可以通过.real等转换

    ```python
    int(10.99)
    # 10
    ```

- 例子：天天向上

- 字符串：单引号和双引号皆可以表示为单行字符串。三引号可以表示单行或多行字符串。字符串中使用其他引号作为字符串的一部分

- 字符串操作：连接`x+y`、复制`x*n或n*x`、子串判断`x in s`、返回第i个字符`str[i]`、返回子串`str[N:M]`

- 内置字符串处理函数：求长度`len(s)`、转字符串`str(num)`、返回Unicode对应编码的字符串`chr(num)`、返回单字符表示的Unicode编码`ord()`、返回数字对应十六进制和八进制的字符串形式`hex(), sum()`

- 内置的字符串处理方法

  - 在python解释器内部，所有数据类型都采用面向对象方式实现，封装为一个类。在面向对象中的函数被称为方法
  - 封装在str类中

- format()：字符串类型按照格式输出

  - 槽{}内可以添加数字以表示format中参数的序号
  - 如果要输出大括号，则要这么写：`{{}}`
  - 格式控制：使用`:`作为引导符，之后添加字段
    - 大于号小于号和`^`表示左中右对齐
    - 任意符号表示填充的符号
    - 数字表示字符串宽度（若空，则用空格填充）
    - 精度：`.3f`
    - 逗号用于显示数字类型的分隔符
    - 类型：二进制、八进制等

```python
"{}:计算机{}的CPU占有率为{}%.".format("1","PYTHON",10)
# 1:计算机PYTHON的CPU占有率为10%.

print("{:^{width}}".format('*'*(2*i+1),width=target))
# 输出长度为变量width，j

print("{:+>30.3f}".format(eval(input())**0.5))
# +++++++++++++++++++++++++3.162
```

- 例子：文本进度条 

##### math库

- 仅支持整数和浮点数运算
- 数学常数：pi、e、inf、nan（非浮点数标记）
- 数值表示函数、三角运算函数、高等特殊函数

##### time库

- sleep()：休眠n秒
- clock()：获取时间
- time()：获取时间戳

[TOC]

#### #4、5 程序的控制结构、函数和代码复用

- 程序流程图

- if-else语句

  ```python
  if a>b:
  	print(a)
  else:
  	print(b)
  
  # 下为更简洁的表达
  print(a) if a>b else print(b)
  ```

- 循环

  - 遍历结构可以是字符串、文件、组合数据类型或range()函数

  ```python
  for i in range(3):
  	print(1)
  else:
  	print(2)
  # 使用break退出时不执行else语句
  ```

- 异常处理：try-except语句

  ```python
  try:
  	# ...
  except NameError:
  	# ...
  except:
    # 捕捉所有错误
  else:
    # try中无报错时执行
  finally:
    # ...
  ```

- 函数

- lambda函数（匿名函数）

  ```python
  def sum(a,b):
  	a+b
  
  # 等价于以下lambda函数。可以以同样方式调用
  sum = lambda a,b: a+b
  ```

  - 可选参数（使用默认值顶替）
  - 参数的位置与名称传递

  ```python
  def knock(str,times=2)
  	print(str*times)
  	
  knock("knock~",4)
  #knock~knock~knock~knock~
  
  knock("knock~")
  #knock~knock~
  
  knock(times=4,str="knock~")
  ```

  - 可变数量参数

  ```python
  def sum(a,*b):
  	print(type(b))
  	for n in b:
  		a+=n
  	print(a)
  	
  sum(1,2,3,4,5)
  # <class 'turple'>
  # 15
  ```

  - return可以返回多个结果
  - 函数中调用全局变量
    - 因为整数变量等不需要被定义类型，所以需要指出是全局变量
    - 列表类型等因为需要初始化（list=[]），所以可以被当作全局变量调用

  ```python
  n = 1
  def func(a):
  	global n
  	n = 10
  ```
  
- 实例7：七段数码管

- 模块化：内紧外松

- 函数递归

- python内置函数

  - all(): 如果（如：数组的）每个元素都是true，则返回true
    - 整数0、空字符串、空列表会被当作false
  - any(): 有一true则true
  - hash(): 返回哈希值
  - id(): 返回编号
  - sorted(): 对组合进行排序
  - type(): 函数返回每个数据对应的类型

##### random库

- 计算机的随机数都是伪随机数，是通过算法得到的
- 指定随机数种子seed()：只要种子相同，每次生成随机数序列也相同。这种情况便于测试和同步数据
- random()：生成一个[0.0-1.0)之间的随机小数
- randint()：生成一个[参数一，参数二]之间的随机整数

##### datetime库

- datetime：处理时间的标准函数库

- now()：获取当前日期和时间对象

- utcnow()：获取当前的时间以世界标准时间UTC表示

- sfrftime()：时间格式化输出

  ```python
  someday.strftime("%Y-%m-%d %H:%M:%S")
  # '2016-09-16 22:33:32'
  
  print("今天是{0:%Y}年{0:%m}月{0:%d}日".format(now))
  # 今天是2016年9月16日
  ```

[TOC]

#### #6 组合数据类型

- 序列类型
  - 字符串
  
  - 元组`()`：不可变序列类型
  
  - 列表`[]`
  
  - 序列类型的12个通用操作符
  
    - in, not in, s[i], s[i:j], s[i:j:k], len(s), min(s), max(s)
    - s+t 连接s和t
    - s*n 将序列s复制n次
    - s.index(x[,i[,j]]) 从i到j返回x第一次出现x的位置
    - s.count(x) 序列中出现x的总次数
  
  - 组合数据类型转换：类型名+被转换类型
  
    ```python
    list("hello")
    >>> ['h','e','l','l','o']
    ```
  
  - 数据类型所指为引用
  
- 集合类型

  - 能够进行哈希运算的类型都可作为集合元素
  - 集合`{}`
  - 集合类型的操作符、操作函数

- 映射类型（键值对）

  - 字典
  
  ```python
  doc = {"中国":"北京","美国":"华盛顿","法国":"巴黎"}
  doc["中国"] = '大北京'
  print(doc)
  >>> {"中国":"大北京","美国":"华盛顿","法国":"巴黎"}
  ```
  
  - zip()：将两个程度相同的列表组合成关系对（包含元组的列表）

##### jieba库

> 中文词库的分词，锤子的bigbang



[TOC]

#### #7 文件和数据格式化

- 文件的使用

  - 打开 - 写入 - 关闭

  ```python
  file = open("test.txt","w+")
  ls = ["旋转","跳跃","闭着眼"]
  # 将列表写入一行内
  file.writelines(ls)
  file.seek(0)
  for line in file:
  		print(line)
  file.close
  ```
  
- 一维、二维、高维数据

  - CSV格式：使用逗号分隔数据，每行表示一个一维数据
  - JSON或XML格式：适合高维数据

##### PIL库

> 图片处理库`from PIL import Image`
>
> `pip install pillow`

- 常用读取：open, new, frombytes根据像素创建图像, verify完整性检查
- 常用属性：format格式, mode色彩模式, size, palette调色板属性
- 读取序列类图像文件（如gif）：seek移动到某一帧, tell返回当前帧
- 常用保存：save, convert, thumbnail创建缩略图
- 常用变换：resize, rotate
- 颜色通道相关：point根据操作返回图像副本, split提取每个颜色通道并返回副本, merge合并通道, blend插值为新的图像
- 图像效果 - ImageFilter类中：BLUR, SMOOTH
- 图像效果 - ImageEnhance类中：Brightness, Sharpness

##### json库

> import json

- 字典类型转成JSON格式字符串：json.dumps()
- JSON格式转字典类型（解码）：json.loads()
- 示例：json与csv互转



### 运用Python

#### #8 程序设计方法论

##### pyinstaller库

- 将py文件打包成可执行文件，可用于不同的操作系统

- 源文件必须是UTF-8编码

  ```shell
  pyinstaller -F my.py
  ```

  

#### #9 科学计算和可视化

#### #10 爬虫

##### requests库

- 处理HTTP请求的第三方库
- get()：返回的网页内容会保存为一个Response对象。

##### bs4库

- 解析和处理HTML页面，会建立解析树

