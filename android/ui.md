[TOC]

# UI

##### 基础

- ViewGroups
  - FrameLayout
  - LinearLayout
  - RelativeLayout
  - GridLayout
  - ConstraintLayout
    - 相对于嵌套复杂的ui，仅使用ConstraintLayout的性能更强
  - 皆继承至View类
- Gravity和LayoutGravity
  - 前者让容器内容居中
  - 后者让容器居于父容器中居中
- padding和layout_margin
- View visibility
- 设置暂时的数据（便于观察布局）
  - 使用`tools:text=""`而不是`android:text=""`

##### 数据绑定

1. 在Gradle中与`compileSdkVersion 28`同级允许数据绑定

   `dataBinding.enabled = true`

2. 在使用数据绑定的页面文件xml中填写(使用)根元素`<layout>...</...>`

3. 创建一个类型为`ActivityMainBinding`的变量

4. 把`DataBindingUtil.setContentView(this,R.layout.指定layout)`的值保存（这是`ActivityMainBinding`类型）

5. 逐个`setText(String target)`



- 这样做的好处是：不需要逐个地为页面中的元素使用`findViewById`，但对数据更改的操作依旧不变

##### 无障碍

- 仅需为含有图片的view增加描述，因为含有文字的view会被安卓朗读

- 添加描述方法：view中增加属性

  `android:contentDescription="@string/origin_label"`

##### 国际化

- 在`values`同级处添加文件夹`values-语言代号`
- 安卓会自动加载和本地语言匹配的语言
- 如果对不打算翻译的字符串使用字符串资源，可以在`strings.xml`中的属性设为`translatable="false"`
- 从右到左阅读的语言配置

##### 响应式布局

- 为了让横向布局的内容展示更加多，可以仅为横向模式新建完全不同的XML布局
- 新建一个文件夹`layout-land`，并将原本布局复制于此
- 可以将几个可以视为一块的view放入不同的xml文件中，要更改那些受影响的约束
- 在原本布局中放入xml中的view使用根标签\<include>，添加属性`layout="@layout/名字"`，无需加.xml



