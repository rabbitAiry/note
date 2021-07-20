# 待办事项TODO：我的第一个app



​	其实也不算是第一个app了，之前还做了一个记帐本，不过因为实在是没有经验也没有勇气，使得开发周期额外的长，最终只有一个半成品的形态。所以这周开始决定去换个方式学习安卓，想了很多我可以实现的，也不太难的app，然后分别每个app花一周时间去完成。待办事项TODO就是这个计划的第一个app



#### 要实现的功能

- 添加todo项，并可以选择属性：加急urgent、常规normal、冷藏cold（暂时不想看见但始终都是要做的事）
- 主页面只显示加急和常规的todo项，且加急项会在优先展示
- 冷宫页面用于显示冷藏的todo项
- 每个todo项可以切换属性（不过最后还是偷懒了，urgent属性不能改为normal属性）



#### 三个界面

##### MainActivity

- 从数据库加载加急、普通的Todo项
- 允许添加待办事项
- 进入冷宫的入口
- 允许每个待办事项：完成、删除、加急或冷却
- 添加待办事项，成功则刷新界面

##### Edit_Todo_Activity

- 选择待办事项的属性
- 添加描述并确认-添加到数据库

##### Cold_Todo_Activity

- 只显示冷暖的Todo项，允许删除、回到普通、加急



​	设置布局的第一件事是挑选颜色，红红绿绿的不太好看，所以我为加急事项选择了橙色，普通事项选择了青色

```xml
    <color name="urgent">#FF9800</color>
    <color name="normal">#2ACD8C</color>
    <color name="cold">#727E7E</color>
```

​	然后是设置布局：其中Cold_Todo_Activity的布局最简单，只有一个RecyclerView；MainActivity的布局则是再多加一个条装至添加Todo的按钮，只不过这个按钮是通过TextView和setOnClickListener一起实现的——因为设计之初希望这个app的风格比较方方正正，所以用TextView添加背景貌似挺方便的。因为布局简单，所以使用什么Layout都没什么所谓

​	使用RecyclerView的目的是为了方便地展示有多少项todo，这就少不了要设置每一项的界面。因为所有的部件都在一条线上，为了方便，使用了`RelativeLayout`。

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="72dp"
    android:background="?attr/cold_bgColor"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/cold_item_description"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_alignParentLeft="true"
        android:layout_marginLeft="28dp"
        android:gravity="center"
        android:textSize="24dp"
        android:background="@null"
        tools:text="吃饭" />

    <ImageButton
        android:id="@+id/become_urgent_fromCold"
        style="@style/cold_button"
        android:layout_marginVertical="20dp"
        android:layout_marginRight="16dp"
        android:layout_toLeftOf="@id/become_no_cold"
        android:background="@drawable/urgent" />

    <ImageButton
        android:id="@+id/become_no_cold"
        style="@style/cold_button"
        android:layout_marginTop="21dp"
        android:layout_marginRight="24dp"
        android:layout_marginBottom="19dp"
        android:layout_toLeftOf="@id/cold_item_status_bar"
        android:background="@drawable/no_cold" />

    <ImageView
        android:id="@+id/cold_item_status_bar"
        android:layout_width="20dp"
        android:layout_height="match_parent"
        android:layout_alignParentRight="true"
        android:layout_marginVertical="16dp"
        android:layout_marginRight="32dp"
        android:background="@color/cold" />


    <ImageView
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_alignParentBottom="true"
        android:layout_marginHorizontal="20dp"
        android:background="@color/dark20" />
</RelativeLayout>
```

​	Edit_Todo_Activity的布局上花了一些时间。因为界面上的部件较多，所以我选用了`constraintlayout`。在设置三个`RadioButton`时我遇到了问题：我希望每个radioButton都可以在左边显示对应属性颜色，中间显示文字，而按钮在最右方。我尝试在RadioGroup中添加一层`constraintlayout`，但这样做的话会让RadioGroup对RadioButton的约束失效，即RadioButton不再只能同时选中一个了。其次，我在`drawable`文件中绘制的颜色方块不能通过`drawableRight`等其他属性添加到RadioButton上，因为这些drawable文件没有具体的长宽。所以最后我在总布局中添加了一层`LinearLayout`，这个布局会在RadioGroup的左边，使得所有项目一一对其；剩下的问题，文字和按钮位置对调，则可以通过属性`layout_Direction`来实现。为了让每一项的属性调整方便，可以将部分属性添加到`styles.xml`中

```xml
<RadioButton
            android:id="@+id/button_normal"
            style="@style/edit_todo_radio_button"
            android:checked="true"
            android:text="常规 normal" />
