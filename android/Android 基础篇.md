# Android 基础篇

[TOC]

## 简介篇

- 系统架构④
  - Linux内核层：为Android提供了底层的驱动
  - 系统运行库层：通过一些C/C++库来为Android系统提供了主要的特性支持，如SQLite库提供了数据库的支持、OpenGL|ES库提供了3D绘图的支持，Webkit提供了浏览器内核的支持。同时在这一层的还有Android运行时库，能够允许开发者使用java语言来编写app，包含了Dalvik虚拟机（5.0后：ART）
  - 应用框架层：提供了构建应用程序时可能用到的各种API
  - 应用层：手机上的所有app
- 日志工具
  - 级别越下越高
    - Log.v() 打印最为琐碎的日志信息verbose
    - Log.d() 打印调试信息debug
    - Log.i() 打印重要信息info
    - Log.w() 打印警告信息warn
    - Log.e() 打印错误信息error
  - 可以自定义logcat过滤器，只显示过滤结果



## Activity篇

### #生命周期

- 返回栈：安卓使用Task(任务)来管理活动，一个Task就是一组存放在栈里的活动的集合，这个栈称为Back Stack(返回栈)
- 活动状态④：运行、暂停（仍可见）、停止（不可见）、销毁
- 生命周期⑦：onCreate(), onStart(), onResume(), onPause(), onStop(), onDestroy(), onRestart()
- 生存期③：完整生存期、可见生存期、前台生存期
- 保存临时数据onSaveInstanceState()。在获取还原时应该先检查其是否为空
- activity的启动模式④
  - 在Manifest的activity标签内 指定launchMode
  - standard（默认）：常规的栈，即使是同样的activity也会被重复进栈
  - singleTop：仅当返回栈的栈顶也就是该activity，不再创建新的activity实例
  - singleTask：如果返回栈中存在该activity，则不断出栈直到栈顶为该activity
  - singleInstance：包含了singleTask模式的所有特性，除此之外，每次启动新activity时都会放入其他返回栈中
- 获取自身信息
  - 返回自身id：this.toString()
  - 返回所在栈id：getTaskId()



### #Fragment

- 代表Activity中用户界面的独立部分，可以看作是灵活的小Activity。可以完全模块化Activity，也可以在多个Activity中重复使用这些fragment
- fragment必须始终嵌入到Activity中，fragment拥有自己的生命周期，但其会受到Activity的生命周期的影响（但不一样）
- 可以在运行时动态地添加或删除




##### *1 创建Fragment

- 创建一个用于展示fragment的布局文件

- 创建一个类，继承自Fragment类。重写Fragment子类中的`onCreateView()`方法：

  - 创建一个View对象，并使其填充自fragment的布局文件
  - 绑定该布局中的view（findViewById）
  - 返回View对象

  ```java
  View view = inflater.inflate(R.layout.fragment, viewGroup对象, false);
  return view;
  ```
  
- 创建静态的fragment：仅使用xml指定

  - 使用fragment标签，并在属性name中指定对应fragment类
  
  ```xml
  <fragment
      ...
      android:name="com.example.app.LeftFragment"/>
  ```
  
- 创建动态的fragment

  - 不使用fragment标签创建view，而是使用容器来放置view。通常使用FrameLayout
  - 使用FragmentManager和事务

    - 在`Activity.java`中实例化fragment的子类
    - 使用FragmentManager的事务transation动态添加、更换fragment
    - 最后使用commit()确认

  ```java
  // transaction：事务
  FragmentManager fm = getSupportFragmentManager();
  fm.beginTransaction()
    .add(R.id.容器, fragment对象).commit();
  ```

  - 同一个FragmentManager的引用可以对屏幕上所有的fragment分别操作
  
- fragment中模拟返回栈效果（按下Back键后，返回上一个fragment）

  - 在transaction中多加一步`.addToBackStack(null)`即可



##### *2 fragment的交互、状态保存（bundle）

> 需求：每当点击fragment时，切换下一张照片；当屏幕翻转时，避免重新创建fragment、照片顺序重新开始

- 在Fragment类中
  - 直接给imageView设置监听器`(setOnClickListener(new OnClickListener(){...}))`
  - 重写`onSaveInstanceState(Bundle state)`，并将成员变量保存至此（list对象可以先进行强制类型转换，然后使用`putIntegerArrayList()`）
  - 在`onCreateView()`中，如果接收到的bundle非空，则把其数据重新给到成员变量
- 在Activity类中
  - 通过判断bundle是否为空的方式决定是否需要重新创建fragment



##### *3 Fragment生命周期

- 4种状态
  - 运行状态
  - 暂停状态：相关的activity进入了暂停状态
  - 停止状态：相关的activity进入了停止状态，或者使用了addToBackStack()方法且fragment仍在栈种
  - 销毁状态：相关的activity被销毁，或调用了replace()、remove()方法，且没有使用addToBackStack()方法
- 回调
  - onAttach()：碎片和活动建立关联时
  - onCreate()：创建时。先关联再创建
  - onCreateView()：创建（加载）布局时
  - onActivityCreated()：与fragment关联的activity创建完毕时
  - onDestroyView()：布局被移除时（使用栈时，只移除布局，不销毁fragment）
  - onDetach()：解除关联时
  - ...



##### *4 Fragment之间的通信、方法调用

> 需求：在fragment1中传递信息给fragment2展示

- Fragment之间不能直接通信，需要借助Interface在Activity中实现

  - 在Fragment1中

    - 创建一个Interface对象，以及方法

      ```java
      MyListener mCallback;
      
      public interface MyListener{
      		void onShow(){//...}
      }
      ```

    - 重写onAttach()方法，确保其容器activity实现了接口

      ```java
      @Override
      public void onAttach(Context context){
      		super.onAttach(context);
      		try{
      				mCallback = (MyListener) context;
      		}catch(ClassCastException e){
      				throw new ClassCastException(context.toString()+" must implement MyListener")
      		}
      }
      ```

    - 在重写方法onCreateView()中，为gridView设置该侦听器

      ```java
      gridView.setOnItemClickListener(new AdapterView.OnItemClickListener(){
      		@Override
      		public void onItemClick(AdapterView<?> adapterView, View view, int position, long l){
      		mCallback.onShow();
      		}
      })
      ```

  - 在Activity中

    - 实现该接口、实现该方法。同时在该方法中将信息放入bundle中

  - 在Fragment2中

    - 通过getIntent()方法尝试获得被传递的值，若获取不到，则使用参数2的默认值

      ```java
      int index = getIntent().getIntExtra("tag",0);
      ```


- 调用Fragment或Activity中的方法

  ```java
  // 获取fragment
  MyFragment frag = (MyFragment)getFragmentManager()
    .findFragmentById(R.id.my_fragment);
  
  // 获取Activity
  MainActivity acti = (MainActivity)getActivity();
  ```

  

##### *5 双fragment布局

> 需求：平板上方便获取更多信息

- 在res文件夹下（使用限定符）
  - 新建Android Resource Directory：选择layout文件、最小宽度为600dp，然后再新建一个和原来layout文件同名的activity.xml
  - 将需要合并的页面也放在这个布局中
  
- 在Activity.java中
  - 判断是否为双页面：通过findViewById()查找一个双页面下有的id，若不为空，则为双页面
  
  ```java
  isTwoP = getActivity()
    .findViewById(R.id.frag) != null;
  ```
  
  - 修改细节：比如传递intent的button可以去掉，传递方式也改为直接修改fragment，设置行宽等



### #Widget

##### *1 创建一个widget放在桌面上

> 点击时启动app

- 创建一个widget的布局

- 在res/xml下创建一个记录widget信息的xml文件

  - updatePeriodMillis用于设置更新时间，安卓规定大于30min

  ```xml
  <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
      android:initialLayout="@layout/plant_widget"
      android:minHeight="40dp"
      android:minWidth="40dp"
      android:previewImage="@drawable/launcher_icon"
      android:resizeMode="horizontal|vertical"
      android:updatePeriodMillis="1800000"
      android:widgetCategory="home_screen" />
  ```

- 创建一个类继承自AppWidgetProvider用于打开activity

  - 在静态方法updateAppWidget中
    - 构建PendingIntent，intent指向要打开的activity
    - 构建RemoteView，并传入widget布局
    - 为RemoteView设置点击事件，传入的参数为被点击对象的id以及intent
    - 更新widget
  
  ```java
  static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                              int appWidgetId) {
  
      // Create an Intent to launch MainActivity when clicked
      Intent intent = new Intent(context, MainActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);
      // Construct the RemoteViews object
      RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget);
      // Widgets allow click handlers to only launch pending intents
    views.setOnClickPendingIntent(R.id.widget_plant_image, pendingIntent);
      // Instruct the widget manager to update the widget
      appWidgetManager.updateAppWidget(appWidgetId, views);
  }
  ```
  
  - 在onUpdate方法中，对widget的id数组依次调用上述静态方法
  
- 在Manifest中

  - 为上述WidgetProvider添加一个receiver
  - 并设置intent-filter为APPWIDGET_UPDATE
  - 并设置meta-data指向widget配置文件

  ```xml
  <receiver android:name=".PlantWidgetProvider">
      <intent-filter>
          <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
      </intent-filter>
      <meta-data
          android:name="android.appwidget.provider"
          android:resource="@xml/plant_widget_info" 		/>
  </receiver>
  ```

- API14及更高级别后，android会自动在小部件周围留下外边距

  - 在res/values下创建dimens.xml文件并指定margin大小
  - 在res/values-v14下也创建dimens.xml文件并指定margin大小



##### *2 使用服务

> 点击widget上的水滴时，使用服务为所有植物浇水

- 在布局文件中，添加水滴的imageView

- 创建服务类继承自IntentService（PlantWateringService）

  - 自定义一个Action的tag

  ```java
  public static final String ACTION_WATER_PLANTS = "com.example.mygarden.action.water_plants";
  ```

  - 构造器中，传入子类名称

  ```java
  public PlantWateringService() {
      super("PlantWateringService");
  }
  ```

  - 开启服务的方法
    - 构造intent》为intent设置action》使用context启动服务

  ```java
  public static void startActionWaterPlants(Context context) {
      Intent intent = new Intent(context, PlantWateringService.class);
      intent.setAction(ACTION_WATER_PLANTS);
      context.startService(intent);
  }
  ```

  - 重写onHandleIntent()方法
    - 获取action，如果是自定义的目标action，则执行下一步操作

  ```java
  if (intent != null) {
      final String action = intent.getAction();
      if (ACTION_WATER_PLANTS.equals(action)) {
          // handleActionWaterPlants();
      }
  }
  ```

  - 上述提到的下一步操作：为所有植物浇水
    - 使用ContentResolver更新数据

- 在Manifest中，注册一个service

  ```xml
  <service android:name=".PlantWateringService" />
  ```

- 在自定义的AppWidgetProvider子类中，为水滴添加点击事件

  - 构建intent》为intent设置action为PlantWateringService中的自定义事件》pendingIntent》在remoteView中设置点击事件



##### *3 使用服务更新widget

> 服务中查询快死的（上次浇水时间最远的）植物并显示到widget中

- 在IntentService的子类中
  - 自定义一个新的actionTag：查询并更新照片
  - 创建启用该服务的方法（见 *2）
  - 在onHandleIntent中为上述tag增加入口，并调用如下操作
    - 查询数据库（使用ContentProvider）
    - 获取图片，调用AppWidgetProvider中的方法设置图片
- 在AppWidgetProvider的子类中
  - updateAppWidget方法中，为ImageView设置图片
  - onUpdate()方法中改为使用服务查询并更新照片
- 在其他更新数据库的操作中，同样调用服务查询并更新照片



