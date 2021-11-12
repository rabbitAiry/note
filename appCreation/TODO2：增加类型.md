# TODO2：增加类型

[TOC]

> 偷了一个暑假得出懒，终于快要完成了/(ㄒoㄒ)/~~

### #前言

​		从TODO1到TODO2，首先第一步是去思考改进什么。这次的版本更新，带来了全新的ui界面：使用了两个fragment来展示TODO项。fragment_NOW用于展示常规类和紧急类的TODO，而fragment_ALL则用于展示所有的TODO，并且用户可以选择自己想看的类型。同时新增了Daily类型的TODO，他会每天生成一个同名的常规类TODO，这个TODO只会在NOW页面中显示。即时今天用户完成与否，第二天会重新生成TODO。

​	回顾了TODO1的代码，发现存在着重复的Adapter类：分别用于展示常规类TODO的Adapter和展示cold类TODO的Adapter。当初对继承和接口的概念都不是很深刻，所以这次也着手优化了代码



### #从UI入手

​	假期其实也有尝试过做其他的app，但最后都没有完成，这个消息对我来说也太绝望了。原因既包括了我在家偷懒，也有开发流程中比较困乱，有时候甚至都不知道自己该做什么，包括产生了bug时也有点无动于衷。

​	所以开学后的第一个app，我给自己的开发流程做了粗略的计划：先尽最大努力完成ui部分，再上数据。尽最大努力完成ui部分的意思是能够安装到手机里正常运行，点击、滑动都能够作出响应。这样一方面能分离开发流程的开发步骤，手上拿着空空如也只有ui的app也会更有动力一些。好的，开始阐述流程。

##### 使用fragment

> 图片

​	fragment的创建需要先创建一个Fragment的子类，重写它的onCreateView()方法，并在其中绑定view。接着在Activity类中通过FragmentManager展示出来

​	我希望fragment与fragment之间的切换既可以通过点击toolbar的按钮来响应，也可以通过滑动屏幕来触发。toolbar下面的粗粗的一划会作为当前fragment的指示器，通过设置Visibility来设置是否可见

​	滑动则是依据了安卓的开发指南重写了onTouchEvent()，以及使用GestureDetector实现了左右滑动

```java
// 设置为fragmentNow页面
private void setFragmentNow() {
    if (!isFragmentNow) {
        mainFragmentManager.beginTransaction()
                .replace(R.id.main_activity_container, fragmentNow)
                .commit();
        isFragmentNow = true;
        toolbarIndicatorNow.setVisibility(View.VISIBLE);
        toolbarIndicatorAll.setVisibility(View.INVISIBLE);
        toolbarIndicatorNow.startAnimation(alphaAnimation);
    }
}
```



##### 添加页面和修改页面

> 图片

​	现在才知道安卓提供了CardView组件。这个组件可以指定边框圆角，默认也自带阴影，是一个颜值在线的view，所以这次todo的添加、修改页面都用上了它。

##### 文字长度

​	todo的内容不宜过长，所以我限制了长度`android:maxLength="15"`。为了能够在主页面中当文字过长时能够让用户知道，所以在todo_item布局中的Text View，我添加了一句`android:ellipsize="end"`，他会在内容不能完全展示完时放置`...`

##### 加点动画

​	这一步实际上是我完成了所有步骤之后才去做的，但是在测试过程中发现了setDuration()中的参数稍大时，来回滑动容易让程序直接闪退，还没了解是什么原因。如下现在duration设为100ms是ok的，但是动画效果就不太明显了

```java
alphaAnimation = new AlphaAnimation(0, 1);
alphaAnimation.setDuration(100);

private void setFragmentAll(){
// ...
toolbarIndicatorAll.startAnimation(alphaAnimation);
}
```

##### 按键按下即添加

​	按下“√”之后就相对于点击提交按钮。这个功能一开始我加上了，但是又注释掉了，最后又加上了😓。这是因为我在非全面屏上的手机做测试，view挤得重叠了，用起来貌似体验不好，所以注释掉了。但是后来在我的全面屏手机上觉得没有这个功能不顺手，所以还是添加上了

```java
@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    switch (keyCode) {
        case KeyEvent.KEYCODE_ENTER:
            todoSubmit();
        default:
            return super.onKeyUp(keyCode, event);
    }
}
```

### #连接数据

##### 重新修改数据库