```

```xml
<style name="edit_todo_radio_button">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">50dp</item>
        <item name="android:layoutDirection">rtl</item>
        <item name="android:textSize">28dp</item>
        <item name="android:layout_marginVertical">8dp</item>
    </style>
```

​	因为学过一点电脑绘图，所以能够自己完成了图标的制作：在项目中直接new一个ImageAsset，选择自己绘制的前景图与背景图就可以了。



#### java部分

##### 数据库

​	因为要面临存储很多项Todo的缘故，所以使用SQLite数据库是必须的。这里使用了`TodoContract`来保存Todo表格的表格名、列名、以及用数字表示的类别，其中`TodoEntry`类实现了`BaseColumns`，这样，每一行都会自带一个独一无二的ID，可以用作PrimaryKey

```java
// TodoContract类
package com.example.todo.data;

import android.provider.BaseColumns;

public class TodoContract {
    private TodoContract() {}

    public class TodoEntry implements BaseColumns {
        public static final String TABLE_NAME = "TODO";
        public static final String COLUMN_ITEM = "ITEM";
        public static final String COLUMN_STATUS = "STATUS";
        public static final int categoryNormal = 0;
        public static final int categoryUrgent = 1;
        public static final int categoryCold = 2;
    }
}
```

​	数据库的操作还离不开`SQLiteOpenHelper`类的帮助，所以新建了一个类`TodoHelper`使其继承该类，创建构造函数，并借用在`TodoContract`中保存的表名和列名重写新建数据表的`onCreate()`和`onUpgrade()`方法，特别要注意的是SQL语句的空格，在字符串连接的时候挺容易漏的

```java
package com.example.todo.data;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import com.example.todo.data.TodoContract.TodoEntry;

import androidx.annotation.Nullable;

public class TodoHelper extends SQLiteOpenHelper {
    public static final int version = 1;
    public static final String name = "todo.db";

    public TodoHelper(@Nullable Context context) {
        super(context, name, null, version);
    }

    private static final String create = "CREATE TABLE " + TodoEntry.TABLE_NAME + " (" + TodoEntry._ID +
            " INTEGER PRIMARY KEY," + TodoEntry.COLUMN_ITEM + " TEXT," + TodoEntry.COLUMN_STATUS + " INTEGER)";

    private static final String delete = "DROP TABLE IF EXISTS " + TodoEntry.TABLE_NAME;

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(create);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL(delete);
        onCreate(db);
    }
}
```



##### RecyclerView

​	既然要使用RecyclerView，那么还要增加它对应的Adapter：`TodoListAdapter`和`ColdTodoAdapter`，以及增加内部类`TodoListViewHolder`和`ColdTodoViewHolder`。在各自的ViewHolder绑定布局的部件，在Adapter需要重写的三个构造方法中，`onCreateViewHolder()`用于指定绑定的布局、返回新的ViewHolder；`onBindViewHolder()`用于为布局添加数据，加急的Todo项理应不能再加急了，所以在这里将向上的箭头设为不可见；`getItemCount()`则用于查找还有多少项数据。其次，还要将RecyclerView和Adapter绑定在一起。

```java
recyclerViewTodoList = (RecyclerView)findViewById(R.id.recyclerView_todo_list);
        adapter = new TodoListAdapter(cursor,this,this);
        recyclerViewTodoList.setAdapter(adapter);
        recyclerViewTodoList.setLayoutManager(new LinearLayoutManager(this));
```

​	在主界面`MainActivity.java`中实例化一个`TodoHelper`并获得可读的数据库和可写的数据库，并使用`Cursor`的实例获取表格数据：加急或常规的Todo项，然后一并传入，这样才能让Adapter方便获取数据。

```java
public TodoListAdapter(Cursor cursor, Context context, TodoItemListener listener) {
    this.cursor = cursor;
    this.context = context;
    this.listener = listener;
}

