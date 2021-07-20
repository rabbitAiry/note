# 使用RecyclerView展示单条地铁线路图》MetroBoard0.1

[TOC]

> 图片1

### 初衷

 	有一段时间觉得自己目前的能力不足以做出很多形式的app，包括自己想要完成的。在无意义地耗费了两个星期后，觉得随便做点东西也比什么都不做好，所以就有了这个展示地铁线路图的app。

​	这个app的完成度很低，地铁图我选择了我所在的城市的广州地铁，目前把1-3号线的站点都使用数组输入到程序中了，在代码中修改参数可以看到其他的线路，但在手机app上就没有办法了，但没关系，这个锅可以在下次更新填上。

​	制作MetroBoard的初衷是想在手机上制作出地铁或公交里面不断展示线路信息、当前站点的显示屏的类似效果，不过以我当前能力来看，这个开发周期估计要特别特别的长

### 流程概述

##### ui

​	如图所见，这个线路图是一个纵向的列表，显然这需要RecyclerView或者ListView来完成。每个item会有当前站点的序号，还模拟了官方的药丸边框；还有当前站点的中英文以及换乘信息。

> 图片2

​	先是处理了ui部分：在左边的药丸中，图形样式通过drawable实现，但drawable的颜色会在java中根据要展示的线路更改；药丸的实现是一个FrameLayout包裹了一个LinearLayout，LinearLayout包括了两个TextView还有中间细细的线。

```xml
<FrameLayout
    android:layout_width="70dp"
    android:layout_height="match_parent"
    android:layout_alignParentLeft="true"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/image_big_line_color"
        android:layout_width="10dp"
        android:layout_height="match_parent"
        android:layout_gravity="center"
        android:background="@color/line2" />

    <LinearLayout
        android:id="@+id/linear_line_color"
        android:layout_width="42dp"
        android:layout_height="22dp"
        android:layout_gravity="center"
        android:background="@drawable/oral">

        <TextView
            android:id="@+id/text_line_id"
            style="@style/oral_text"
            android:paddingLeft="4dp"
            android:maxLines="1"
            tools:text="2" />

        <ImageView
            android:id="@+id/image_line_color"
            android:layout_width="2dp"
            android:layout_height="match_parent"
            android:background="@color/line2" />

        <TextView
            android:id="@+id/text_station_id"
            style="@style/oral_text"
            android:paddingRight="2dp"
            tools:text="12" />
    </LinearLayout>
</FrameLayout>
```

​	站点中英文部分使用了LinearLayout，为了还原官方线路图中中英文的间隔很紧，对中文的TextView添加了负的marginBottom；同时给英文的TextView设置了小的行间距，这样，当有两行英文时视觉效果不会太拉跨。

```xml
<LinearLayout
    android:layout_width="0dp"
    android:layout_height="match_parent"
    android:layout_weight="6"
    android:orientation="vertical"
    android:paddingHorizontal="10dp">

    <Space
        style="@style/station_text"
        android:layout_weight="2" />

    <TextView
        android:id="@+id/text_station_ch"
        style="@style/station_text"
        android:layout_marginBottom="-6dp"
        android:layout_weight="5"
        android:gravity="bottom"
        android:text="海珠广场"
        android:textSize="22dp" />

    <TextView
        android:id="@+id/text_station_en"
        style="@style/station_text"
        android:layout_weight="5"
        android:lineSpacingExtra="-4dp"
        android:text="Haizhu Square"
        android:textSize="12dp" />
</LinearLayout>
```

​	至于换乘部分，部分站点会有可能有1-3个换乘线路，这里同样使用FrameLayout完成，包含了要换乘线路的数字以及一个彩色的圈，这个彩色的圈会在java中更改颜色（埋坑了，下文会讲）。一开始以为RecyclerView中不能嵌套RecyclerView，所以复制了三个FrameLayout，默认是invisible的，有换乘再修改。复制代码确实是愚蠢的行为，应该早点百度的o(TヘTo)

```xml
<FrameLayout
  android:id="@+id/frame_ring0"
  style="@style/ring_frame"
  android:visibility="visible">

  <ImageView
      android:id="@+id/image_ring_color0"
      android:layout_width="32dp"
      android:layout_height="32dp"
      android:layout_gravity="center"
      android:background="@drawable/circle6" />

  <TextView
      android:id="@+id/text_ring_line_id0"
      style="@style/ring_text"
      android:maxLines="1"
      android:text="6"
      android:textSize="16dp" />
</FrameLayout>
```

##### 逻辑

​	逻辑部分，先是愉快地设置上adapter，愉快地绑定view，愉快地设置内容，代码随后呈现

​	线路的站点正如前面提到，是靠数组完成的，其中数组的第三四五项是可以换乘的线路。在adapter的onBindViewHolder()中通过其position获取对应数组

```java
    private static String[][] line3 = {{"番禺广场", "Panyu Square"}, {"市桥", "Shiqiao"},
            {"汉溪长隆", "Hanxi Changlong", "7"}, {"大石", "Dashi"}, {"厦滘", "Xiajiao"}, {"沥滘", "Lijiao","GF"},
            {"大塘", "Datang"}, {"客村", "Kecun","8"},{"广州塔","Canton Tower","APM"},
            {"珠江新城","Zhujiang New Town","5"},{"体育西路","Tiyu Xilu","1"},{"石牌桥","Shipaiqiao"},
            {"岗顶","Gangding"},{"华师","South China Normal University"},{"五山","Wushan"},
            {"天河客运站","Tianhe Coach Terminal","6"}};
```

