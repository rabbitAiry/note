[TOC]

## Recycler View

##### 原理

- 原理是其view中的内容一直在队列里待命，是ListView的升级版

- | **Layout Manager** |   指导   | **Recycler View** |
  | ------------------ | :------: | ----------------- |
  |                    |          | 通过ViewHolder    |
  | **Data Source**    | 数据绑定 | **Adapter**       |

  - 使用ViewHolder对象来传输数据来缓存布局中表示的视图对象以降低成本
  - Layout Manager将会指导RecyclerView如何排列这些数据：垂直，网格等
  - RecyclerView 负责回收视图



##### 继承自adapter类中的函数

- 成员变量：`LayoutInflater`、`需要的数据格式`

- 构造器：获取Context，并通过调用`=LayoutInflater.from(context)`保存到成员变量中

- `onCreateViewHolder()`当recyclerView实例化一个新的ViewHolder时，会调用该函数。在这个函数中：

  - 获取context（或者从构造器中获取）：直接获取或者使用`viewGroup.getContext()`

  - 使用xml将其填充（inflate）或以代码的形式创建（或者从构造器中获取）：获取LayoutInflater(inflater)

    `= LayoutInflater.from(context);`

  - 保存view

    `= (inflater).inflate(R.layout.某个xml文件, ViewGroup parent, boolean attachToRoot)`,暂时，最后一个参数填false

  - 返回viewHolder:`return new TodoHolder(view)`

- `onBindViewHolder()`recyclerView在展示数据时会调用该函数，此时adpter已准备好所有的viewholder，其中接收的`int position`是当前item的位置。每当用户滚动页面的时候也会重新调用这个函数以绑定新的数据。*应该先完成`ViewHolder`的绑定部分再处理这个函数*：

  - 调用`(holder)`中对每个item进行的操作

- `getItemCount()`返回一共有多少个项目，这个函数会被RecyclerView多次调用以询问



##### ViewHolder

- 在构造器中通过`itemView.findViewById()`来绑定每个itemt的内容，例如textview
- 对每个item进行单独操作也应该在该类中创建方法，比如单击事件，然后等待其在`onBindViewHolder()`中被执行



##### UI

- 在使用`RecyclerView`的layout上输入

  ```xml
  <androidx.recyclerview.widget.RecyclerView
          ...
          tools:listitem="@layout/word_list_item"
          ... />
  ```

  可以便于查看使用效果

- 编写在item项的layout

  - 特别注意：item的根排列方向（默认`layout_height`）**应该是**`wrap-content`或指定高度，属性为`match-parent`时，每个item的大小会占据整个屏幕，但是可以划动。

##### 第一步

1. 添加依赖`com.android.support:recyclerview-v7:24.1.1`

2. 在xml中增加recyclerview

3. 新建**一个类**继承自`RecyclerView.Adapter<viewHolder的类>` `(GreenAdapter)`以及**一个类**继承自`RecyclerView.ViewHolder` `(NumberViewHolder)`

4. 在`onCreate()`中使用`findViewById()`绑定该recyclerView

   - 为该RecyclerView绑定id

   - 实例化一个对应的adapter

   - 搭线

      `recyclerView.setAdapter(...)`

   - 设置布局管理器

      `recyclerView.setLayoutManager(new LinearLayoutManager(this))`

5. 当知道recyclerView的布局大小不变时可以使用`mNumbersList.setHasFixedSize(true);`来提升性能



##### 增加点击事件

1. 创建一个接口来定义侦听器

   ```java
   public interface ListItemClickListener{
           void onListItemClick(int clickedItemIndex);
       }
   ```

2. 向`(GreenAdapter)`添加一个成员变量用来存储对列表项点击侦听器的引用

   `private final ListItemClickListener mOnClickListener;`

3. 修改`(GreenAdapter)`的构造函数，使其接受一个额外参数ListItemClickListener

4. 修改`(NumberViewHolder)`，令其实现接口`View.OnClickListener`

5. 并在其中重写`onClick(View view)`，通过`getAdapterPosition()`获得被点击item的位置，并通过`.onListItemClick`传给`mOnClickListener`

6. 在`(GreenAdapter)`的构造器中调用`itemView.setOnClickListener(this)`

7. 在`MainActivity`中实现接口`GreenAdapter.ListItemClickListener`，并重写`onListItemClick`，在其中添加点击事件

8. 修改实例化的Adapter类使其传递多一个额外参数ListItemClickListener





##### 其他

- Toast
  - toast会按照先后顺序来依次展示，所以每当展示一个toast时应该（若非空）先取消上一个toast`.cancel()`

- 单位尺度dp与sp
  - px = dp * (dpi / 160)
  - sp取决于用户将字体设置的大小