##### *4 widget之间传递信息

> 将“为所有植物浇水”改为“为一棵植物浇水”，根据植物ID浇水，所以需要传递信息。除此以外：若最近浇水时间不足限定范围，则隐藏widget中的浇水按钮；单击该植物时，启动植物详细页面Activity

- 在IntentService的子类中

  - 自定义一个新的extraTag：传递id数据

  ```java
  public static final String EXTRA_PLANT_ID = "com.example.android.mygarden.extra.PLANT_ID";
  ```

  - 在重写方法onHandleIntent()中，获取intent中的id并传递

- 在AppWidgetProvider的子类中

  - 如果植物id非空，则放入id（使用上述extraTag）



##### *5 自适应的widget

> widget不大时，只展示一棵植物；够大时，展示多科植物（使用GridView）
>
> 与普通layout不同，widget layout不使用adapter，而是使用RemoteViewsFactory和RemoteViewsService来构建视图

- 构造GridView版本的布局，在GridView参数中指出栏数numColumns

- 创建RemoteViewsService的子类

  - 重写方法onGetViewFactory，这会返回一个GridRemoteViewsFactory的实例

- 创建实现RemoteViewsFactory的类（RemoteViewsService的内部类）

  - 构造器：设置context为成员变量，并传入context
  - 重写方法onDataSetChanged：关闭cursor、重新获取
  - 重写方法onDestroy：关闭cursor
  - 重写方法getViewAt
    - 像onBindViewHolder一样使用该方法
    - 因为为每个item都设置一个PendingIntent的代价很高，所以这是不被允许的。应该每个item都创建一个intent，然后在被点击时传递给PendingIntentTemplate构造Pending Intent
    - 设置点击事件但不需要构建pendingIntent（使用views.setOnClickFillInIntent()）

- 在IntentService的子类中

  - 在更新植物的方法中，调用AppWidgetManager的notifyAppWidgetViewDataChanged()方法以更新gridView

- 在AppWidgetProvider的子类中

  - 在静态方法updateAppWidget中，获取widget长和宽以指定不同的视图

  ```java
  {
      Bundle options = appWidgetManager.getAppWidgetOptions(appWidgetId);
      int width = options.getInt (AppWidgetManager.OPTION_APPWIDGET_MIN_WIDTH);
      RemoteViews rv;
      if (width < 300) {
          // ...
      } else {
          // ...
      }
  }
  ```

  - 重写onAppWidgetOptionsChanged方法。每当widget大小发生变化都会调用该方法
    - 更新widget然后调用父类方法
  - 在创建gridView的方法中
    - 给views设置PendingIntentTemplate以及emptyView

- 在Manifest中

  - 注册RemoteViewsService
  - 申请权限BIND_REMOTEVIEWS

  ```xml
  <service
      android:name=".GridWidgetService"
      android:permission="android.permission. BIND_REMOTEVIEWS" />
  ```



[TOC]

## Intent篇

### #Intent

##### *1 Intent类型

- 显示intent：打开其他页面

- 隐式intent：指定了启动活动的action和category，然后交由系统去分析这个intent并找出合适的activity

  - 先检查是否有能够响应的activity

  - 隐式intent填入的是action
  - 配置action、category以及data：在Manifest中，Activity的intent-filter内添加标签
    - 示例中的category是默认类型的，所以不需要调用`.addCategory()`
    - 每个intent只能指定一个action，但能指定多个category
    - data不是必须的，但只有intent中的uri中scheme指向的数据类型相同才能响应（若有uri）

  ```java
  Intent intent = new Intent(/*action*/)
  if(intent.resolveActivity(getPackageManager()) != null){
    	startActivity(intent);
  }
  ```

  ```xml
  <intent-filter>
    <action android:name=" ... "/>
    <category andriod:name="android.intent.category.DEFAULT"/>
    <data android:scheme="http">
  </intent-filter>
  ```

  - 使用隐式intent启动其他程序
    - 指定Intent的action为`Intnet.ACTION_VIEW`，这是一个系统内置动作；同时第二个参数填入URI

- Uri

  - URI：统一资源标识符（Identifier）

  - URL：统一资源定位符（Loactor）（网址）

  - URI完整形式：

    `scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]`

    - []内为可选部分
    - scheme描述指向的资源类型：http、ftp、https、geo（地图）、tel（拨打电话）……
    - authority部分（第一个中括号）表示登录用户名、主机（站点）
    - 资源路径（path）
    - query部分（？q=）以问号开头
    - #指示路径资源将使用的辅助数据

  - 构建地图Uri(scheme为geo时)

    - 初始化Uri.builder
    - `Uri.Builder = new Uri.Builder()`
    - 添加参数
    - `builder.scheme("geo").path("0,0").appendQueryParameter("q","仲恺农业工程学院")`

  - 分享文本、图片等媒体类型

    - 媒体类型字符串由类型、子类型、可选参数组成，如：text/html; chaeset=UTF-8

    - 使用ShareCompat.IntentBuilder来共享

    ```java
    String mimeType = "text/plain";
    String title = "标题;
    ShareCompat.IntentBuilder
        .from(this)
        .setType(mimeType)
        .setChooserTitle(title)
        .setText("分享内容")
        .startChooser();
    ```

##### *2 使用intent传递数据

- 向Intent添加数据，获取数据（先检查）：putExtra()、hasExtra()、getStringExtra()

- 返回数据给上一个活动：

  1. 在activity1中调用：startActivityForResult()
  2. 在activity2中调用：setResult()、finish()
  3. 在activity1中调用：onActivityResult()
  4. 在activity2中补充-当通过按下back键返回时：重写onBackPressed()并在其中执行第二步操作
  
- 向intent添加一个对象

  1. 对象实现Serializable接口或Parcelable接口
  2. 然后就可以直接添加
  3. 获取这个对象

  ```java
  Person person = (Person)getIntent()
    	.getSerializableExtra("person_data")
  ```

  



[TOC]

## 界面篇

### #UI基础

##### *1 组件与布局

- ProgressBar

  - 可以指定不同样式，如：横条。横条样式需要手动增加进度

  ```xml
  <ProgressBar>
    ...
    style="?android:attr/progressBarStyleHorizontal"
    android:max="100"
  </ProgressBar>
  ```

  ```java
  int progress = progressBar.getProgress();
  progress += 10;
  progressBar.setProgress(progress);
  ```

- AlertDialog

  - 在当前页面弹出对话框。须在java中设置

  ```java
  AlertDialog.Builder dialog = new AlertDialog.Builder(this);
  dialog.setTitle("This is a Dialog");
  dialog.setMessage("Something important.");
  dialog.setCancelable(false);
  dialog.setPositiveButton("OK", new DialogInterface.OnClickListener() {
      @Override
      public void onClick(DialogInterface dialog, int which) {
          // ...
      }
  });
  dialog.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
      @Override
      public void onClick(DialogInterface dialog, int which) {
          // ...
      }
  });
  dialog.show();
  ```

- ProgressDialog

  - 展示进度的对话框

  ```java
  ProgressDialog progressDialog = new ProgressDialog(this);
  progressDialog.setTitle("This is a ProgressDialog");
  progressDialog.setMessage("Loading..");
  progressDialog.setCancelable(true);
  progressDialog.show();
  ```

- 百分比布局PercentFrameLayout

- 引入布局

  ```xml
  <include layout="@layout/title"/>
  ```

- 自定义控件（自定义组合的View，继承自某个layout）

  ```java
  LayoutInflater.from(context).inflate(R.layout.title,this);
  ```

##### *2 通知

- NotificationManager》create channel》notify notification

- 通知剖析

  <img src="https://developer.android.google.cn/images/ui/notifications/notification-callouts_2x.png" alt="包含基本详情的通知" style="zoom:50%;" />

  1. 小图标：`setSmallIcon()`
  2. 应用名称，由系统提供
  3. 时间戳，由系统提供，可用`setWhen()`替换或`setShowWhen(false)`隐藏
  4. 大图标：可选，`setLargeIcon()`
  5. 标题：可选，`setContentTitle()`
  6. 文本：可选，`setCotentText()`
  7. 设置优先级（重要程度）：`setPriority()`
  8. 设置通知的点按操作，可选：`setContentIntent()`
  9. 设置通知点击后自动取消，可选：`setAutoCancel(true)`
  10. 添加按钮，可选：`addAction()`，传入的intent指向后台服务

  ```java
  // Create an explicit intent for an Activity in your app
  // 使用notificationCompat是因为通知的api不稳定
  Intent intent = new Intent(this, AlertDetails.class);
  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
  PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
  
  NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID)
          .setSmallIcon(R.drawable.notification_icon)
          .setContentTitle("My notification")
          .setContentText("Hello World!")
          .setPriority(NotificationCompat.PRIORITY_DEFAULT)
          // Set the intent that will fire when the user taps the notification
          .setContentIntent(pendingIntent)
          .setAutoCancel(true);
  ```

- 进阶

  - 设置音频、震动（需要权限）、LED灯、默认效果

  ```java
  .setSound()
  .setVibrate()
  .setLights()
  .setDefaults(NotificationCompat.DEFAULT_ALL);
  ```

  - 设置样式（大（多）文字样式、大照片样式）

- 重要程度

  - Android8.0+：通知的重要程度由通知发布到的渠道的重要程度决定
  - Android7.1-：每条通知的重要程度均由通知的priority决定
  - urgent级别的通知会以pop-on的形式通知

- 创建通知渠道

  ```java
  if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
      // 构建NotificationChannel
      NotificationChannel channel = new NotificationChannel(
              CHANNEL_ID,
              "ChannelName",
              NotificationManager.IMPORTANCE_HIGH);
      channel.setDescription("this is description");
      NotificationManager notificationManager = getSystemService(NotificationManager.class);
      notificationManager.createNotificationChannel(channel);
  }
  ```
  
- 展示通知

  ```java
  NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
  notificationManager.notify(NOTIFICATION_ID,builder.build());
  ```


##### *3 ActionBar操作栏

- 隐藏ActionBar：java实现

  ```java
  ActionBar actionBar = getSupportActionBar();
  if (actionBar != null) {
      actionBar.hide();
  }
  ```

- 为操作栏添加按钮

  ```xml
  <menu xmlns:android="http://schemas.android.com/apk/res/android" >
      <item
          android:id="@+id/action_favorite"
          android:icon="@drawable/ic_favorite_black_48dp"
          android:title="@string/action_favorite"
          app:showAsAction="ifRoom"/>
  </menu>
  ```

- 响应操作栏点击操作：重写方法

  ```java
  @Override
  public boolean onOptionsItemSelected(MenuItem item) {
      switch (item.getItemId()) {
          case R.id.action_settings:
              return true;
          default:
              // If we got here, the user's action was not recognized.
              // Invoke the superclass to handle it.
              return super.onOptionsItemSelected(item);
      }
  }
  ```

##### *4 使用View Binding

- 在module级别的build.gradle中

  ```groovy
  android {
      ...
      buildFeatures {
          viewBinding true
      }
  }
  ```

- 在代码中使用

  - 默认每个布局文件都会生成一个binding类，名字为“布局名+Binding”

  ```java
  private ResultProfileBinding binding;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      binding = ResultProfileBinding.inflate(getLayoutInflater());
      View view = binding.getRoot();
      setContentView(view);
  }
  ```

  - 在fragment中，则应该这样写

  ```java
  private ResultProfileBinding binding;
  
  @Override
  public View onCreateView (LayoutInflater inflater,
                            ViewGroup container,
                            Bundle savedInstanceState) {
      binding = ResultProfileBinding.inflate(inflater, container, false);
      View view = binding.getRoot();
      return view;
  }
  
  @Override
  public void onDestroyView() {
      super.onDestroyView();
      binding = null;
  }
  ```