​	然后，我希望当我选择不同线路时，状态栏和actionBar都会显示该线路的颜色，所以首先在themes.xml中将parent改为`... .NoActionBar`（我的代码中忘记配置暗夜模式了），然后添加一个Toolbar

```xml
<androidx.appcompat.widget.Toolbar
    android:id="@+id/toolbar_list_station"
    android:layout_width="match_parent"
    android:layout_height="70dp"
    android:background="@color/line2"
    app:layout_constraintTop_toTopOf="parent">
    <!--20dp is for status bar-->
    <TextView
        android:id="@+id/text_list_station_title"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        tools:text="Line 2"
        android:textColor="@color/white"
        android:textSize="20sp"
        android:gravity="center"/>
</androidx.appcompat.widget.Toolbar>
```

​	将状态栏设置为透明的，这里参考了别人的[文章](https://www.jianshu.com/p/e89ee0a77bb5)

```java
StatusBarUtils.setStatusBarFullTransparent(this);
toolbar = findViewById(R.id.toolbar_list_station);
toolbar.setFitsSystemWindows(true);
```

​	最后一步是设置toolbar的颜色，直接调用`.setColor`即可。为了防止部分深色背景下黑色字不容易看清，使用switch-case为这些颜色设置了白色字体

```java
switch (lineId){
    case "2":
    case "6":
    case "21":
        textViewListStationTitle.setTextColor(getColor(R.color.white));
    default:
        textViewListStationTitle.setTextColor(getColor(R.color.black));
}
```

​	其实还想为recyclerView的添加底部，用作留白，但实现方法并没有看懂，也没有操作出来。

​	这就是编写这个app的全部流程了，但是实际上这个流程并不顺利，在RecyclerView上遇到了很多问题

### 遇到的问题

​	遇到的第一个问题：换乘站的圆形颜色无法显示正确颜色，而部分item的药丸会显示原有的颜色

> 图片

​	遇到的第二个问题：随着在RecyclerView中不断划动，会出现数据错乱的现象

​	设置RecyclerView的禁止复用可以解决问题二，但会让界面变得很卡，且不能够解决问题一。后来搜索了网上的解决方案，发现每个if语句都需要有对应的else语句可以解决这个问题。一开始我的if-else语句是这样的：三个可能出现的换乘对应三个framelayout，如果是换乘站，则把frameLayout设置为可见

```java
if(l>2){
	if(l>3){
		if(l>4){
		
		}else{
		
		}
	}else{
	
	}
}else{

}
```

​	这丝毫不起作用，第一个换乘位不会错乱了，但第二个换乘位会经常串位

> 图片

​	相应的，改为下面这种形式解决了问题二。因为复用，所以当划动到某个位置的item需要绘制ui时，会拿到别人的item修改后直接使用。若别人的item符合if语句并做出了相应的ui变化，但是当前item并不符合该if语句，此时由于没有else语句，当前item会用上了这种ui变化，从而导致出错。所以else语句中的内容应该是指导item如何复原，不要接着套用这种ui变化。上一个代码块中，虽然有if-else语句，但是当不符合l>2时，便不会走到l>3时的else语句，所以复用时候依然会出错

```java
if (l > 2) {
    holder.imageViewRingColor0.setBackground(getRingBackground(st[2]));
    holder.textViewRingLineId0.setText(st[2]);
    holder.frameRing0.setVisibility(View.VISIBLE);
} else {
    holder.frameRing0.setVisibility(View.INVISIBLE);
}
if (l > 3) {
    holder.imageViewRingColor1.setBackground(getRingBackground(st[3]));
    holder.textViewRingLineId1.setText(st[3]);
    holder.frameRing1.setVisibility(View.VISIBLE);
} else {
    holder.frameRing1.setVisibility(View.INVISIBLE);
}
// ...
```

​	然后来解决问题一。明明debug时候获取的颜色是正确的，但是就是无法保持正确的显示。无助的我只能够猜测这是ui还没来得及换颜色。

```java
GradientDrawable oral = (GradientDrawable) context.getDrawable(R.drawable.oral);
oral.setStroke(6, lineColor);
```

```java
int lineColorExtra = LineUtils.getLineColor(st[2], context);
GradientDrawable circle = (GradientDrawable) context.getDrawable(R.drawable.circle6);
circle.setColor(lineColorExtra);
holder.frameRing0.setBackground(getRingBackground(st[2]));
```

​	药丸的颜色虽然会随线路颜色而改变，但是RecyclerView的每一刻都应该只会出现一种颜色，这意味着药丸颜色的修改不必在每个viewHolder中都修改一次，而是直接在setAdapter时设置就好了，所以我把这行代码移到了Activity中。这样确实解决了问题

​	至于换乘的小圆环，我没有想到好的解决方法。想到一个RecyclerView中要出现这么多次圆环，每次还要上色，所以干脆把所有颜色的圆环都单独设置为一个drawable。但是drawable居然没有继承而言，所以只好做点龌龊的复制粘贴，最终解决了问题

​	在网上搜索了很多解决方案，还跟着部分博主看了点RecyclerView的源码——看得我挺自闭的，这大概就是“路还远着呢”的体现吧。

​	最后奉上[源码](https://gitee.com/rabbit-airy/metro-board)，宿舍打开github太难了，只好用了gitee

