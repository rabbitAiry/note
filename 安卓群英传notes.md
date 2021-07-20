[TOC]

# 安卓群英传notes

##### 安卓结构控件

- view group控件
- view控件
  - 控件树
  - 上层控件负责下层子控件的测量与绘制，并传递交互事件。
  - 每棵控件树的顶部，都有一个ViewParent对象

- set content view方法
  - Android界面布局
    每个activity都包含一个Windows对象（通常由PhoneWindow来实现）。PhoneWindow将一个DecorView设置为整个应用窗口的根View，在显示上，它将屏幕分成两部分，一个是TitleView，另一个是ContentView。
  - 它是一个ID为content的Framelayout, activity_main.xml就是设置在这样一个Framelayout里。
  - 用户通过设置requestWindowFeature（Window.FEATURE_NO_TITLE)来设置全屏显示，视图树中的布局就只有Content了，这就解释了为什么调用requestWindowFeature()方法一定要在调用setContentView()方法之前才能生效的原因
  - 当程序在onCreate（）方法中调用setContentView()方法后，ActivityManagerService会回调onResume()方法，此时系统才会把整个DecorView添加到PhoneWindow中

##### 测量与绘制

- view的测量：on Measure（）

  > 通过MeasureSpec类

  - 测量模式
    - exactly：默认，将控件属性指定为具体数值时或者match_parent
    - at_most：属性指定为wrap_content
    - unspecified：绘制自定义view时
    - 非默认属性下需要重写

  ```
  private int measureWidth(int measureSpec) {
              int result = 0;
              int specMode = MeasureSpec.getMode(measureSpec);
              int specSize = MeasureSpec.getSize(measureSpec);
              if (specMode == MeasureSpec.EXACTLY) {
                result = specSize;
              } else {
                result = 200;
                if (specMode == MeasureSpec.AT_MOST) {
                    result = Math.min(result, specSize);
                }
              }
              return result;
          }
  ```

- view的绘制：onDraw（）

  - 使用canvas对象
    `Canvas canvas = new Canvas(bitmap);`
  - bitmap 用于记载所有绘制在canvas上的像素信息

- viewgroup测量
  - 负责子view显示大小
  - 当ViewGroup的大小为wrap_content时，ViewGroup就需要对子View进行遍历，以便获得所有子View的大小，从而来决定自己的大小


