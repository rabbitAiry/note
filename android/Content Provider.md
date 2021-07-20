## Content Provider

###### 1 使用

##### 用处

- 使得其他app能够安全地访问，使用以及修改



##### 由ContentResolver连接ContentProvider

- 使用URI标识手机文件的具体位置
  - `content://com.example.udacity.droidtermsexample/terms`
  - 这是一个安卓uri的例子
    - `content://`是安卓应用中URI标准起始方式(Content Provider Prefix)，和`http://`一样
    - `com.example. ... example`指定了要使用哪个内容提供器，叫做URI的内容权限（Content Authority）
    - `/`后的数据表示要访问的数据
- 增加几行`update()`

- 删除几行`delete()`
- 更改（更新）数据`insert()`
- 读取数据`query()`



##### 使用

1. 获取使用权限

   ```
   <uses-permission "某个app的package TERMS_READ">
   ```

2. 获得`ContentResolver`

   - 使用`getContentResolver()`获取ContentResolver

   - 使用URI标识手机文件的具体位置

     `content://com.example.udacity.droidtermsexample/terms`

     这是一个安卓uri的例子

     - `content://`是安卓应用中URI标准起始方式(Content Provider Prefix)，和`http://`一样
     - `com.example. ... example`指定了要使用哪个内容提供器，叫做URI的内容权限（Content Authority）
     - `/`后的数据表示要访问的数据

3. 对数据进行增删改查

   通过resolver以及操作类型操作数据，并赋予给Cursor类

   `Cursor cursor = resolver.query(对应uri ,null,null,null,null) `

4. 数据识别、利用

   - `4`中的代码四个null，分别表示
     - 选择哪些列
     - 如何选择行
     - 选择哪些行
     - 排序方式



##### Cursor

- `.moveToNext()`光标不断向下移，若可移动返回true，否则返回false
- `.moveToFirst()`回到第一行
- `.getColumnIndex(String heading)`获得位置
- `.getString(int position)`获取元素
- `.close`不要忘了关闭它



###### 2 提供

##### 在清单中声明

- `<application></application>`标签内，使用`<provider/>`标签
- 标签内增加三个属性：
  - `authorties`填入package的名称，这可以在清单里面看到
  - `name`填入package的名称以及provider的名字，这里的provider在`data`文件夹中，所以是`.data.TaskContentProvider`
  - `exported`设为false时其他应用不能轻易访问