##### *5 DrawerLayout(md)

- 侧划滑动菜单

- 使用DrawerLayout，第一个子控件为主屏幕中显示内容，第二个子控件为菜单中内容

- 第二个子控件需要设置layout_gravity以表示从哪一侧滑动

  ```xml
  android:layout_gravity="start"
  ```

- 代码中控制开关抽屉

  ```kotlin
  drawerLayout.openDrawer(GravityCompat.START)
  ```

- 使用NavigationView定制滑动菜单页面

  - 先准备一份MenuResourceFile：在\<menu>标签下使用\<group>包裹所有item。group标签设置属性`android:checkableBehavior="single"`表示只能单选
  - 再准备一份布局文件作为滑动菜单顶部(Header)
  - 直接将NavigationView当作DrawerLayout的第二个子控件，并设定menu和header

  ```xml
  app:menu="@menu/nav_menu"
  app:headerLayout="@layout/nav_header"
  ```

  - 代码中设置点击事件和默认项

  ```kotlin
  navView.setCheckedItem(R.id.navCall)
  navView.setNavigationItemSelectedListener {
      drawerLayout.closeDrawers()
      true
  }
  ```

##### *6 Snackbar与CoordinatorLayout

- 允许在提示中增加一个交互按钮

  ```kotlin
  Snackbar.make(v, "Data deleted", Snackbar.LENGTH_SHORT)
      .setAction("Undo"){
          Toast.makeText(this, "Data restored", Toast.LENGTH_SHORT).show()
      }
      .show()
  ```

- CoordinatorLayout是一个加强的FrameLayout，可以监听各种子控件，并自动作出合理的响应以防止控件被遮挡（因为Snackbar由FAB触发，所以被监听到了）

- CoordinatorLayout的使用：只需把FrameLayout替换为CoordinatorLayout即可

- AppBarLayout：解决CoordinatorLayout子控件被遮挡的情况。作为其子部件存在

  - 将Toolbar嵌套在AppBarLayout其中，然后为其余的子部件指定布局行为

  ```xml
  app:layout_behavior="@string/appbar_scrolling_view_behavior"
  ```

  - 自动隐藏toolbar（toolbar随着用户操作决定是否出现）：为toolbar设置属性

  ```xml
  app:layout_scrollFlags="scroll|enterAlways|snap"
  ```

##### *7 下拉刷新

- 添加依赖

  ```groovy
  implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
  ```

- 在RecyclerView外嵌套一层SwipeRefreshLayout，即可拥有下拉刷新功能，但此时下拉后进度条不会隐藏

- 在代码中设置

  ```kotlin
  swipeRefresh.setColorSchemeResources(R.color.teal_200)
  swipeRefresh.setOnRefreshListener {
      thread {
          Thread.sleep(2000)
          runOnUiThread {
              swipeRefresh.isRefreshing = false
          }
      }
  }
  ```

##### *8 CollapsingToolbarLayout

- 只能作为AppBarLayout的直接子布局来使用

  - contentScrim：折叠状态下toolbar的背景色
  - scrollFlags
    - scroll：表示layout会随页面滚动而滚动
    - exitUntilCollapsed：完成折叠后就不再移出屏幕
    - enterAlways：向下划动时，toolbar也向下滑动
    - snap：还未完全隐藏时，自动决定是否隐藏
  - layout_collapseMode
    - pin：表示折叠过程中位置始终不变
    - parallax：表示折叠过程中会产生一定的错位偏移

  ```xml
  <com.google.android.material.appbar.CollapsingToolbarLayout
      android:id="@+id/collapsingToolbar"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:theme="@style/Theme.AppCompat.DayNight.DarkActionBar"
      app:contentScrim="@color/purple_200"
      app:layout_scrollFlags="scroll|exitUntilCollapsed">
  
      <ImageView
          android:id="@+id/fruitImageView"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:scaleType="centerCrop"
          app:layout_collapseMode="parallax"/>
      <androidx.appcompat.widget.Toolbar
          android:id="@+id/toolbar"
          android:layout_width="match_parent"
          android:layout_height="?attr/actionBarSize"
          app:layout_collapseMode="pin"/>
  </com.google.android.material.appbar.CollapsingToolbarLayout>
  ```

- 使用NestedScrollView：能够响应滚动事件的ScrollView

- 使用锚点定位

  - layout_anchor：按钮将会在appBar区域内出现

  ```xml
  <Button
      // ...
      app:layout_anchor="@id/appBar"
      app:layout_anchorGravity="bottom|end"/>
  ```

- 透明的系统状态栏（5.0+）

  - 设置控件的属性：为所有出现在顶部的view及viewgroup都设置此属性

  ```xml
  android:fitsSystemWindows="true"
  ```

  - themes.xml中设置属性

  ```xml
  <style name="FruitActivityTheme" parent="Theme.Firstline_Material_Design">
      <item name="android:statusBarColor">@android:color/transparent</item>
  </style>
  ```

  - Manifest中设置

  ```xml
  <activity
      android:name=".FruitActivity"
      android:exported="true"
      android:theme="@style/FruitActivityTheme">
  ```

  





### #ListView与Adapter

- MVC模式：Model数据模型 - Controller控制器 - View视图
- Adapter家族：
  - BaseAdapter：抽象类，是以下类的父类
    - SimpleAdapter：具有良好的扩展性
    - ArrayAdapter：最简单的adapter，自带了一些布局

##### *1 快速入门

- 在BaseAdapter的子类中getView方法下，为convertView填充View

  - convertView是系统提供的缓存对象

  ```java
  if(convertView == null){
    	convertView = LayoutInflater
        .from(mContext)
        .inflate(R.layout.item,parent,false);
  }
  ```

- ListView设置表头表尾和分割线

  - addHeaderView()：这个方法需放在`setAdapter()`前
  - addFooterView()
  - xml属性
    - divider、dividerHeight分割线及其高度
    - footer\headerDividersEnabled 头或尾处是否设置分割条

- 设置列表从底部往上显示（最后一项贴着底部，默认是第一项贴着顶部）

  - xml属性stackFromBottom

- 设置缓存颜色

  - xml属性cacheColorHint

- 隐藏滑动条

  - setVerticalScrollBarEnabled(true)或xml属性scrollbars

- 使用ViewHolder优化：减少findViewById的多次调用（从n个元素降为屏幕可展示的n个元素）
  - 自定义一个ViewHolder类（不需要继承），然后给所有需要绑定的组件添加成员变量
  - 依然使用convertView来绑定，但是绑定结果交给holder
  - 若convertView为空，则执行绑定操作并为convertView设置setTag
  - 若非空，则直接调用convertView.getTag()
- ListView焦点问题
- checkbox错位问题：加入boolean变量用于保存checkbox状态的值

##### *2 ListView的数据更新

- 当ListView中数据为空时，展示不同的View：setEmptyView()
- 使用notifyDataSetChanged()



### #RecyclerView

- RecyclerView是通过回收已有的结构而不是持续创建新的列表项，所以它可以有效提高应用的时间效率和空间效率

