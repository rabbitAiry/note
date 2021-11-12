

# Android开发艺术探索 笔记

[TOC]



### #1 Activity的生命周期和启动模式

##### 1.1 Activity的生命周期全面分析

- 典型情况下的生命周期分析

  - 生命周期

    - `onCreate()`和`onDestroy()`是配对的
    - `onStart()`和`onStop()`是配对的，熄屏等操作会调用。此时Activity存在（可能可见）但处于后台
    - `onResume()`和`onPause()`是配对的，熄屏等操作会调用
    - `onPause()`和`onStop()`都不应该执行耗时的操作（尤其是前者）
    - 只要是从`onStop()`到`onStart()`，中间一定经过`onRestart()`（不销毁就重启）

  - 打开第二个Activity的生命周期

    ```
    当前activity：activity1
    准备启动activity2
    ------
    Activity1	onPause()
    Activity2	onCraete()>>onStart()>>onResume()
    Activity1	onStop()
    ```

- 异常情况下的生命周期分析

  - 情况1：资源相关的系统配置发生改变导致Activity被杀死重建：例如横屏换竖屏

    - 系统会在`onDestroy()`执行前调用`onSaveInstanceState()`来保存当前Activity的状态
    - 系统会在新Activity调用`onCreate()`后执行`onRestoreInstanceState()`。系统会自动保存当前Activity的视图结构，并在此恢复，如EditText中用户输入的数据，ListView滚动的位置
    - 正常启动时，`onCreate()`传入的bundle为空

  - 情况2：资源内存不足导致低优先级的Activity被杀死

    - 对于Activity，处于后台且不可见的Activity优先级最低，系统会杀死Activity所在进程。杀死后仍通过bundle存储和恢复数据
    - 如果一个进程没有四大组件在运行，则很快被杀死

  - 令Activity屏蔽发生变化的系统配置（不杀死以重新创建）：给Activity指定configChanges属性

    ```groovy
    android:configChanges="orientation|ScreenSize"
    // 屏幕方向
    ```

    - 更多configChanges：p14

##### 1.2 Activity的启动模式

- Activity的LaunchMode④
  - standard：默认
  - singleTop：如果当前Activity已位于栈顶，则此Activity不会被重新创建（不会调用这个Activity的onCreate和onStart），同时它的`onNewIntent()`方法会被调用
  - singleTask：系统首先会寻找是否存在activityA想要的任务栈，如果不存在，则重新创建一个任务栈，然后将activityA加到栈中。如果存在且activityA在栈内，则在activityA上的Activity全部出栈。activityA的`onNewIntent()`方法会被调用
  - singleInstance：具有singleTask模式的所有特性，且此种模式的Activity只能单独地位于一个任务栈中
  - 特殊情况
    - 使用ApplicationContext去启动standard模式的Activity时会报错。解决方法是为待启动的Activity指定标志位FLAG_ACTIVITY_NEW_TASK，此时以singleTask模式启动该Activity
    - 若存在两个任务栈，前台任务栈有ActivityBA（顶》底），后台任务栈有ActivityDC。CD启动模式均为singleTask。当在前台请求启动D时，整个后台任务栈都会加载到前台中。当在前台请求C时，只有C会加载到前台中
- Activity所要的任务栈：与参数TaskAffinity相关，在Manifest中或intent中指定
  - :flags:
- Activity的Flags
  - FLAG_ACTIVITY_NEW_TASK：指定启动模式为singleTask
  - FLAG_ACTIVITY_SINGLE_TOP：指定启动模式为singleTop
  - FLAG_ACTIVITY_CLEAR_TOP：指被启动的Activity在栈中以上的所有Activity都出栈。若启动模式为standard，则该Activity先出栈再重新创建
  - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：具有这个标记的Activity不会出现在历史列表中

##### 1.3 IntentFilter的匹配规则

- 需指定action、category和data信息，可以匹配多个。一个activity中也可以有多个intent-filter
- 匹配规则：:flags:



### #2 IPC机制

> Inter-Process Communication进程间通信

