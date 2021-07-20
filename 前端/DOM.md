## 1 文档对象模型

- （浏览器）解析一个HTML：
  - 标签转为token
  - token转为节点
  - 节点转为树形结构dom
- 在浏览器中输入document可以查看该页面的dom
- dom不属于js，但js可以通过document对象进行访问
- 选择器（get）

  - document.getElementById()需要引号
    - 查无id时返回null
  - document.getElementsByClassName()
  - document.getElementsByTagName()
    - 必须是Elements
    - 返回类似数组但并非数组，而是一种集合
-   元素

  - interface接口：用于创建元素的蓝图
  - properties属性：数据
  - methods方法
- 选择器(query)
  - querySelector
    - 需要带上和CSS一样的选择器表示以及引号
    - querySelectorAll返回元素列表nodelist，可用for-each遍历
