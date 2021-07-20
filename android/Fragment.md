[TOC]

# Fragment

> 代表Activity中用户界面的独立部分，可以看作是灵活的小Activity

- 可以完全模块化Activity，也可以在多个Activity中重复使用这些fragment
- fragment必须始终嵌入到Activity中，fragment拥有自己的生命周期，但其会受到Activity的生命周期的影响（但不一样）
- 可以在运行时动态地添加或删除



##### 生命周期

- onCreateView
- onDestroyView



------

#### 使用

###### 创建Fragment

- 创建一个用于展示fragment的布局文件，以及一个类，继承自`Fragment`

- 重写类中的`onCreateView()`方法：

  - 创建一个View对象，并使其填充自fragment的布局文件
  - 绑定该View中的widget（findViewById）
  - 返回View对象

  ```java
  View v = inflater.inflate(R.layout.fragment, viewGroup对象, false);
  // ...
  return rootView;
  ```

- 在activity布局文件中放入fragment布局的容器（通常使用FrameLayout）

  - 除非是一个静态的fragment（在Activity运行期间不会更改形态），否则必须添加容器

###### 使用FragmentManager和事务

> FragmentManager管理着fragment的增删改

- 在`Activity.java`中实例化一个fragment类
- 新建并获取一个FragmentManager
- 调用FragmentManager的方法开启事务以展示fragment
  - add()\replace()
  - commit()

  ```java
  FragmentManager fm = getSupportFragmentManager();
  // transaction：事务
  fm.beginTransaction()
    .add(R.id.容器, fragment对象).commit();
  ```