- [x] ——Binder前
- [x] ——Binder
- [x] ——AIDL部分
- [ ] ——AIDL
- [ ] ——2.5前
- [ ] ——#3前

##### 2.1 Android IPC简介

- 进程与线程
  - 线程是CPU调度的最小单元，是一种有限的资源
  - 进程指一个执行单元，在PC和移动设备上指一个程序或app。一个进程可以包括多个线程
  - 任何一个系统都有IPC机制。Android中IPC方式有Binder、Socket等
- 使用多进程的场景
  - 因为某些原因需要运行在单独的进程中
  - 加大应用可使用的内存而使用多进程
  - ContentProvider或者其他直接获取其他应用的数据的方式
- 利用adb shell查看进程
  - `ps -A`
  - 从结果中匹配名字：`ps -A|grep com.***.***`

##### 2.2 Android中的多进程模式

- 开启多进程模式

  - 在Android中使用多进程方法：给四大组件在Manifest中指定`android:process`属性，或在native层fork一个进程

    ```xml
    // :代表了包名。以冒号开头的进程都是私有进程
    android:process=":test2"
    ```

- 多线程的运行机制

  - 多进程会带来许多问题
    - 无法通过内存来共享数据，静态成员和单例模式完全失效（不同进程分配给了不同的虚拟机，所以不同进程访问同一个对象的时候实际访问的是该对象的多个副本之一，副本之间互不干扰）
    - 线程同步机制完全失效（锁的不是同一个对象）
    - SharedPreference的可靠性下降（不支持两个进程同时执行读写操作）
    - App会多次创建（基于独立性）

##### 2.3 IPC基础概念介绍

- Serializable接口和Parcelable接口可以完成对象的序列化过程

- Serializable接口
  - java提供，简单但开销大，需要大量IO操作，存储到设备上或者网络传输也更方便
  - 只需要在类的声明中指定一个id即可自动实现默认的序列化过程（id用于表示类的版本，java会自动生成。手动指定该id可以提高恢复成功率）
  - 经过序列化和反序列化后两者内容一样，但不是同一个对象
  
- Parcelable接口
  - android提供的序列化方式，麻烦但效率高。主要用于内存序列化上
  - 只要实现了这个接口，一个类的对象就可以实现序列化并可以通过Intent和Binder传递
  - CREATOR和private构造器用于反序列化
  - `writeToParcel()`用于序列化
  - `describeContents()`用于内容描述，默认返回0，仅当当前对象中存在文件描述符时返回1
  
- Binder

  - 所有可以在Binder中传输的接口都需要继承IInterface接口

  - transcat过程

    - 当客户端和服务端位于同一个进程下时，方法调用不会走跨进程的transcat过程；反之则需走transcat过程，使用proxy完成
    - 使用了id标识在transcat过程中的客户端所请求的是哪个方法。会自动为每个aidl中声明的方法生成id

  - Binder（Stub和Stub的内部代理类Proxy）的属性与方法

    > Stub就是客户端中假装成服务的客户端helper。Skeleton是服务器端helper

    - DESCRIPTOR：Binder的唯一标识。一般以Binder的包名+类名表示
    - asInterface：将服务端的Binder对象转换为客户端所需要的AIDL接口类型对象。如果客户端和服务端位于同一进程，则返回的是服务端的Stub对象本身，否则返回的是系统封装后的Stub.proxy
    - asBinder：用于返回当前的Binder对象
    - onTransact：此方法运行在服务端中的Binder线程池中。根据方法id处理客户端发起的请求
    - Proxy#（= Proxy下的）getBookList：此方法运行在客户端。根据id执行不同的操作

  - Binder工作机制

    <img src="image/Binder工作机制.png"  />

    - 当客户端发起请求时，当前线程将会挂起直到服务端进程返回数据
    - 服务端的Binder应该采用同步的方式实现，因为已经存在于线程池中
    - linkToDeath与unlinkToDeath：因为服务器端有可能由于某种原因而异常终结，设置死亡代理以防止客户端不知道服务器端暴毙的情况

##### 2.4 Android中的IPC方式