@NonNull
@Override
public TodoListViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
    View view = LayoutInflater.from(context)
            .inflate(R.layout.todo_item, parent, false);

    return new TodoListViewHolder(view);
}

@Override
public void onBindViewHolder(@NonNull TodoListViewHolder holder, int position) {
    if (!cursor.moveToPosition(position)) return;

    String description = cursor.getString(cursor.getColumnIndex(TodoEntry.COLUMN_ITEM));
    int category = cursor.getInt(cursor.getColumnIndex(TodoEntry.COLUMN_STATUS));
    long id = cursor.getLong(cursor.getColumnIndex(TodoEntry._ID));

    holder.textViewTodoDescription.setText(description);
    int statusColor = 0;
    switch (category) {
        case 0:
            statusColor = ContextCompat.getColor(context, R.color.normal);
            break;
        case 1:
            statusColor = ContextCompat.getColor(context, R.color.urgent);
            holder.imageButtonBecomeUrgent.setVisibility(View.INVISIBLE);
    }
    holder.imageViewTodoStatusBar.setBackgroundColor(statusColor);
    holder.itemView.setTag(id);
    holder.button.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // ...
        }
    });
    holder.imageButtonBecomeUrgent.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // ...
        }
    });
    holder.imageButtonBecomeCold.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // ...
        }
    });
}

@Override
public int getItemCount() {
    return cursor.getCount();
}

```

​	因为布局中，当点击方框即代表完成该Todo项，此时应该删除；当点击向上箭头或向下箭头时，则应该更新该Todo项。但对Adapter而言，没有获得可写的数据库，因此干不了这些事。我参照了Udacity安卓基础课的方法（RecyclerView那章的最后一个练习），给Adapter增加了一个接口，里面有两个方法`onDoneClick()`和`onStatusChangeClick()`；在构造器中传入这个接口，然后在上面`onBindViewHolder()`的代码中省略号的位置放入这两个方法。

```java
holder.button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                listener.onDoneClick(id);
            }
        });
        holder.imageButtonBecomeUrgent.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                listener.onStatusChangeClick(id, TodoEntry.categoryUrgent);
            }
        });
        holder.imageButtonBecomeCold.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                listener.onStatusChangeClick(id, TodoEntry.categoryCold);
            }
        });
    }
```

​	最后要在`MainActivity`中实现这两个方法，完成数据库的操作时，还应该刷新我的RecyclerView：在Adapter中添加方法`swapCursor()`。这个方法会传入新的Cursor以代替旧的那个，然后调用`this.notifyDataSetChanged()`

```java
public void swapCursor(Cursor newCursor) {
        if (cursor != null) cursor.close();
        cursor = newCursor;
        if (newCursor != null) {
            this.notifyDataSetChanged();
        }
    }
```

​	`ColdTodoActivity.java`中，删除一项Todo的方法与主页面不同，是通过左划或右划删除的。这可以通过新增一个`ItemTouchHelper`实现，并绑定到对应的RecyclerView上

```java
new ItemTouchHelper(new ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT) {
            @Override
            public boolean onMove(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, @NonNull RecyclerView.ViewHolder target) {
                return false;
            }

            @Override
            public void onSwiped(@NonNull RecyclerView.ViewHolder viewHolder, int direction) {
                long id = (long) viewHolder.itemView.getTag();
                dbWrite.delete(TodoEntry.TABLE_NAME, TodoEntry._ID + "=" + id, null);
                adapter.swapCursor(getCursorCold());
            }
        }).attachToRecyclerView(coldRecyclerView);
```

​	页面与页面之间的跳转是通过Intent来实现的，每次跳转页面都有可能会对数据库产生操作，所以这里使用了`startActivityForResult()`。当其他页面成功对数据库操作了，就会返回一个`RESULT_OK`。在主页面中重写`onActivityResullt()`方法，只有返回结果为`RESULT_OK`时才调用Adapter的`swapCursor()`方法

```java
@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(resultCode == RESULT_OK){
            adapter.swapCursor(getCursorMainActivity());
        }
    }
```





​	这大概就是所有内容了。不过在运行app时，概率性会遇到常规的Todo项没有向上的箭头，但我还没有找到原因。第一次写博客，觉得我说得有点乱了。

​	最后放上github链接https://github.com/rabbitAiry/TODO