​	这次的数据库表格添加了“IS_DAILY”栏，可以看作是一个标签，只要这个todo是由每日型的todo生成的都会有这么一个标签。事实上我还希望我的app以后能够设置提醒事件，但是暂时用不上，所以加了注释

```java
public static class TodoEntry implements BaseColumns {
    public static final String TABLE_TODO = "TODO";
    public static final String TODO_COLUMN_CONTENT = "CONTENT";
    public static final String TODO_COLUMN_TYPE = "TYPE";
    public static final String TODO_COLUMN_IS_CREATED_BY_DAILY = "IS_DAILY";
//        public static final String TODO_COLUMN_DATE = "DATE";
//        public static final String TODO_COLUMN_HAS_REMINDER = "HAS_REMINDER";
//        public static final String TODO_COLUMN_REMIND_TIME = "REMIND_TIME";
}
```



##### 查询数据，传递给Adapter

​	正如开头所提，TODO1中的两个adapter代码重复度高，所以这次用上了继承。这里有两个adapter，一个用于制造“Now”页面的列表NowAdapter，另一个用于制造“全部”页面的列表AllAdapter，他们的区别在于前者的item中会多展示两个按钮，分别代表着：用户完成了该todo、用户需要加急该todo；同时NowAdapter还会展示该todo是否是被生成的，这只需要检索数据库即可知道。所以我的NowAdapter继承了AllAdapter，并重写其onBindViewHolder()以为按钮绑定点击事件。

​	在item_button_done的点击事件中，这次我尝试了用notifyItemRemoved()以代替notifyDataSetChanged()，发现这个参数居然自带动画，有点小激动。



##### 针对Daily类型TODO的代码

​	Daily类型的TODO要格外优化这两件事：一是当创建或更改为Daily类型的TODO时，需要直接多生成一个常规类型的TODO项，二是倘若今天是你第一次打开app，则会查找所有的TODO项并逐个生成常规类型的TODO。因此第二步需要用到偏好设置SharedPreference来保存每一次打开该app的日期，然后每次打开app时，就拿今天的日期和偏好设置中的日期作比较。这些事我担心耗时太长，所以放在了AsyncTask中完成。届时遇到了studio给我提warning（整个task直接标黄，有点恐怖），提示是This AsyncTask class should be static or leaks might occur 有机会内存泄漏，但我不知道该怎么做，只好根据提示加了行标志`@SuppressLint("StaticFieldLeak")



### #遇到的问题

##### 数据库挂了

​	其实这次代码改进后，数据库的查询写得有些乱，重复部分也很多。开始写代码时的思路是SQLReadDatabase可以一直常驻，但SQLWriteDatabase应该用完即关，因为在这个app中，每次更改fragment都会用到查询数据，但是写数据库的操作倒不是经常用到。所以一开始我在实现完成按钮和加急按钮的方法中加了一行`todoWriteDatabase.close()`，但之后发现程序直接闪退，因为报错原因是todoReadDatabase已被关闭。不知道为什么关闭写数据库也会导致读数据库被关闭，当时我还没发现错误原因是什么，然后就测试类debug和log的方式去找原因，（其实当时我懵了，直接趴在床上玩手机了）。log的效率会比debug高太多了

##### 滑动失效了 

​	这事我也有点懵，为什么本来能够正常滑动，但是添加了数据后就划不了了。所以我找了万能的的c站我应该怎么靠手势切换fragment，发现更多人用的都是viewPager。最后发现了这一篇文章：[包含listview的Fragment左右手势滑动切换](https://blog.csdn.net/u012604745/article/details/49946775?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163214506816780274148309%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=163214506816780274148309&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-49946775.pc_search_insert_js_new&utm_term=%E5%B7%A6%E5%8F%B3%E6%BB%91%E5%8A%A8%E5%88%87%E6%8D%A2fragment&spm=1018.2226.3001.4187) 。才发现是和recyclerView的手势识别冲突了，跟着大佬走，问题得到了解决。。。但是新的问题又出现了：只有在空白的区域滑动fragment才能够得到切换，在有item的区域划动是没有反应的。。。偷懒了，下次再更新吧~



### #小结

​	这次的app开发流程很长，因为中途要上课，空闲的时间也可能只是在玩游戏。所以当遇到bug的时候感觉心态崩了，感觉这四周过得很拉跨，但幸好最后还是做出了成品，这是从计划安排层面的小结。代码上会有些重复和混乱，所以也期待下一个版本会改善这些问题