- 使用Bundle：在Bundle中添加数据并使用Intent发送，在四大组件间通信，对数据类型有限制
- 使用文件共享
  - 对文件格式无限制，但是难免避开并发性的问题。适用于对同步要求不高的进程之间通信
  - SharedPreference也属于文件的一种，但由于系统对其读写有一定的缓存策略，因此在多进程模式下，系统对它的读写并不可靠，面对高并发读写访问时，有很大几率会丢失数据
- 使用Messenger

  - 可以在不同进程之间传递Message对象。本质是AIDL
  - 只能一个个处理，难以完成并发请求
  - :flags: 
- 使用AIDL：
  - 服务端：创建Service用来监听客户端的连接请求、创建一个AIDL文件用于暴露接口、在Service中实现这些接口
  - 客户端：绑定Service并将其返回的Binder还原、调用AIDL提供的方法
  - AIDL接口的创建
  - :flags: 实践部分
- 使用ContentProvider
- 使用Socket

##### 2.5 Binder连接池

##### 2.6 选用合适的IPC方式

##### ex#1 初次使用多进程

- 创建AIDL文件

  - 右键创建aidl文件，编写代码完成后执行assembleDebug。执行后生成了对应(同名)的Binder接口，位于generated目录下
  - 如果AIDL文件使用到了自定义的Parcelable对象，那么必须新建一个和它同名的AIDL文件，并在其中声明为Parcelable类型

  ```java
  // IBookManager.aidl
  package com.example.myaidl;
  import com.example.myaidl.Book;
  
  interface IBookManager{
      List<Book> getBookList();
      void addBook(in Book book);
  }
  ```

  ```java
  // Book.java
  public class Book implements Parcelable{...}
  ```

  ```java
  // Book.aidl
  package com.example.myaidl;
  parcelable Book;
  ```

- Service中构建Binder对象

  - CopyOnWriteArrayList支持并发读写的List
  - 与基础篇的Binder构建不同，此处的Binder类由AIDL产生的类提供
  - 声明进程

  ```xml
  <service android:name=".BookManagerService"
      		 android:process=":remote"/>
  ```

  ```java
  // BookManagerService.java
  private CopyOnWriteArrayList<Book> mBooklist = new CopyOnWriteArrayList<>();
  
  private Binder mBinder = new IBookManager.Stub() {
      @Override
      public List<Book> getBookList() throws RemoteException {
          return mBooklist;
      }
  
      @Override
      public void addBook(Book book) throws RemoteException {
          mBooklist.add(book);
      }
  };
  
  @Override
  public void onCreate() {
      super.onCreate();
      mBooklist.add(new Book("十万个为什么",0));
      mBooklist.add(new Book("吸血鬼的故事",1));
  }
  
  @Override
  public IBinder onBind(Intent intent) {
      return mBinder;
  }
  ```

- Activity中连接：和基础篇绑定Binder一样，使用ServiceConnection

  ```java
  private ServiceConnection mConnection = new ServiceConnection() {
      @Override
      public void onServiceConnected(ComponentName name, IBinder service) {
          IBookManager bookManager = IBookManager.Stub.asInterface(service);
          try {
              List<Book> list = bookManager.getBookList();
              Log.i(TAG, "query book list, list type: "+list.getClass().getCanonicalName());
              Log.i(TAG, "query book list: "+list.toString());
          }catch (RemoteException e){
              e.printStackTrace();
          }
      }
  
      @Override
      public void onServiceDisconnected(ComponentName name) {
  
      }
  };
  ```

  ```java
  Intent intent = new Intent(this, BookManagerService.class);
  bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
  ```

##### ex#2 完善

- :flags: 链接p92+



[TOC]

### #3 View的事件体系

##### 3.1 View的基础知识

- 什么是View
- View的位置参数
- MotionEvent和TouchSlop
- VelocityTracker、GestureDetector和Scroller

##### 3.2 View的滑动

- 使用scrollTo或scrollBy
- 使用动画
- 改变布局参数
- 各种滑动方式的对比

##### 3.3 弹性滑动