- `Adapter`类从数据源获得数据，并且将数据传递给正在更新其所持视图的 `ViewHolder`

  ![RecyclerView 丁字型关系图](https://mmbiz.qpic.cn/mmbiz_png/icFp8MFO4IKypbqU5o8Uq2gQnR4Rvtm7S3rpiciblP0UiajlUSicUPvVE7IXpg5HuS2uuykxLubOc0PtsU0ActxYITw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### *1 使用RecyclerView

- 在布局文件中：

  - 放入一个带id的recyclerView、设置listitem

  ```xml
  <androidx.recyclerview.widget.RecyclerView
       android:id="@+id/recycler_view"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       app:layoutManager="LinearLayoutManager"
   		 tools:listitem="@layout/todo_item"/>
  ```

- 创建Adapter、ViewHolder

  - 先创建ViewHolder，继承自`RecyclerView.ViewHolder`。该ViewHolder应该与其构造器中的view绑定子view
  - 再创建Adapter，继承自`RecyclerView.Adapter<自定义ViewHolder>`
  - Adapter重写的三个方法
    - onCreateViewHolder()：指定一个view，并返回ViewHolder
    - onBindViewHolder()：将Data和ViewHolder绑定在一起
    - getItemCount()：返回一共有多少项

- 在MainActivity中

  - 给RecyclerView指定一个adapter、layoutManager。如果列表数据总数不变，可以降低消耗

    ```java
    stationRecyclerView.setAdapter(new StationAdapter());
    stationRecyclerView.setLayoutManager(new LinearLayoutManager(this));
    stationRecyclerView.setHasFixedSize(true);
    ```



##### *2 在RecyclerView中使用ListAdapter

> 应对列表数据中的动态变化

- 不使用ListAdapter
  - 指定任务添加的位置`notifyItemInserted()`
  - 删除`notifyItemRemoved()`
  - 重绘整个视图`notifyDataSetChanged()`
- 使用ListAdapter：无需重绘视图，自动会审视变化内容，且自带动画
  - 在 Adapter 类中添加 DiffUtil 对象，并且复写 areItemsTheSame() 和 areContentsTheSame()
  - 将 Adapter 的父类由 RecyclerView.Adapter 改为 ListAdapter，并传入 DiffCall对象
  - ListAdapter 通过 submitList() 方法获取数据，该方法提交了一个列表来与当前列表进行对比并显示。也就是说您无需再重写 getItemCount()，因为 ListAdapter 会负责管理列表
  - 在 Activity 类中，调用 Adapter 的 submitList() 方法并传入数据列表。
  - 在 Adapter 类中，onBindViewHolder() 现在可以使用 getItem() 从数据列表中获取指定位置的元素了
- 让RecyclerView滑动至某一位置
  - `.scrollToPosition()`
- 处理点击事件
  - 使用接口



##### *3 RecyclerView中添加头部或尾部

- 为头部view创建adapter
- 在Activity类中使用ConcatAdapter（构成一个新的adapter），传递给RecyclerView







[TOC]

## 多媒体篇

### #基础

##### *1 调用摄像头

- 先创建File对象用于存储拍照后的照片。这是存放在SD卡的应用专属目录下

  ```java
  File outputImage = new File(getExternalCacheDir(),"output_image.jpg");
  try {
      if(outputImage.exists()){
          outputImage.delete();
      }
      outputImage.createNewFile();
  }catch (IOException e){
      Log.d(TAG, "storage error");
      e.printStackTrace();
  }
  ```

- 提取文件uri

  ```java
  if(Build.VERSION.SDK_INT>=24){
      imageUri = FileProvider.getUriForFile(this, "包名.fileprovider(authorities)",outputImage);
  }else{
      imageUri = Uri.fromFile(outputImage);
  }
  ```

- 声明上述provider

  - file_path.xml文件中，标签external_path下填入共享路径。name可以随便填，path为`.`时代表全部

  ```groovy
  <provider
      android:authorities="com.example.cameraalbumtest.fileprovider"
      android:name="androidx.core.content.FileProvider"
      android:exported="false"
      android:grantUriPermissions="true">
      <meta-data
          android:name="android.support.FILE_PROVIDER_PATHS"
          android:resource="@xml/file_paths"/>
  </provider>
  ```

- 启动摄像头

  ```java
  Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
  intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
  startActivityForResult(intent, TAKE_PHOTO);
  ```

- 在onActivityResult中，获取照片并展示

  ```java
  try {
      Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
      picture.setImageBitmap(bitmap);
  } catch (FileNotFoundException e) {
      Log.d(TAG, "FileNotFound");
      e.printStackTrace();
  }
  ```

##### *2 调用相册

- 需要运行时权限WRITE_EXTERNAL_STORAGE

- 启动相册选择照片

  ```java
  Intent intent = new Intent("android.intent.action.GET_CONTENT");
  intent.setType("image/*");
  startActivityForResult(intent,CHOOSE_PHOTO);
  ```

- 在onActivityResult中处理图片：从intent获得并解析uri

  - 如果Uri是document类型
  - 如果Uri的authority是media类型的

  ```java
  String imagePath = null;
  Uri uri = data.getData();
  if(DocumentsContract.isDocumentUri(this,uri)){
      String docId = DocumentsContract.getDocumentId(uri);
      if("com.android.providers.media.documents".equals(uri.getAuthority())){
          String id = docId.split(":")[1];
          String selection = MediaStore.Images.Media._ID+"="+id;
          imagePath = getImagePath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,selection);
      }else if("com.android.providers.downloads.documents".equals(uri.getAuthority())){
          Uri contentUri = ContentUris.withAppendedId(Uri.parse("content://downloads/public_downloads"),Long.valueOf(docId));
          imagePath = getImagePath(contentUri, null);
      }
  }else if("content".equalsIgnoreCase(uri.getScheme())){
      imagePath = getImagePath(uri, null);
  }else if("file".equalsIgnoreCase(uri.getScheme())){
      imagePath = uri.getPath();
  }
  displayImage(imagePath);
  ```

- 解析uri后，获取指向的照片路径

  ```java
  private String getImagePath(Uri uri, String selection) {
      String path = null;
      Cursor cursor = getContentResolver().query(uri,null,selection,null,null);
      if(cursor!=null){
          if(cursor.moveToFirst()){
              path = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
          }
          cursor.close();
      }
      return path;
  }
  ```

- 获取照片并展示

##### *3 使用MediaPlayer播放多媒体文件

- 指定文件、初始化

  ```java
  File file = new File(Environment.getExternalStorageDirectory(), "music.mp3");
  mediaPlayer.setDataSource(file.getPath());
  mediaPlayer.prepare();
  ```

- 按钮点击事件：播放、暂停、停止

  ```java
  switch (v.getId()){
      case R.id.play:
          if (!mediaPlayer.isPlaying()){
              mediaPlayer.start();
          }
          break;
      case R.id.pause:
          if(mediaPlayer.isPlaying()){
              mediaPlayer.pause();
          }
          break;
      case R.id.stop:
          if(mediaPlayer.isPlaying()){
              mediaPlayer.reset();
              initMediaPlayer();
          }
  }
  ```

- activity被销毁时应该释放

  ```java
  @Override
  protected void onDestroy() {
      super.onDestroy();
      if(mediaPlayer!=null){
          mediaPlayer.stop();
          mediaPlayer.release();
      }
  }
  ```

- 播放视频：使用VideoView，代码与音频文件相似（因为背后仍是MediaPlayer）

  ```java
  // 播放初始化
  videoView.setVideoPath(file.getPath());
  
  // 播放释放
  videoView.suspend();
  ```

  

### #ExoPlayer：猜歌游戏

- 每个多媒体应用都会有两部分组成：媒体播放器和媒体控制器
- Player > MediaSession > MediaController（可能有多个） > UI
- 安卓提供的播放器：
  - MediaPlayer 定制性低，但易用
  - ExoPlayer 是一个不在 Android 框架内的开放源代码项目。支持多种格式。ExoPlayer还可以在布局中忽略MediaSession直接使用
- https://github.com/udacity/AdvancedAndroid_ClassicalMusicQuiz/tree/TMED.01-Exercise-AddExoPlayer

##### *1 应用ExoPlayer

> 布局中，上半界面是一个播放器，且会显示问号，或者作曲家的图片，同时会播放他的音乐；下半部分则是四个按钮以及作曲家的名字，要求用户猜出他的名字，猜出来名字时，将问号改为作曲家的名字。数据集存放于json文件中，每个问题都会有name、id、指向音乐的uri、composer以及albumArtId

- 在布局中

  - 添加SimpleExoPlayerView作为播放器窗口

- 创建一个QuizUtils类，用于获取问题以及问题解

  - generateQuestion()：打乱装有数据id的ArrayList顺序后，返回仅装有四个id的ArrayList
  - get\setHighScore()：从偏好设置中获取\设置最高分
  - get\setCurrentScore()：从偏好设置中获取\设置现在的分数
  - getCorrectAnswerID()：从四个id的ArrayList随机挑一个成为问题
  - userCorrect()：判断用户挑选的答案是否正确
  - endGame()：关闭页面

- 创建一个Sample类，负责从json文件中获取数据集并提取用于指向这些数据的id，然后将所有id放入ArrayList中

  - getComposerArtBySampleId()：根据albumArtId获取背景图
  - getSampleById()：根据id找到这个问题
  - getAllSampleIds()：返回装着所有id的ArrayList
  - readEntry()：解析一个问题，并将数据放入新的Sample中（name、id等），返回
  - readJsonFile()：为JsonReader返回实例

- 在Activity中

  - 使用findViewById绑定上述view

  - 为上述view设置播放器显示的图片为问号

    ```java
    mPlayerView. setDefaultArtwork (BitmapFactory. decodeResource (getResources(), R.drawable.question_mark));
    ```

  - 获取数据集、生成问题

  - 初始化播放器、播放音乐

    - 初始化SimpleExoPlayer
    - 为SimpleExoPlayerView绑定SimpleExoPlayer
    - 绑定音乐

  ```java
  private void initializePlayer(Uri mediaUri) {
          if (mExoPlayer == null) {
              // Create an instance of the ExoPlayer.
              TrackSelector trackSelector = new DefaultTrackSelector();
              LoadControl loadControl = new DefaultLoadControl();
              mExoPlayer = ExoPlayerFactory.newSimpleInstance(this, trackSelector, loadControl);
              mPlayerView.setPlayer(mExoPlayer);
              // Prepare the MediaSource.
              String userAgent = Util.getUserAgent(this, "ClassicalMusicQuiz");
              MediaSource mediaSource = new ExtractorMediaSource(mediaUri, new DefaultDataSourceFactory(
                      this, userAgent), new DefaultExtractorsFactory(), null, null);
              mExoPlayer.prepare(mediaSource);
              mExoPlayer.setPlayWhenReady(true);
          }
      }
  ```

  - 初始化按钮集、设置点击事件

    - 展示正确答案：为正确答案添加绿色滤镜，错误答案添加红色滤镜

    ```java
    if (buttonSampleID == mAnswerSampleID) {
                    mButtons[i].getBackground().setColorFilter(ContextCompat.getColor
                                    (this, android.R.color.holo_green_light),
                            PorterDuff.Mode.MULTIPLY);
                    mButtons[i].setTextColor(Color.WHITE);
    }else{ // ... } 
    ```

    - 获取用户点击的按钮是否为正确答案（从onClick()传入的参数View中检查id和哪个一样）
    - 移除这个数据，因为已经问过了
    - 等待几秒使得用户能够看清这个答案，过几秒后用intent重新载入该页面

    ```java
    final Handler handler = new Handler();
    handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    // TODO (9): Stop the playback when you go to the next question.
                  mExoPlayer.stop();
                    Intent nextQuestionIntent = new Intent(QuizActivity.this, QuizActivity.class);
                    nextQuestionIntent.putExtra(REMAINING_SONGS_KEY, mRemainingSampleIDs);
                    finish();
                    startActivity(nextQuestionIntent);
                }
            }, 1);
    ```

  - 在onDestroy()中，暂停、释放SimpleExoPlayer
  
    ```java
    mExoPlayer.stop();
    mExoPlayer.release();
    mExoPlayer = null;
    ```

##### *2 自定义SimpleExoPlayerView外观

> 自己创建一个layout文件，只要名字一样，便可覆盖Exo默认外观。这里修改了exo_playback_control_view

##### *3 ExoPlayer事件监听

> 获知ExoPlayer状态

- 在Activity中

  - 实现`View.OnClickListener`，并为SimpleExoPlayer设置监听器

    ```java
    mExoPlayer.addListener(this);
    ```

  - 重写方法后，在onPlayerStateChanged()中获取状态

    ```java
    if((playbackState == ExoPlayer.STATE_READY) && playWhenReady){
        Log.d(TAG, "onPlayerStateChanged: PLAYING");
    } else if((playbackState == ExoPlayer.STATE_READY)){
        Log.d(TAG, "onPlayerStateChanged: PAUSED");
    }
    ```

##### *4 添加MediaSession

> 尽管SimpleExoPlayerView可以无需MediaSession控制播放，但是当我们想用外部事件控制播放时，如：通知、耳机，这就需要我们重新回到MediaSession。
>
> 所以，这里我们要①：构建MediaSession ②：将ExoPlayer的事件监听状况告知MediaSession

- 在Activity中

  - 创建一个mySessionCallback类，继承自MediaSessionCompat.Callbaack
    - 这个类用于控制播放器状态（外部按下后执行）
    - 重写方法：暂停、播放、下一首
  - 初始化MediaSession
    - 创建一个MediaSessionCompact对象，这就是提到的MediaSession
    - 设置标志表示需要的功能
    - 设置媒体按钮接收组件（用于打开Activity，这里不需要，设置null）
    - 设置可用操作和初始状态。这里用到了Play、Pause、Play/Pause、SkipToPrevious
    - 设置回调，使用mySessionCallback
    - 设置开始、终止会话（onCreate()/onDestroy()）

  ```java
  // Create a MediaSessionCompat.
  mMediaSession = new MediaSessionCompat(this, TAG);
  
  // Enable callbacks from MediaButtons and TransportControls.
  mMediaSession.setFlags(
          MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS |
                  MediaSessionCompat.FLAG_HANDLES_TRANSPORT_CONTROLS);
  
  // Do not let MediaButtons restart the player when the app is not visible.
  mMediaSession.setMediaButtonReceiver(null);
  
  // Set an initial PlaybackState with ACTION_PLAY, so media buttons can start the player.
  mStateBuilder = new PlaybackStateCompat.Builder()
          .setActions(
                  PlaybackStateCompat.ACTION_PLAY |
                          PlaybackStateCompat.ACTION_PAUSE |
                          PlaybackStateCompat.ACTION_SKIP_TO_PREVIOUS |
                          PlaybackStateCompat.ACTION_PLAY_PAUSE);
  
  mMediaSession.setPlaybackState(mStateBuilder.build());
  
  
  // MySessionCallback has methods that handle callbacks from a media controller.
  mMediaSession.setCallback(new MySessionCallback());
  
  // Start the Media Session since the activity is active.
  mMediaSession.setActive(true);
  ```
  - 在上一步中的事件监听中，当发生状态变化时，通知PlaybackStateCompat.builder的实例


##### *5 创建媒体样式的通知

> 为了测试外部事件是否触发播放器状态改变，所以设置了通知

- 在Activity中，展示通知。结束播放时则取消通知

- 还需要添加按钮接收器。在Activity中创建静态内部类MediaReceiver继承自BroadcastReceiver

- 在Manifest中注册该receiver

  ```xml
  <receiver android:name=".QuizActivity$MediaReceiver">
      <intent-filter>
          <action android:name="android.intent.action.MEDIA_BUTTON" />
      </intent-filter>
  </receiver>
  ```

##### *6 额外知识

- 音频焦点：如果有其他语言通知（如：导航）出现时自动避让
- Noisy Intent：耳机拔出来时通知播放
- 音频流：确保在暂停时，用户调整音量时调整的是你app的播放音量（而不是闹钟音量 etc.）
- ExoPlayer额外功能
  - 加载字幕
  - 循环播放视频



[TOC]

## 任务篇

### #服务

- 即使程序被切换到后台或者用户打开了其他app，服务仍能够正常运行
- 服务并不是运行在一个独立的进程当中，而是依赖创建服务所在的应用程序进程。当app的进程被杀掉时，服务也将结束
- 服务不会自动开启线程，默认在主线程当中，有可能出现主线程被阻塞的情况

##### *1 安卓多线程

- 子线程不允许执行ui操作。子线程可以借用runOnUIThread()或handler来执行ui操作

- Android异步消息处理机制

  - Message：在线程之间传递消息
    - what字段、arg1、arg2字段、obj字段传递对象
  - Handler：用于发送和处理消息
    - sendMessage()：发送
    - handleMessage()：处理
  - MessageQueue：存放所有通过Handler发送的消息，等待被处理
    - 每个线程中只会有一个MessageQueue对象
  - Looper：MessageQueue的管家，当发现队列中有消息时，取出并传递给Handler的handleMessage()方法中
    - 每个线程中只会有一个Looper对象

- 实践

  - 发送消息

  ```java
  public static final int UPDATE_TEXT = 1;
  changeText.setOnClickListener(v->{
      new Thread(()->{
          Message message = new Message();
          message.what = UPDATE_TEXT;
          handler.sendMessage(message);
      }).start();
  });
  ```

  - 主线程中创建handler

  ```java
  private Handler handler = new Handler(){
      @Override
      public void handleMessage(@NonNull Message msg) {
          switch (msg.what){
              case UPDATE_TEXT:
                  text.setText("Nice to meet you");
                  break;
          }
      }
  };
  ```

- AsyncTask

  - 指定3个泛型参数
    - Params：在执行AsyncTask时需要传入的参数，可用于在后台任务中使用
    - Progress：后台执行任务时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位
    - Result：当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型
  - onPreExecute()：后台任务执行前
  - doInBackground()：后台任务，于子线程中执行。不能进行ui操作
  - onProgressUpdate()：更新ui界面。后台任务更新ui的方式
  - onPostExecute()：doInBackground()后台任务return后调用，会接收到return的数据

  ```java
  class MyTask extends AsyncTask<Void,Integer,Integer> {}
  ```

##### *2 使用服务

- Android四大组件都需要在Manifest中声明

- 构建service：创建Service的子类

  ```java
  public class MyService extends Service {
      private static final String TAG = "ServiceTest";
      public MyService() {
      }
  
      @Override
      public void onCreate() {
          super.onCreate();
          Log.d(TAG, "onCreate: ");
      }
  
      @Override
      public int onStartCommand(Intent intent, int flags, int startId) {
          Log.d(TAG, "onStartCommand: ");
          return super.onStartCommand(intent, flags, startId);
      }
  
      @Override
      public void onDestroy() {
          Log.d(TAG, "onDestroy: ");
          super.onDestroy();
      }
  
      @Override
      public IBinder onBind(Intent intent) {
          // ...
      }
  }
  ```

- 服务的启动与关闭：使用Intent指向Service子类

  ```java
  Intent intent = new Intent(this, MyService.class);
  // 服务的启动
  startService(intent);
  // 服务的关闭
  stopService(intent);
  ```

- 让服务自动结束：在MyService的任意位置调用`stopSelf()`方法

- `onCreate()`与`onStartCommand()`或`onBind()`：service未创建或绑定时，两者都会被调用。创建后，启动服务时只会执行后者

- 活动和服务进行通信：使用Binder

  - 在Serivce子类中

    - 创建Binder子类
    - 指定Binder对象，并在`onBind()`方法中返回

  - 在Activity子类中

    - 指定Binder对象
    - 指定ServiceConnection，并在`onServiceConnected()`方法中指定将Binder对象指向service。然后就可以调用Binder内的方法了

    ```java
    private MyService.DownloadBinder downloadBinder;
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder)service;
            downloadBinder.startDownload();
            downloadBinder.getProgress();
        }
    
        @Override
        public void onServiceDisconnected(ComponentName name) {}
    };
    ```

    - 绑定或取消绑定服务

    ```java
    // 绑定
    Intent intent = new Intent(this, MyService.class);
    bindService(bindIntent, connection, BIND_AUTO_CREATE);
    // 取消绑定：不需要intent
    unbindService(connection);
    ```

- 服务的生命周期

##### *3 前台服务与IntentService

- 使用前台服务可以提高服务的系统优先级

- 前台服务要求有一个正在运行的通知。在`onCreate()`中创建通知

  ```java
  NotificationChannel channel = new NotificationChannel(CHANNEL_ID, "Channel", NotificationManager.IMPORTANCE_HIGH);
  channel.setDescription("description");
  getSystemService(NotificationManager.class).createNotificationChannel(channel);
  
  Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
          .setContentTitle("title")
          .setContentText("text")
          .setShowWhen(false)
          .setSmallIcon(R.drawable.ic_launcher)
          .build();
  startForeground(1,notification);
  ```

- 使用IntentService可以简单创建一个异步的、会自动停止的服务

  ```java
  Intent intent= new Intent(this, MyIntentService.class);
  startService(intent);
  ```

- IntentService同样需要在Manifest中声明



### #广播

- 广播分为标准广播和有序广播。仅有序广播可被截断
- 广播接收器分为静态注册（使用Manifest）和动态注册（代码中注册）

##### *1 动态广播的接收

> 使得app可以感知网络变化

- 创建BroadcastReceiver的子类

  - 重写onReceive方法。这个方法会在接收到广播后被调用
  - 在这个方法中，使用系统服务获取网络信息（这里不是广播内容）。获取像网络信息这些较敏感数据需要在Manifest中声明权限

  ```java
  ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
  NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
  if(networkInfo!=null && networkInfo.isAvailable()){
     // ...
  }
  ```

- 在activity中

  - 设置IntentFilter使得能够选择并接收对应的广播
  - 动态注册、注销BroadcastReceiver。分别在onCreate和onDestroy中完成

  ```java
  IntentFilter filter = new IntentFilter();
  filter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
  NetworkChangeReceiver receiver = new NetworkChangeReceiver();
  registerReceiver(receiver, filter);
  ```

- 在Manifest文件中添加权限

  ```groovy
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  ```



##### *2 静态广播的接收

> 接收开机完成通知（貌似已经不行了）

- 静态注册时，可以选择直接创建一个BroadcastReceiver子类，studio会自动添加模板代码，包括：

  - 在Manifest中自动注册。enabled表示启用，exported表示可接受本程序以外的广播
  - 直接创建BroadcastReceiver的子类

  ```groovy
  <receiver
      android:name=".BootCompleteReceiver"
      android:enabled="true"
      android:exported="true"/>
  ```

- 在Manifest中设置intent-filter、声明权限

  ```groovy
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
  ...
  <intent-filter>
      <action android:name="android.intent.action.BOOT_COMPLETED"/>
  </intent-filter>
  ```
  
- 新版系统配置静态广播


##### *3 自定义广播、有序广播与本地广播

- 点击按钮发送广播：使用Intent发送

  ```java
  Intent intent = new Intent("com.example.mytest.MyBROADCAST");
  
  // 安卓8.0+： 所有广播发送须带包名
  intent.setPackage("com.example.broadcastbestpractice");
  
  sendBroadcast(intent);
  ```

- 使用静态广播接收，方法同上

  ```xml
  <intent-filter>
      <action android:name="com.example.mybroadcast.MyBROADCAST"/>
  </intent-filter>
  ```

- 发送有序广播，只需改一行

  ```java
  sendOrderedBroadcast(intent, null);
  ```

- 收者有序，也只需增一句

  ```xml
  <intent-filter android:priority="100">
  </intent-filter>
  ```

- 截断广播，也只需一句

  ```java
  abortBroadcast();
  ```

- 本地广播的发送与接收

  - 本地广播不能使用静态注册

  ```java
  private LocalBroadcastManager localBroadcastManager;
  
  // 获取实例
  localBroadcastManager = LocalBroadcastManager.getInstance(this);
  
  // 发送广播(8.0+需设置包名)
  localBroadcastManager.sendBroadcast(intent);
  
  // 动态注册(也需要注销)
  intentFilter = new IntentFilter();
  intentFilter.addAction("com.example.broadcast.LOCAL_BROADCAST");
  localReceiver = new LocalReceiver();
  localBroadcastManager.registerReceiver(localReceiver,intentFilter);
  ```



[TOC]

## 应用数据和文件篇

### #基础

##### *1 文件存储

- 存储文件

  - 文件名不可以包含路径，所有的文件都默认存储到`/data/data/<packeage name>/files`目录下
  - 两种文件的操作模式
    - MODE_PRIVATE：当存在同名文件时，覆盖
    - MODE_APPEND：当存在同名文件时，追加内容

  ```java
  public void save(String data){
    	// 获取路径
    	// 配置输送
    	// 输送数据
      FileOutputStream out;
      BufferedWriter writer = null;
      try {
          out = openFileOutput("data", Context.MODE_PRIVATE);
          writer = new BufferedWriter(new OutputStreamWriter(out));
          writer.write(data);
      }catch (IOException e){
          e.printStackTrace();
      }finally {
          try {
              if(writer!=null)writer.close();
          }catch (IOException e){
              e.printStackTrace();
          }
      }
  }
  ```

- 读取数据

  ```java
  public String load(){
      FileInputStream in = null;
      BufferedReader reader = null;
      StringBuilder content = new StringBuilder();
      try{
          in = openFileInput("data");
          reader = new BufferedReader(new InputStreamReader(in));
          String line = "";
          while((line = reader.readLine())!=null){
              content.append(line);
          }
      }catch (IOException e){
          e.printStackTrace();
      }finally {
          if (reader!=null){
              try {
                  reader.close();
              }catch (IOException e){
                  e.printStackTrace();
              }
          }
      }
      return content.toString();
  }
  ```

  

##### *2 SharedPreferences存储

- 使用键值对保存数据，存放在`/data/data/<packeage name>/shared_prefs/`目录下

- 三种获取SharedPreference对象的方法

  - Context类中的`getSharedPreference()`，第一个参数是文件名（仅该方法需要自己命名），第二个参数目前只能填MODE_PRIVATE
  - Activity类中的`getPreferences()`方法。会将当前Activity的类名作为文件名
  - PreferenceManager类中的`getDefaultSharedPreferences()`方法。会将包名作为文件名

- 存放SharedPreferences数据

  ```java
  SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
  editor.putString("name", "Tom");
  editor.putInt("age",28);
  editor.apply();
  ```

- 获取SharedPreferences数据

  ```java
  SharedPreferences pref = getSharedPreferences("data", MODE_PRIVATE);
  String name = pref.getString("name", "");
  int age = pref.getInt("age", 0);
  ```


##### *3 SQLite存储

- 数据库文件存放在`data/data/<package name>/databases`目录下
- 直接使用SQL操作数据库数据
  - 除了查询数据使用rawQuery() ，其余使用execSQL()



### #Content Provider

- Content Provider主要用于在不同的应用程序之间实现数据共享的功能，同时能够保证被访数据的安全性

##### *1 运行时权限

- 普通权限和危险权限（安卓6.0+）
  - 普通权限只需在Manifest中申请
  - 危险权限还需要用户手动点击授权。这些权限被分了权限组，其中一个权限得到授权后，其他权限也得到了授权
  
- 调出权限授权页面

  ```java
  if(ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE)!= PackageManager.PERMISSION_GRANTED){
      ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CALL_PHONE},1);
  }else{
      // your code
  }
  ```

- 获知是否授权成功

  ```java
  @Override
  public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
      switch (requestCode){
          case 1:
              if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                  call();
              }else{
                  Toast.makeText(this, "no permission", Toast.LENGTH_SHORT).show();
              }
              break;
      }
  }
  ```

##### *2 使用ContentResolver访问

- ContentResolver提供了一系列的方法用于对数据进行CRUD操作

- 使用Uri表示表名参数。Uri由authority和path组成，authority用于区分不同程序，path用于区分不同的表

  - 在URI的后面加一个id表示访问对应id的数据
  - 通配符：*，#都表示全部
  
  ```java
  content://com.example.app.provider/table1
  content://com.example.app.provider/table2
  ```
  
  ```java
  content://com.example.app.provider/table/1
  content://com.example.app.provider/*
  content://com.example.app.provider/table/#
  ```
  
- 解析URI

  ```java
  Uri uri = uri.parse("content://com.ex../table2");
  Cursor cursor = getContentResolver().query(
    	uri,
    	projection,
    	// ...
  )
  ```

##### *3 创建ContentProvider

- 使用官方模板
  - 创建类继承自ContentProvider，并实现方法
    - `getType()`：根据传入的内容URI来返回相应的MIME类型
    
    - MIME类型：以路径结尾填dir，以id结尾填item
    
      ```java
      vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book
      ```
    
  - 使用UriMatcher匹配内容，以获知是要调用什么数据
  
  ```java
  // 预设
  static {
      uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
      uriMatcher.addURI(AUTHORITY, "book", BOOK_DIR);
    	// ...
  }
  
  // 检查
  switch (uriMatcher.match(uri)){
  		case BOOK_DIR:
      		return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book";
      // ...
  }
  ```
  
  - 模板会自动在Manifest中注册
  
  ```groovy
  <provider
    android:name=".DatabaseProvider"
    android:authorities="com.example.databasetest.provider"
    android:enabled="true"
    android:exported="true"></provider>
  ```
  
- 安卓10+引入了分区存储的概念（沙盒存储机制），以防止应用读取其他应用数据。为了使得provider能提供数据，还需要Manifest的application标签中添加：

  ` android:requestLegacyExternalStorage="true"`

- 安卓11+还需要在Manifest中声名要访问的软件包

- 将ContentProvider放在独立进程中时，需要添加permission





[TOC]

## 位置信息篇

### #ShushMe：根据位置决定是否静音的app

- 使用GPS（Global Position System）获取的位置信息准确度高，但是耗电量大；而使用网络（Wifi和基地台）确定位置准确度不高，但是耗电量低，还能更快确定设备所在国家或地区
- 这里将使用Google Play Services（也可以叫GPS）获取地理位置信息。通过使用其API，可以获取最后已知信息、接收位置更新、纬度转地址等
- 某些Google Play Services API需要先创建一个与Google Play Services连接的客户端，并使用该连接与API通信。需要先获取密匙、创建客户端
- Google Maps API禁止保存地点信息（小于30天），地点id除外
- 安卓会跟踪设备相对于任何注册的地理围栏（Geofence）的位置。每台设备的地理围栏上限为100个，每个围栏都可以设置时限
- 使用Geofence的优势在于它被优化过（更多地使用了网络，有时使用GPS），减少了不断使用GPS查询的机会

##### *1 连接Google API Client

> 以调用API

- 在Manifest.xml中

  - 使用meta-data标签放入密匙

  ```xml
  <meta-data		  		  
  android:name="com.google.android.geo.API_KEY"
  android:value="insert-your-api-key-here" />
  ```

  - 添加用户权限（uses-permission）
    - android.permission.INTERNET
    - android.permission.ACCESS_FINE_LOCATION
  
  ```xml
  <uses-permission android:name="..." />
  ```
  
- 在build.gradle(app)中

  - 添加依赖
    - com.google.android.gms:play-services-places:9.8.0
    - com.google.android.gms:play-services-location:9.8.0

- 在布局中

  > 放置一个checkbox用于告知用户是否已经给了定位权限，若已给，则不可按

  - 放置checkbox

- 在MainActivity.java中

  - 实现ConnectionCallbacks、OnConnectionFailedListener
  - 创建一个GoogleApiClient

  ```java
  GoogleApiClient client = new GoogleApiClient.Builder(this)
          .addConnectionCallbacks(this)
          .addOnConnectionFailedListener(this)
          .addApi(LocationServices.API)
          .addApi(Places.GEO_DATA_API)
          .enableAutoManage(this, this)
          .build();
  ```

  - 重写以下三种方法，仅用于在console中输出现在的状态（logi）
    - onConnected()
    - onConnectionSuspended()
    - onConnectionFailed()
  - 重写onResume()。检查是否得到了定位权限，并据此设置checkbox状态

  ```java
  if (ActivityCompat.checkSelfPermission( MainActivity.this,
  android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
      locationPermissions.setChecked(false);
  } else {
      locationPermissions.setChecked(true);
      locationPermissions.setEnabled(false);
  }
  ```

  - checkbox响应点击事件：申请权限

  ```java
  public void onClicked(View view) {
    ActivityCompat.requestPermissions(MainActivity.this, new String[]{android.Manifest.permission.ACCESS_FINE_LOCATION}, PERMISSIONS_REQUEST_FINE_LOCATION);
  }
  ```


##### *2 打开地点选择器

- 在布局文件中

  - 添加一个按钮，用于选择位置

- 在MainActivity中

  - 添加位置button响应点击事件

    - 先检查是否拥有权限，若无，发出toast并返回
    - 构造PlacePicker的intent并打开activity。因为用户手机可能没有GooglePlayServices，所以这部分应该使用try-catch块。

    ```java
    try {
        PlacePicker.IntentBuilder builder = new PlacePicker.IntentBuilder();
        Intent i = builder.build(this);
        startActivityForResult(i, PLACE_PICKER_REQUEST);
    }catch(Exception e){
      // ...
    }
    ```

    - 使用`startActivityForResult()`以获取添加结果

  - 重写onActivityResult()以处理获得的结果。若添加成功，则应该记录该地点，提取数据并放到数据库中

  ```java
  Place place = PlacePicker.getPlace(this, data);
  if (place == null) {
      Log.i(TAG, "No place selected");
      return;
  }
  
  // Extract the place information from the API
  String placeName = place.getName().toString();
  String placeAddress = place.getAddress().toString();
  String placeID = place.getId();
  ```

[TOC]

##### *3 根据id获取地点

> 一是因为Google地图服务条款要求不得保存地点信息，只能保存id，二是因为id可能会变。所以能联网时就需要更新，使用id获取地点信息。除此之外，Google地图服务条款还要求添加"powered by google"

- 在MainActivity中
  - 每当连接上服务器时，在onConnected()中更新/获取位置信息。获取数据后，刷新adapter
- 在布局中
  - 添加一个名为"powered_by_google"的drawable，并使用src添加到widget中



##### *4 使用Geofence》触发静音事件

- 构造一个新类，用于管理geofence的存储
  - 写一个获取GeofenceingRequest、GeofencePendingIntent的方法
  - 写一个更新/添加方法
  - 写一个注册、注销所有geofence的方法
    - 通过使用`LocationServices.GeofencingApi.*`实现
  
- 构造一个新类，继承自BroadcastReceiver

  - 在重写方法onReceive中：设置静音、发送消息

- 在MainActivity中
  - 在连接上服务器并查询结果后(onResult())，更新一个Geofence列表
  - 请求响铃模式（安卓N及之后）权限（由button点击事件引起），在`onResume`中初始化这个button状态

  ```java
  Intent i = new Intent(android.provider.Settings.ACTION_NOTIFICATION_POLICY_ACCESS_SETTINGS);
  startActivity(intent);
  ```

- 在Manifest中
  - 添加一个receiver

```
测试位置应用》》
可以在模拟器中设置位置甚至是线路，但是因为是仿真的，缺乏网络提供的地理位置，所以地理位置未必会及时。打开地图类app可以使得手机强制实时更新地理位置
```



## 基于网络篇

### #基础

##### *1 WebView的使用

- 在app内展示网页

- 在布局文件中添加标签WebView

- 在代码中配置

  - getSettings()：设置一些浏览器属性
  - setWebViewClient()：让网页在当前页面展示
  - loarUrl()：网址

  ```java
  WebView webView = findViewById(R.id.web_view);
  webView.getSettings().setJavaScriptEnabled(true);
  webView.setWebViewClient(new WebViewClient());
  webView.loadUrl("http://www.baidu.com");
  ```

- 声明权限INTERNET

- application标签下添加属性设置

  ```xml
  android:usesCleartextTraffic="true"
  ```

##### *2 使用HTTP协议访问网络

- 协议流程：客户端发送HTTP请求》服务器返回数据》客户端对数据进行解析处理

- 使用HttpUrlConnection或OkHttp

- 配置HttpUrlConnection

  - 设置HTTP请求所使用的方法：GET和POST
  - 设置连接超时：setConnectionTimeout()
  - 设置读写超时：setReadTimeout()

  ```java
  HttpURLConnection connection = null;
  BufferedReader reader = null;
  try {
      URL url = new URL("http://www.baidu.com");
      connection = (HttpURLConnection) url.openConnection();
      connection.setRequestMethod("GET");
      connection.setConnectTimeout(8000);
      connection.setReadTimeout(8000);
      InputStream in = connection.getInputStream();
  
      reader = new BufferedReader(new InputStreamReader(in));
      StringBuilder response = new StringBuilder();
      String line;
      while((line=reader.readLine())!=null){
          response.append(line);
      }
    
    	// 自己写的展示方法
      showResponse(response.toString()); 
  }catch (Exception e){
      e.printStackTrace();
  }finally {
      if(reader!=null){
          try {
              reader.close();
          }catch (IOException e){
              e.printStackTrace();
          }
      }
      if (connection!=null){
          connection.disconnect();
      }
  }
  ```

- 配置OkHttp

  - 需添加依赖`implementation 'com.squareup.okhttp3:okhttp:3.4.1'`

  ```java
  try {
      OkHttpClient client = new OkHttpClient();
      Request request = new Request.Builder()
              .url("http://www.baidu.com")
              .build();
      Response response = client.newCall(request).execute();
      String responseData = response.body().string();
    
    	// 自己写的展示方法
      showResponse(responseData);
  }catch (Exception e){
      e.printStackTrace();
  }
  ```

- 初步使用线程：在子线程中使用http请求

  - 在runOnUiThread()下执行ui操作

  ```java
  runOnUiThread(new Runnable() {
      @Override
      public void run() {
          responseText.setText(response);
      }
  });
  ```

- 声明权限INTERNET

##### *3 解析XML、JSON

- 使用Pull解析XML

  ```java
  try {
      XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
      XmlPullParser xmlPullParser = factory.newPullParser();
      xmlPullParser.setInput(new StringReader(xmlData));
      int eventType = xmlPullParser.getEventType();
      String id = "";
      String name = "";
      String version = "";
      while(eventType != XmlPullParser.END_DOCUMENT){
          String nodeName = xmlPullParser.getName();
          switch (eventType){
              case XmlPullParser.START_TAG:
                  if ("id".equals(nodeName)){
                      id = xmlPullParser.nextText();
                  }else if("name".equals(nodeName)){
                      name = xmlPullParser.nextText();
                  }else if("version".equals(nodeName)){
                      version = xmlPullParser.nextText();
                  }
                  break;
              case XmlPullParser.END_TAG:
                  if("app".equals(nodeName)){
                      // ...
                  }
              default:
                  break;
          }
          eventType = xmlPullParser.next();
      }
  }
  ```

- 使用SAX解析xml

  - 创建DefaultHandler的子类

  ```java
  public class ContentHandler extends DefaultHandler {
      private String nodeName;
      private StringBuilder id;
      private StringBuilder name;
      private StringBuilder version;
  
      @Override
      public void startDocument() throws SAXException {
          id = new StringBuilder();
          name = new StringBuilder();
          version = new StringBuilder();
      }
  
      @Override
      public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
          nodeName = localName;
      }
  
      @Override
      public void characters(char[] ch, int start, int length) throws SAXException {
          if ("id".equals(nodeName)){
              id.append(ch, start, length);
          }else if("name".equals(nodeName)){
              name.append(ch, start, length);
          }else if("version".equals(nodeName)){
              version.append(ch, start, length);
          }
      }
  
      @Override
      public void endElement(String uri, String localName, String qName) throws SAXException {
          if("app".equals(localName)){
              Log.d(TAG, "id: "+id.toString().trim());
              Log.d(TAG, "name: "+name.toString().trim());
              Log.d(TAG, "version: "+version.toString().trim());
              // clear
              id.setLength(0);
              name.setLength(0);
              version.setLength(0);
          }
      }
  
      @Override
      public void endDocument() throws SAXException {
          super.endDocument();
      }
  }
  ```

  - 初始化SAX

  ```java
  try {
      SAXParserFactory factory = SAXParserFactory.newInstance();
      XMLReader xmlReader = factory.newSAXParser().getXMLReader();
      ContentHandler handler = new ContentHandler();
      xmlReader.setContentHandler(handler);
      xmlReader.parse(new InputSource(new StringReader(xmlData)));
  }
  ```

- json格式数据

  ```json
  [{"id":"5", "version":"5.5", "name":"Clash of Clans"},
  {"id":"6", "version":"7.0", "name":"Boom Beach"},
  {"id":"7", "version":"3.5", "name":"Clash Royale"}]
  ```

- 使用JSONObject解析json

  ```java
  try {
      JSONArray jsonArray = new JSONArray(jsonData);
      for (int i = 0; i < jsonArray.length(); i++) {
          JSONObject jsonObject = jsonArray.getJSONObject(i);
          String id = jsonObject.getString("id");
          String name = jsonObject.getString("name");
          String version = jsonObject.getString("version");
          Log.d(TAG, "id: "+id);
          Log.d(TAG, "name: "+name);
          Log.d(TAG, "version: "+version);
      }
  }catch (Exception e){
      e.printStackTrace();
  }
  ```

- 使用GSON解析json

  ```groovy
  implementation 'com.google.code.gson:gson:2.8.6'
  ```

  - 先指定类以放置数据（这里是app类）。变量名需与json字段同名
  - 配置GSON

  ```java
  Gson gson = new Gson();
  List<App> appList = gson.fromJson(jsonData, new TypeToken<List<App>>(){}.getType());
  for(App app: appList){
      Log.d(TAG, "id: "+app.getId());
      Log.d(TAG, "name: "+app.getName());
      Log.d(TAG, "version: "+app.getVersion());
  }
  ```

  



[TOC]

## 测试篇

### #Espresso框架：测试ui

- UI测试是一种设备化测试，需要在真机或模拟器上运行。设备化测试始终位于src/androidTest/java/...下

##### *1 测试点击事件

> 测试一次点击事件后（这里是修改咖啡订单的数量），view中产生的数据是否符合预期

- 在build.gradle中添加依赖性

  ```groovy
  androidTestCompile 'com.android.support:support-annotations:与已存在的support-annotation相同版本号'
  androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
  ```

- 在src/androidTest/java/...下创建类用于测试(OrderActivityBasicTest)

  - 类的命名应该能让别人知道是测试类、测试的是什么
  - 在类上面添加注解`@RunWith(AndroidJUnit4.class)`
  - 在类里面创建一个rule，用于提供测试工具

  ```java
  @Rule
  public ActivityTestRule<OrderActivity> mActivityTestRule = new ActivityTestRule<>(OrderActivity.class);
  ```

  - 在类中创建测试方法
    - 使用onCheck()指定view
    - 测试中，点击按钮(perform)
    - 使用matches断言是否相等

  ```java
  @Test
  public void clickDecrementButton_ChangesQuantityAndCost() {
      // Click on decrement button
      onView((withId(R.id.decrement_button)))
              .perform(click());
  
      // Verify that the decrement button decreases the quantity by 1
      onView(withId(R.id.quantity_text_view)).check(matches(withText("0")));
  
      // Verify that the increment button also increases the total cost to $5.00
      onView(withId(R.id.cost_text_view)).check(matches(withText("$0.00")));
  
  }
  ```

- 最后，运行该测试：右键该测试类，选择Run



##### *2 测试adapterView

>虽然 `onView()` 可以处理我们的 UI 中的大部分视图，但是 Espresso 在调用 AdapterView 小部件时，需要不同的方法调用。因为 AdapterView（例如 *ListView* 和 *GridView*）会从 Adapter 中动态加载数据，所以每次可能只有一部分内容可能会加载到当前的视图层级中。这意味着 `onView()` 可能无法找到所需的视图。 
>
>要解决这一问题，我们需要使用 `onData()`，它会将我们感兴趣的适配器项加载到屏幕上，然后再在上面进行操作

- 同样的位置创建一个新的类

  - 同样的JUnit、Rule、Test注解

  ```java
  @Test
  public void clickGridViewItem_OpensOrderActivity() {
      onData(anything()).inAdapterView(withId(R.id.tea_grid_view)).atPosition(1).perform( click());
      onView(withId(R.id.tea_name_text_view)).check(matches(withText(TEA_NAME)));
  
  
  }
  ```

- 区别

  ```java
  // finding data in views
  onView(ViewMatcher)
  	.perform(ViewAction)
  	.check(ViewAssertion)
  	
  // finding data in adapterViews
  onData(ObjectMatcher)
  	.DataOptions
  	.perform(ViewAction)
  	.check(ViewAssertion)
  ```



##### *3 测试Intent

> - Intent stub：当你的app使用intent打开某个列表并期待用户选择并返回数据时，可以使用Intent Stub替代这个打开列表并选择数据的过程，以测试app。（测试intent响应）
> - Intent Vertification：验证发送的intent包括了需要的数据（如：跳转到电话页面，需要传入电话号码等）

- Intent stub》同样的位置创建一个新的类

  - 同样的RunWith注解，但是使用不同的Rule注解

  ```java
  @Rule
  public IntentsTestRule<OrderSummaryActivity> mActivityRule = new IntentsTestRule<>(
          OrderSummaryActivity.class);
  ```

  - 挡住所有外部intent（使用Before注解）

  ```java
  @Before
  public void stubAllExternalIntents() {
      // By default Espresso Intents does not stub any Intents. Stubbing needs to be setup before
      // every test run. In this case all external Intents will be blocked.
      intending(not(isInternal())).respondWith(new ActivityResult(Activity.RESULT_OK, null));
  }
  ```

  - 需要先授予打电话的权限（安卓M+，同样使用Before注解）

  ```java
  @Before
  public void grantPhonePermission(){
    if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.M){
      getInstrumentation().getUiAutomation().executeShellCommand("pm grant "+getTargetContext().getPackageName()+" android.permission.CALL_PHONE");
    }
  }
  ```

  - 测试

  ```java
  @Test
  public void pickContactButton_click_SelectsPhoneNumber() {
      // Stub all Intents to ContactsActivity to return VALID_PHONE_NUMBER. Note that the Activity
      // is never launched and result is stubbed.
      intending(hasComponent(hasShortClassName(".ContactsActivity")))
              .respondWith(new ActivityResult(Activity.RESULT_OK,
                      ContactsActivity.createResultData(VALID_PHONE_NUMBER)));
  
      // Click the pick contact button.
      onView(withId(R.id.button_pick_contact)).perform(click());
  
      // Check that the number is displayed in the UI.
      onView(withId(R.id.edit_text_caller_number))
              .check(matches(withText(VALID_PHONE_NUMBER)));
  }
  ```

- Intent Vertification》同样的位置创建一个新的类

  - 仅测试不同：模拟了输入数据并确认、验证了intent

  ```java
  @Test
  public void typeNumber_ValidInput_InitiatesCall() {
      // Types a phone number into the dialer edit text field and presses the call button.
      onView(withId(R.id.edit_text_caller_number))
              .perform(typeText(VALID_PHONE_NUMBER), closeSoftKeyboard());
      onView(withId(R.id.button_call_number)).perform(click());
  
      // Verify that an intent to the dialer was sent with the correct action, phone
      // number and package. Think of Intents intended API as the equivalent to Mockito's verify.
      intended(allOf(
              hasAction(Intent.ACTION_CALL),
              hasData(INTENT_DATA_PHONE_NUMBER)));
  }
  ```



##### *4 测试使用闲置资源(Idling Resources)

- 闲置：消息队列中没有ui事件、默认的AsyncTask线程池中没有更多任务时

- 设置使用闲置资源：Espresso会等到app闲置时（后台线程没事做了）才执行下一步操作。假若app使用线程从网络上加载数据，则应该等到加载完成后才进行测试

- 初始代码中，需要设置对Idling Resources是否空闲的判断：实现接口IdlingResource。然后在Handler中，调用前将其设为false（非空），调用后设为true（空）

- 在build.gradle中添加注解

  ```groovy
  compile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'
  ```

- 同样的位置创建一个新的类

  - 同样的RunWith注解，但是使用不同的Rule注解

  ```java
  @Rule
      public ActivityTestRule<MenuActivity> mActivityTestRule =
              new ActivityTestRule<>(MenuActivity.class);
  ```

  - 因为需要在测试之前初始化IdlingResource，所以使用了`@Before`注解。Activity创建之后才会执行该注解中内容
  - 在`@After`中注销注册

  

  ```java
  @Before
  public void registerIdlingResource() {
      mIdlingResource = mActivityTestRule.getActivity().getIdlingResource();
      // To prove that the test fails, omit this call:
      Espresso.registerIdlingResources(mIdlingResource);
  }
  
  @After
  public void unregisterIdlingResource() {
      if (mIdlingResource != null) {
          Espresso.unregisterIdlingResources(mIdlingResource);
      }
  }
  ```

  



##### *5 拓展

- 还可以测试：Web、RecyclerView（与上述不同，不使用onData）
- 使用Espresso Teset Recoder



[TOC]

## Jetpack篇

### #kotlin

##### *1 基本语法

- 变量声明

  - var声明可变变量
  - val声明只读变量
  - Kotlin 中编译器可以通过变量的值来自动推导变量是什么类型的，这种功能称为自动类型推导。不指定变量类型的声明方式叫隐式声明

  ```kotlin
  class Student {
    var name: String = "小明"
    var age = 10
    val sex: String = "男"    
  
    fun learn() {    
      print("$name is learning")    
      }
  }
  ```

- when表达式：条件语句

  ```kotlin
  val b = when (num) {     
    in 0..9 -> {true}     
    else -> { false }
   }
  ```

- in关键字

  ```kotlin
  //判断是否在区间内 
  if (num in 1..9) {     
      print("ok") 
  }  
  //不在区间内 
  if (num !in 1..9) {     
      print("no") 
  }  
  //遍历数组 
  for (name in names) {     
      print(name)
  }  
  //判断name是否在数组内
  if (name in names) {     
      print("ok") 
  }
  ```

- 类型判断

  ```kotlin
  fun foo(o: Any): Int {     
    if (o is String) {         
       //判断完类型之后，o会被自动转换为String类型         
       return o.length    
    }      
    //可以使用!is来取反    
    if (o !is String) {         
        return 0  
     }     
     return 0
  }
  ```

- 空值检测：使用 ? 来进行空值检测，例如 str?.length 来表示 str 不为空时执行获取长度操作，可以避免空指针异常

- 函数声明

  - 返回int类型

  ```kotlin
  fun plus(x: Int, y: Int) : Int {    
     return x + y 
  }
  ```

  - 如果函数只有一行，可以简写

  ```kotlin
  fun plus(x: Int, y: Int): Int = x + y
  
  // 如果是编译器能够推断出的类型
  fun plus(x: Int, y: Int) = x + y
  ```
  - 函数默认参数：减少重载数量

  ```kotlin
  fun plus(x: Int, y: Int = 10) : Int {     
        return x + y 
  }
  ```

  - 可变参数

  ```kotlin
  //java中，可变参数使用...表示
  public void selectCourse(String... strArray){
  
  }
  
  //kotlin中，可变参数使用vararg关键字表示 
  fun selectCourse(vararg strArray: String?) {    
  
  }
  ```

##### *2 类与对象

- 主构造函数与次构造函数：主构造函数跟在类名之后，没有可省略

  ```kotlin
  //声明带一个参数的主构造函数
  class Person constructor(name:String){
    
     init {
          //初始化的代码可以放到init初始化块中
          //初始化块是主构造函数的一部分，因此所有的初始化块中的代码都会在次构造函数体之前执行
      }
  
      //次级构造函数委托给主构造函数直接委托
      constructor(name:String，parent:Person):this(name){
      }
  
      //委托给别的次级构造函数间接委托
      constructor(name:String,parent:Person,age:Int):this(name,parent){
      }
  }
  ```

- 类的继承

  - kotlin中所有类都有一个共同的超类Any。Any有三个方法：equals(), hashCode(), toString()
  - 如果派生类有一个主构造函数，其基类型则必须用基类的主构造函数参数初始化

  ```kotlin
  class Derived(p: Int) : Base(p){}
  ```

- 类的属性

  - kotlin类中变量会自动生成getter和setter
  - 自定义setter需要使用幕后字段，使用关键字field表示

  ```kotlin
  // 以下代码会引起崩溃
  class Person {
      var name = ""
          set(value) {
              this.name = value
          }
  }
  
  // 使用幕后字段
  class Person {
      var name = ""
          set(value) {
             filed = value
          }
  }
  ```

- 常量val：值不可变。使用const或`@JvmField`注解

- 一般地，属性声明为非空类型必须在构造函数中初始化。使用lateinit修饰符可以延迟初始化

- 内部类：默认为静态内部类。添加inner标记后变为非静态内部类

- 数据类：用来保存数据的类。使用data关键字标记

- 枚举类：Kotlin 中枚举类中每一个枚举都是一个对象，并且之间用逗号分隔

  ```kotlin
  enum class Direction {
    NORTH, SOUTH, WEST, EAST
  }
  ```

- 枚举常量的匿名类：必须提供一个抽象方法 (必须重写的方法)。且该方法定义在枚举类内部。而且必须在枚举变量的后面。

- 委托

##### *3 函数

- 局部函数

  ```kotlin
  fun outer(str: String) {
        fun inner(index: Int) {
            str.substring(0, index)
        }
        inner(2)
  }
  ```

- 函数类型

- lambda表达式

- 匿名函数、高阶函数

- 中缀表达式

- 内联函数

- 标准库函数

  - run：返回值为函数块最后一行。或是return表达式
  - apply：调用某对象的 apply 函数，在函数块内可以通过 this 指代该对象。返回值为该对象自己
  - let：调用某对象的 let 函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定 return 表达式
  - also：调用某对象的 also 函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为该对象自己
  - with：将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定 return 表达式




### # Jetpack

##### *1 ViewModel

- 专门用于存放与界面相关的数据，即使Activity因旋转屏幕而销毁数据仍存在不会丢失

- 添加依赖

  ```groovy
  implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
  ```

- 创建ViewModel子类

  - 应该给每一个Activity和Fragment都创建一个ViewModel

  ```kotlin
  class MainViewModel() : ViewModel(){
      var counter = 0
  }
  ```

  - Activity中初始化ViewModel

  ```kotlin
  lateinit var viewModel : MainViewModel
  ```

  ```kotlin
  viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
  ```

  - 获取ViewModel中数据：直接访问即可

- 如果要往ViewModel中传入数据，则需要使用到ViewModelProvider.Factory的子类（比如传入保存到sp中的数据）

  ```kotlin
  class MainViewModel(countReserved:Int) : ViewModel(){
      var counter = countReserved
  }
  ```

  ```kotlin
  class MainViewFactory(private val countReserved: Int) : ViewModelProvider.Factory{
      override fun <T : ViewModel?> create(modelClass: Class<T>): T {
          return MainViewModel(countReserved) as T
      }
  }
  ```

  ```kotlin
  viewModel = ViewModelProvider(this, MainViewFactory(countReserved)).get(MainViewModel::class.java)
  ```



##### *2 Lifecycles

- 用于获取或监听生命周期

- 实现LifecycleObserver接口

  - 使用`@OnLifecycleEvent()`注解并传入生命周期。当Activity处于对应生命周期时执行
  - 通过`.currentState`来获取当前生命周期的状态

  ```kotlin
  class MyObserver(val lifecycle: Lifecycle) : LifecycleObserver{
      private val TAG = "MainActivity"
  
      @OnLifecycleEvent(Lifecycle.Event.ON_START)
      fun activityStart(){
          Log.d(TAG, lifecycle.currentState.toString())
      }
  
      @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
      fun activityStop(){
          Log.d(TAG, lifecycle.currentState.toString())
      }
  }
  ```

  - 在activity中，只需传入Observer

  ```kotlin
  lifecycle.addObserver(MyObserver(lifecycle))
  ```



##### *3 LiveData

- 单纯使用ViewModel无法将数据及时通知给Activity。使用LiveData订阅数据变化

- 重新编辑ViewModel子类中的代码

  - MutableLieData：可变的LiveData

  ```kotlin
  val counter = MutableLiveData<Int>()
  init{
      counter.value = countReserved
  }
  
  fun plusOne(){
      val count = counter.value ?:0
      counter.value = count+1
  }
  
  fun clear(){
      counter.value = 0
  }
  ```

  - Acttivity中传入观察者
    - 第一个参数是LifecycycleOwner对象，Activity本身就是一个LifecycleOwner对象
    - 第二个参数是Observer接口，数据发生改变时执行以下代码

  ```kotlin
  viewModel.counter.observe(this,Observer{ count ->
      infoText.text = count.toString()
  })
  ```

- 为LiveData提高安全性

  - counter变量改名为`_counter`变量，并加上private修饰符。此时counter对外不可见
  - 重新定义一个counter变量，并声明类型为不可变的LiveData，并在它的get()属性方法中返回了一个_counter变量

  ```kotlin
  val counter: LiveData<Int>
  	get() = _counter
  
  private val _counter = MutableLiveData<Int>()
  init {
      _counter.value = countReserved
  }
  
  fun plusOne(){
      val count = counter.value ?:0
      _counter.value = count+1
  }
  
  fun clear(){
      _counter.value = 0
  }
  ```


##### *4 Room

- Room：官方推出的ORM框架，可以使用面向对象思维与数据库交互

- Room结构

  - Entity：用于定义封装实际数据的实体类。每个实体类都会在数据库中有一张对应的表，并且表中的列是根据实体类的字段自动生成的
  - Dao：数据访问。在这里对数据库的各项操作进行封装
  - Database：用于定义数据库中的关键信息，包括版本、返回实例

- 添加依赖

  ```groovy
  // kotlin版
  plugins {
      // ...
      id 'kotlin-kapt'
  }
  // ...
  
  dependencies {
  		// ...
    
      implementation 'androidx.room:room-runtime:2.3.0'
    
    	// kotlin下
      kapt 'androidx.room:room-compiler:2.3.0'
    
    	// java下
    	annotationProcessor "androidx.room:room-compiler:2.3.0"
  }
  ```

- 配置实体类

  - 这个表格名字叫做User，但是其每一列都是一个User对象
  - 可以使用以下方法更改命名
    - `@Entity(tableName = "word_table")` 
    - `@ColumnInfo(name = "word")`

  ```kotlin
  @Entity
  data class User(var firstName:String, var lastName:String, var age:Int){
    
      @PrimaryKey(autoGenerate = true)
      var id:Long = 0
    
  }
  ```

- 配置Dao接口

  - 默认配置下，所有查询必须在单独的线程中执行
    - 冒号：使用参数列表中的参数


  ```kotlin
  @Dao
  interface UserDao {
      @Insert
      fun insertUser(user:User):Long
  
      @Update
      fun updateUser(newUser:User)
  
      @Query("select * from User")
      fun loadAllUsers():List<User>
  
      @Query("select * from User where age > :age")
      fun loadUserOlderThan(age:Int):List<User>
  
      @Delete
      fun deleteUser(user: User)
  
      @Query("delete from User where lastName = :lastName")
      fun deleteUserByLastName(lastName: String):Int
  }
  ```

- 配置Database类

  - companion object：类似静态变量

  ```kotlin
  @Database(version = 2, entities = [User::class, Book::class])
  abstract class AppDatabase : RoomDatabase() {
      abstract fun userDao():UserDao
  
      companion object{
          private var instance:AppDatabase?=null
  
        	// val MIGRATION_1_2 = object:Migration(1,2){ 见下
        
          @Synchronized
          fun getDatabase(context: Context):AppDatabase{
              instance?.let{
                  return it
              }
              return Room.databaseBuilder(context.applicationContext,
                  AppDatabase::class.java,"app_database")
                  .addMigrations(MIGRATION_1_2)
                  .build().apply {
                      instance = this
                  }
          }
      }
  }
  ```

- 数据库的升级（在`.addMigrations(MIGRATION_1_2)`中）

  ```kotlin
  val MIGRATION_1_2 = object:Migration(1,2){
      override fun migrate(database: SupportSQLiteDatabase) {
          database.execSQL(
              "create table Book (id integer primary key autoincrement not null, name text not null, pages integer not null)"
          )
      }
  }
  ```

- 执行数据库操作

  ```kotlin
  // 获取实例
  val userDao = AppDatabase.getDatabase(this).userDao()
  ```

  ```kotlin
  // 增
  val user1 = User("Tom","Brady",40)
  val user2 = User("Tom", "Hanks",63)
  addDataBtn.setOnClickListener{
      thread {
          user1.id = userDao.insertUser(user1)
          user2.id = userDao.insertUser(user2)
      }
  }
  ```

  ```kotlin
  // 改
  thread {
      user1.age = 42
      userDao.updateUser(user1)
  }
  ```



##### *5 WorkManager

- WorkManager适合用于处理一些要求定时执行的任务

  - 可以根据系统的版本自动选择底层如何实现后台任务
  - WorkManager注册的周期性任务不能保证一定会准时执行。系统会将触发时间接近的几个任务放在一起执行以减少CPU被唤醒次数，从而延迟续航
  - 可能在国产手机上不稳定

- 添加依赖

  ```groovy
  implementation 'androidx.work:work-runtime:2.2.0'
  ```

- WorkerManager使用步骤

  1. 定义一个后台任务，并实现具体的任务逻辑
  2. 配置该后台任务的运行和约束信息，并构建后台任务请求
  3. 将该后台任务请求传入WorkManager的enqueue()方法中。系统会在合适的时间运行

- 创建Worker子类

  - doWork()方法
    - 不会运行在主线程当中
    - 要求返回一个Result对象用于表示任务的运行结果。运行结果包括了success()、failure()和retry()

  ```kotlin
  class SimpleWorker(context:Context, params:WorkerParameters): Worker(context, params) {
      override fun doWork(): Result {
          Log.d("SimpleWorker", "doWork: ")
          return Result.success()
      }
  }
  ```

- 配置以及传入

  - OneTimeWorkRequest.Builder用于构建一次性的后台任务
  - PeriodcWorkRequest.Builder用于构建周期性的后台任务，要求运行时间不得小于15"

  ```kotlin
  val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
  
  WorkManager.getInstance(this).enqueue(request)
  ```

  