- 使用Scroller
- 通过动画
- 使用延时策略

##### 3.4 View的事件方法机制

- 点击事件的传递规则
- 事件分发的源码解析

##### 3.5 View的滑动冲突

- 常见的滑动冲突场景
- 滑动冲突的处理规则
- 滑动冲突的解决方式



### #4 View的工作原理

##### 4.1 初始ViewRoot和DecorView

##### 4.2 理解MeasureSpec

- MeasureSpec
- MeasureSpec和LayoutParams的对应关系

##### 4.3 View的工作流程

- measure过程
- layout过程
- draw过程

##### 4.4 自定义View

- 自定义View的分类
- 自定义View须知
- 自定义View示例
- 自定义View的思想



### #5 理解RemoteViews

##### 5.1 RemoteViews的应用

- 通知栏上
- 桌面小部件上
- PendingIntent概述

##### 5.2 RemoteViews的内部机制

##### 5.3 RemoteViews的意义



### #6 Android的Drawable

##### 6.1 Drawable简介

##### 6.2 Drawable的分类

- BitmapDrawable
- ShapeDrawable
- LayerDrawable
- StateListDrawable
- LevelListDrawable
- TransitionDrawable
- InsetDrawable
- ScaleDrawable
- ClipDrawable

##### 6.3 自定义Drawable



### #7 Android动画深入分析

##### 7.1 View动画

- View动画的种类
- 自定义View动画
- 帧动画

##### 7.2 View动画的特殊使用场景

- LayoutAnimation
- Activity的切换效果

##### 7.3 属性动画

- 使用属性动画
- 理解插值器和估值器
- 属性动画的监听器
- 对任意属性做动画
- 属性动画的工作原理

##### 7.4 使用动画的注意事项



### #8 理解Window和WindowManager

##### 8.1 Window和WindowManager

##### 8.2 Window的内部机制

- Window的添加过程
- Window的删除过程
- Window的更新过程

##### 8.3 Window的创建过程

- Activity的Window创建过程
- Dialog的Window创建过程
- Toast的Window创建过程



### #9 四大组件的工作过程

##### 9.1 四大组件的运行状态

##### 9.2 Activity的工作过程

##### 9.3 Service的工作过程

- Service的启动过程
- Service的绑定过程

##### 9.4 BroadcastReceiver的工作过程

- 广播的注册过程
- 广播的发送和接受过程

##### 9.5 ContentProvider的工作过程



### #10 Android的消息机制

##### 10.1 Android的消息机制概述

##### 10.2 Android的消息机制分析

##### 10.3 主线程的消息循环



### #11 Android的线程和线程池

##### 11.1 主线程和子线程

##### 11.2 Android中的线程形态

- AsyncTask
- AsyncTask的工作原理
- HandlerThread
- IntentService

##### 11.3 Android中的线程池

- ThreadPoolExecutor
- 线程池的分类



### #12 Bitmap的加载和Cache

##### 12.1 Bitmap的高效加载

##### 12.2 Android中的缓存策略

- LruCache
- DiskLruCache
- ImageLoader的实现

##### 12.3 ImageLoader的使用

- 照片墙效果
- 优化列表的卡顿现象



### #13 综合技术

##### 13.1 使用CrashHandler来获取应用的crash信息

##### 13.2 使用multidex来解决方法数越界

##### 13.3 Android的动态加载技术

##### 13.4 反编译初步

- 使用dex2jar和jd-gui反编译apk
- 使用apktool对apk进行二次打包



### #14 JNI和NDK编程

##### 14.1 JNI的开发流程

##### 14.2 NDK的开发流程

##### 14.3 JNI的数据类型和类型签名

##### 14.4 JNI调用Java方法的流程



### #15 Android性能优化

##### 15.1 Android的性能优化方法

- 布局优化
- 绘制优化
- 内存泄漏优化
- 响应速度优化和ANR日志分析
- ListView和Bitmap优化
- 线程优化
- 一些性能优化建议

##### 15.2 内存泄漏分析之MAT工具

##### 15.3 提高程序的可维护性