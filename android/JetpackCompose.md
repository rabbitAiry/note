[TOC]

# Jetpack Compose

> 先决条件：kotlin语法（包括lambda）的经验



### basic

##### 创建一个项目

- 新建一个Empty Compose Activity，确保选择的API级别至少为21的*minimumSdkVersion*

- 在gradle中配置该项目

  ```
  buildscript {
      ext {
          compose_version = '1.0.0-beta07'
          kotlin_version = '1.4.32'
      }
      ...
  }
  ```

##### Compose入门

- 可组合函数

```kotlin
@Composable
fun Greeting(name: String) {
   Text(text = "Hello $name!")
}
```

- 在`setContent`中定义布局，调用Composable函数

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            // ...
        }
    }
}
```

- 只需要在`@Preview`注释中标记Composable函数，然后构建项目即可

##### 声明式用户界面

- 设置背景色、padding

  ```kotlin
  @Composable
  fun Greeting(name: String) {
      Surface(color = Color.Yellow){
          Text(text = "Hello $name!",modifier = Modifier.padding(24.dp))
      }
  }
  ```

  - modifier参数：一般的ui元素都会接受modifier参数

  - 需要导入

    ```kotlin
    import androidx.compose.foundation.layout.padding
    import androidx.compose.ui.Modifier
    import androidx.compose.ui.unit.dp
    ```

- 组件复用

  - 以一个Composable函数作为参数（content），返回一个Unit。并使用content

    ```kotlin
    @Composable
    fun MyApp(content: @Composable () -> Unit) {
        BasicsCodelabTheme {
            Surface(color = Color.Yellow) {
                content()
            }
        }
    }
    ```

  - 使用Layout（Column）

    ```kotlin
    import androidx.compose.foundation.layout.Column
    import androidx.compose.material.Divider
    ...
    
    @Composable
    fun MyScreenContent() {
        Column {
            Greeting("Android")
            Divider(color = Color.Black)
            Greeting("there")
        }
    }
    ```

    - Divider：分割线

  - 在ui界面中使用逻辑（循环）

    ```kotlin
    @Composable
    fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
        Column {
            for (name in names) {
                Greeting(name = name)
                Divider(color = Color.Black)
            }
        }
    }
    ```

##### Compose组件的状态

- 通过使用recomposing来达到组件更新

  - 要向可组合对象添加内部状态，请使用mutableStateOf函数
  - 使用`remember`函数使得state不变

  ```kotlin
  import androidx.compose.material.Button
  import androidx.compose.runtime.mutableStateOf
  import androidx.compose.runtime.remember
  ...
  
  @Composable
  fun Counter() {
  
      val count = remember { mutableStateOf(0) }
  
      Button(onClick = { count.value++ }) {
          Text("I've been clicked ${count.value} times")
      }
  }
  @Composable
  fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
      Column {
          // ...
          Divider(color = Color.Transparent, thickness = 32.dp)
          Counter()
      }
  }
  ```

##### 更改布局

- 使用权重，将button放在底部

  - 这里是指内部的Column将占用外部Column除了button的所有空间

  ```kotlin
  import androidx.compose.foundation.layout.fillMaxHeight
  ...
  
  @Composable
  fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
      val counterState = remember { mutableStateOf(0) }
  
      Column(modifier = Modifier.fillMaxHeight()) {
          Column(modifier = Modifier.weight(1f)) {
              for (name in names) {
                  Greeting(name = name)
                  Divider(color = Color.Black)
              }
          }
          Counter(
              count = counterState.value,
              updateCount = { newCount ->
                  counterState.value = newCount
              }
          )
      }
  }
  ```

- 使得button对点击事件做出相应（ui）

  ```kotlin
  import androidx.compose.material.ButtonDefaults
  ...
  
  @Composable
  fun Counter(count: Int, updateCount: (Int) -> Unit) {
      Button(
          onClick = { updateCount(count+1) },
          colors = ButtonDefaults.buttonColors(
              backgroundColor = if (count > 5) Color.Green else Color.White
          )
      ) {
          Text("I've been clicked $count times")
      }
  }
  ```

- 显示成千个数字

  - 使用lambda函数指代list的index
  - 使用LazyColumn：相当于RecyclerView的划动视图，但不回收子项目
    - 这里相当于是使用循环完成的`items(items = names)`
  - 最后，把MyScreenContent中的内column用NameList替换
  
  ```kotlin
  import androidx.compose.foundation.lazy.LazyColumn
  import androidx.compose.foundation.lazy.items
  ...
  
  @Composable
  fun NameList(names: List<String>, modifier: Modifier = Modifier) {
     LazyColumn(modifier = modifier) {
         items(items = names) { name ->
             Greeting(name = name)
             Divider(color = Color.Black)
         }
     }
  }
  ```

##### 给列表加特效

- 为每一项选中时变色做准备，并为使用modifier添加多个样式

  - 此时，当子项目划出页面时，子项目不再得到track，状态isSelected会变回false。将isSelected状态提升至namelist上可以解决这个问题

  ```kotlin
  @Composable
  fun Greeting(name: String) {
      var isSelected by remember{ mutableStateOf(false)}
      val backgroundColor by animateColorAsState(targetValue = if(isSelected)Color.Red else Color.Transparent)
  
      Text(
          text = "Hello $name!",
          modifier = Modifier
              .padding(24.dp)
              .background(color = backgroundColor)
              .clickable(onClick = { isSelected = !isSelected })
      )
  }
  ```

- 使用MaterialDesign

  ```kotlin
  @Composable
  fun MyApp(content: @Composable () -> Unit) {
      MaterialTheme{
          // ...
      }
  }
  
  @Composable
  fun Greeting(name: String) {
      // ...
      Text(
          // ...
          style = MaterialTheme.typography.h1
      )
  }
  ```

- 制作自己的主题方格

  ```kotlin
  import androidx.compose.foundation.isSystemInDarkTheme
  import androidx.compose.runtime.Composable
  
  @Composable
  fun BasicsCodelabTheme(
      darkTheme: Boolean = isSystemInDarkTheme(),
      content: @Composable () -> Unit
  ) {
  
      // TODO 
  }
  
  ```

  

[TOC]

### layouts

##### 继续设置样式

- 设置字重和字体透明度

  - 通过传递数据给CompositionLocalProvider可以获得一些属性

  ```kotlin
  @Composable
  fun PhotographerCard() {
      Column {
          Text("Alfred Sisley", fontWeight = FontWeight.Bold)
          // LocalContentAlpha is defining opacity level of its children
          CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
              Text("3 minutes ago", style = MaterialTheme.typography.body2)
          }
      }
  }
  ```

- 展示图片框（placeholder）

  - 使用Surface并指定一个圆形
  - 在图片加载时可以看到placeholder

  ```kotlin
  Surface(
      modifier = Modifier.size(50.dp),
      shape = CircleShape,
      color = MaterialTheme.colors.onSurface.copy(alpha = 0.2f)
  ) {
      // Image goes here
  }
  ```

- 字体居中、加点左padding

  ```kotlin
  Column(
      modifier = Modifier
          .padding(start = 8.dp)
          .align(Alignment.CenterVertically)
  ) {
      //...
  }
  ```

- 自定义你的composable

  - 第一个参数应该为modifier

  ```kotlin
  @Composable
  fun PhotographerCard(modifier: Modifier = Modifier) {
      Row(modifier) { ... }
  }
  ```

- 要留意modifiers串联起来（chain）的顺序

  - 先padding后clickable：此时会出现padding区域不可被点击（无响应）

    ```kotlin
    @Composable
    fun PhotographerCard(modifier: Modifier = Modifier) {
        Row(modifier
            .padding(16.dp)
            .clickable(onClick = { /* Ignoring onClick */ })
        ) {
            ...
        }
    }
    ```

##### compose组件中加入布局（Slot APIs）

- 例如：为一个button添加logo与文字

  - 旧的表达方式可读性差

    ```kotlin
    Button(
        text = "Button",
        icon: Icon? = myIcon,
        textStyle = TextStyle(...),
        spacingBetweenIconAndText = 4.dp,
        ...
    )
    ```

  - 可以添加布局

    ```kotlin
    Button {
        Row {
            MyImage()
            Spacer(4.dp)
            Text("Button")
        }
    }
    ```

##### Material部件

- 使用Scaffold：最高级别的compose组件（composable）

  - 传递padding

    ```kotlin
    @Composable
    fun LayoutsCodelab() {
        Scaffold { innerPadding ->
            BodyContent(Modifier.padding(innerPadding))
        }
    }
    
    @Composable
    fun BodyContent(modifier: Modifier = Modifier) {
        Column(modifier = modifier) {
            Text(text = "Hi there!")
            Text(text = "Thanks for going through the Layouts codelab")
        }
    }
    ```

  - 设置标题栏（会应用默认颜色）

    ```kotlin
    Scaffold(
            topBar = {
                TopAppBar(
                    title = {
                        Text(text = "LayoutsCodelab")
                    }
                )
            }
        ) { 
        // ...
        }
    ```

  - 给标题栏加图标（使用MaterialDesign自带图标）

    ```kotlin
    topBar = {
        TopAppBar(
            title = {
                Text(text = "LayoutsCodelab")
            },
            actions = {
                IconButton(onClick = { /* doSomething() */ }) {
                    Icon(Icons.Filled.Favorite, contentDescription = null)
                }
            }
        )
    }
    ```

- 使用更多material icon：需要增加依赖

  ```
  dependencies {
    ...
    implementation "androidx.compose.material:material-icons-extended:$compose_version"
  }
  ```

##### 列表lists

- 制作一个可以滚动的列表（使用Column而非LazyColumn。这会渲染所有项目，影响性能）

  - 使用`repeat()`来循环
  - 使用`verticalScroll`modifier使得可以滚动：将位置保存在scrollState中

  ```kotlin
  @Composable
  fun SimpleList() {
      // We save the scrolling position with this state that can also 
      // be used to programmatically scroll the list
      val scrollState = rememberScrollState()
  
      Column(Modifier.verticalScroll(scrollState)) {
          repeat(100) {
              Text("Item #$it")
          }
      }
  }
  ```

- 使用LazyColumn：只渲染可见项目

  - 在preview中看起来都一样

  ```kotlin
  @Composable
  fun LazyList() {
      // We save the scrolling position with this state that can also 
      // be used to programmatically scroll the list
      val scrollState = rememberLazyListState()
  
      LazyColumn(state = scrollState) {
          items(100) {
              Text("Item #$it")
          }
      }
  }
  ```

- 远程获取并在列表中展示图片 使用第三方库Coil或Glide

  - 不使用第三方库的步骤：下载图片》译码为bitmap》在Image中将其渲染。使用第三方库可以一步到位

  - 添加依赖、联网权限

    - 使用coil出错，未知原因

    ```
    implementation "com.google.accompanist:accompanist-glide:0.10.0"
    ```

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

  - 子项目。最后在lazycolumn中调用

    ```kotlin
    @Composable
    fun ImageListItem(index: Int) {
        Row(verticalAlignment = Alignment.CenterVertically) {
    
            Image(
                painter = rememberGPainter(
                    request = "https://developer.android.com/images/brand/Android_Robot.png"
                ),
                contentDescription = "Android Logo",
                modifier = Modifier.size(50.dp)
            )
            Spacer(Modifier.width(10.dp))
            Text("Item #$index", style = MaterialTheme.typography.subtitle1)
        }
    }
    ```

- 控制划动

  - 为了防止滚动过程中会卡顿，使用了协程(Coroutine)的CoroutineScope
  - 使用scrollState.animateScrollToItem(0)控制位置

  ```kotlin
  // 替代SimpleList
  @Composable
  fun ScrollingList() {
      val listSize = 100
      // We save the scrolling position with this state
      val scrollState = rememberLazyListState()
      // We save the coroutine scope where our animated scroll will be executed
      val coroutineScope = rememberCoroutineScope()
  
      Column {
          Row {
              Button(onClick = {
                  coroutineScope.launch {
                      // 0 is the first item index
                      scrollState.animateScrollToItem(0)
                  }
              }) {
                  Text("Scroll to the top")
              }
  
              Button(onClick = {
                  coroutineScope.launch {
                      // listSize - 1 is the last index of the list
                      scrollState.animateScrollToItem(listSize - 1)
                  }
              }) {
                  Text("Scroll to the end")
              }
          }
  
          LazyColumn(state = scrollState) {
              items(listSize) {
                  ImageListItem(it)
              }
          }
      }
  }
  ```

  

##### 自定义布局

> 在View system中，自定义布局需要继承自ViewGroup，但在Compose中，只需要创建一个使用Layout Composable的函数即可

- Compose UI只允许为子元素测量一次

- 使用layout modifier》添加属性：在Modifier中添加新属性，使得padding的计算为顶部到字的baseline

  - 架构：有两个lambda参数
    - measurable 被测量和放置的子元素
    - constraints 子元素的宽、高最大值与最小值

  ```kotlin
  fun Modifier.customLayoutModifier(...) = Modifier.layout { measurable, constraints ->
    ...
  })
  ```

  - 先是测量这个部件大小，通过调用`measure`方法，这会返回一个`Placeable`，用于定位
  - 然后计算这个部件的位置，通过调用`layout(width,height)`
    - 最后通过调用`placeable.placeRelative(x,y)`以展示这个部件

  ```kotlin
  @Composable
  fun Modifier.firstBaselineToTop(
      firstBaselineToTop:Dp
  ) = this.then(
      layout { measurable, constraints ->
          val placeable = measurable.measure(constraints)
  
          check(placeable[FirstBaseline]!=AlignmentLine.Unspecified)
          val firstBaseline = placeable[FirstBaseline]
  
          val placeableY = firstBaselineToTop.roundToPx() - firstBaseline
          val height = placeable.height + placeableY
          layout(placeable.width,height){
              placeable.placeRelative(0,placeableY)
          }
      }
  )
  ```

  - 调用这个属性

  ```kotlin
  @Composable
  fun TextWithPaddingToBaselinePreview() {
    LayoutsCodelabTheme {
      Text("Hi there!", Modifier.firstBaselineToTop(32.dp))
    }
  }
  ```

- 使用Layout Coposable》

  - 可以控制测量、摆放方式
  - 架构：需要两个参数
    - modifier
    - content

  ```kotlin
  @Composable
  fun CustomLayout(
      modifier: Modifier = Modifier,
      // custom layout attributes 
      content: @Composable () -> Unit
  ) {
      Layout(
          modifier = modifier,
          content = content
      ) { measurables, constraints ->
          // measure and position children given constraints logic here
      }
  }
  ```

  - 先测量大小，再设置位置，但这次用的是列表（多个子元素）
  
  ```kotlin
  @Composable
  fun MyOwnColumn(
      modifier: Modifier = Modifier,
      content: @Composable () -> Unit
  ) {
      Layout(
          modifier = modifier,
          content = content
      ) { measurables, constraints ->
          // Don't constrain child views further, measure them with given constraints
          // List of measured children
          val placeables = measurables.map { measurable ->
              // Measure each child
              measurable.measure(constraints)
          }
  
          // Track the y co-ord we have placed children up to
          var yPosition = 0
  
          // Set the size of the layout as big as it can
          layout(constraints.maxWidth, constraints.maxHeight) {
              // Place children in the parent layout
              placeables.forEach { placeable ->
                  // Position item on the screen
                  placeable.placeRelative(x = 0, y = yPosition)
  
                  // Record the y co-ord placed up to
                  yPosition += placeable.height
              }
          }
      }
  }
  
  ```



##### 自定义布局：复杂点的

> 自定义一个StaggeredGrid，用于制作一个横向瀑布流的多按钮（checkbox）

- 从模板开始制作布局，依然先测量大小，再放置

  ```kotlin
  @Composable
  fun StaggeredGrid(
      modifier:Modifier=Modifier,
      rows:Int=3,
      content:@Composable () -> Unit
  ){
      Layout(
  //        children = children,
          modifier = modifier,
          content = content
      ){measurables,constraints ->
          val rowWidths = IntArray(rows){0}
          val rowHeights = IntArray(rows){0}
          val placeables = measurables.mapIndexed{index,measurable ->
              val placeable = measurable.measure(constraints)
              val row = index % rows
              rowWidths[row] += placeable.width
              rowHeights[row] = max(rowHeights[row],placeable.height)
              placeable
          }
  
          val width = rowWidths.maxOrNull()
              ?.coerceIn(constraints.minWidth.rangeTo(constraints.maxWidth))?:constraints.minWidth
          val height = rowHeights.sumBy{ it }
              .coerceIn(constraints.minHeight.rangeTo(constraints.maxHeight))
  
          val rowY = IntArray(rows){0}
          for(i in 1 until rows){
              rowY[i] = rowY[i-1] + rowHeights[i-1]
          }
  
          layout(width,height){
              val rowX = IntArray(rows){0}
              placeables.forEachIndexed{index,placeable ->
                  val row = index%rows
                  placeable.placeRelative(
                      x = rowX[row],
                      y = rowY[row]
                  )
                  rowX[row] += placeable.width
              }
          }
  
      }
  }
  ```

- 调用这个布局

  ```kotlin
  val topics = listOf(
      "Arts & Crafts", "Beauty", "Books", "Business", "Comics", "Culinary",
      "Design", "Fashion", "Film", "History", "Maths", "Music", "People", "Philosophy",
      "Religion", "Social sciences", "Technology", "TV", "Writing"
  )
  
  @Composable
  fun BodyContentGrid(modifier: Modifier = Modifier){
      StaggeredGrid(modifier = modifier) {
          for(topic in topics){
              // ...
          }
      }
  }
  ```

##### Layout modifiers的原理

- padding modifier的源码

  - width和height是加上padding后的值

  ```kotlin
  // How to create a modifier
  @Stable
  fun Modifier.padding(all: Dp) =
      this then PaddingModifier(start = all, top = all, end = all, bottom = all, rtlAware = true)
  
  // Implementation detail
  private class PaddingModifier(
      start: Dp = 0.dp,
      top: Dp = 0.dp,
      end: Dp = 0.dp,
      bottom: Dp = 0.dp,
      rtlAware: Boolean
  ) : LayoutModifier {
      override fun MeasureScope.measure(
          measurable: Measurable,
          constraints: Constraints
      ): MeasureScope.MeasureResult {
          val horizontal = start.roundToPx() + end.roundToPx()
          val vertical = top.roundToPx() + bottom.roundToPx()
  
          val placeable = measurable.measure(constraints.offset(-horizontal, -vertical))
  
          val width = constraints.constrainWidth(placeable.width + horizontal)
          val height = constraints.constrainHeight(placeable.height + vertical)
          return layout(width, height) {
              if (rtlAware) {
                  placeable.placeRelative(start.roundToPx(), top.roundToPx())
              } else {
                  placeable.place(start.roundToPx(), top.roundToPx())
              }
          }
      }
  }
  ```

- modifier的顺序确实影响效果

  - 会影响的内容：比如布局长宽，可以用背景和内容来验证

##### ConstraintLayout

- 在gradle中添加

  ```
  implementation "androidx.constraintlayout:constraintlayout-compose:1.0.0-alpha07"
  ```

- 初始化组件之间联系（reference）的方法：`createRefs()`

- 建立与parent联系的方法：`linkTo()`

- 指定自身为参考物的方法：`constrainAs()`

- 例》Text在Button下方

  ```kotlin
  @Composable
  fun ConstraintLayoutContent() {
      ConstraintLayout {
  
          // Create references for the composables to constrain
          val (button, text) = createRefs()
  
          Button(
              onClick = { /* Do something */ },
              // Assign reference "button" to the Button composable
              // and constrain it to the top of the ConstraintLayout
              modifier = Modifier.constrainAs(button) {
                  top.linkTo(parent.top, margin = 16.dp)
              }
          ) {
              Text("Button")
          }
  
          // Assign reference "text" to the Text composable
          // and constrain it to the bottom of the Button composable
          Text("Text", Modifier.constrainAs(text) {
              top.linkTo(button.bottom, margin = 16.dp)
          })
      }
  }
  
  @Preview
  @Composable
  fun ConstraintLayoutContentPreview() {
      LayoutsCodelabTheme {
          ConstraintLayoutContent()
      }
  }
  ```

  - 让文字居中：use the `centerHorizontallyTo` function 

    ```kotlin
    Text("Text", Modifier.constrainAs(text) {
                top.linkTo(button.bottom, margin = 16.dp)
                // Centers Text horizontally in the ConstraintLayout
                centerHorizontallyTo(parent)
            })
    ```

- 使用更多ConstraintLayout属性

  - barrier：自适应的布局无形的分割线（比如中文切换为英文时，布局也不会改变）

  ```kotlin
  @Composable
  fun LargeConstraintLayout() {
      ConstraintLayout {
          // ...
          val barrier = createEndBarrier(button1, text)
          Button(
              onClick = { /* Do something */ },
              modifier = Modifier.constrainAs(button2) {
                  top.linkTo(parent.top, margin = 16.dp)
                  start.linkTo(barrier)
              }
          ) { 
              Text("Button 2") 
          }
      }
  }
  ```

  - dimension：位置不够时自动换行。例子中修改了width

  ```kotlin
  @Composable
  fun LargeConstraintLayout() {
      ConstraintLayout {
          val text = createRef()
  
          val guideline = createGuidelineFromStart(0.5f)
          Text(
              "This is a very very very very very very very long text",
              Modifier.constrainAs(text) {
                  linkTo(guideline, parent.end)
                  width = Dimension.preferredWrapContent
              }
          )
      }
  }
  ```

- 解耦api（Decoupled API）

##### Intrinsics(固有）

- intrinsics可以让你在唯一一次测量前选择测量子项目的大小的方式

  - 下例为获取子项目高度的最小值

  ```kotlin
  Row(modifier = modifier.height(IntrinsicSize.Min)) {
  	// ...
  }
  ```




[TOC]

### state

- state就是在运行过程中会改变的变量

##### 了解单向数据流

- UI更新循环

  - state is updated in response to events
  - Event》Update State》Display State

- 使用单向数据流，而不是非结构化的state

  （如：将state移到livedata中，而不是放在Activity）

  - 这样做的好处是

    - 便于测试：因为ui的状态与view分离
    - 简化代码：逻辑部分与ui分离
    - 不会因忘记而只更新了state或ui

  - 打字跟踪的例子

    - 使用非结构化的state

    ```kotlin
    class HelloCodelabActivity : AppCompatActivity() {
    
       private lateinit var binding: ActivityHelloCodelabBinding
       var name = ""
    
       override fun onCreate(savedInstanceState: Bundle?) {
           /* ... */
           binding.textInput.doAfterTextChanged {text ->
               name = text.toString()
               updateHello()
           }
       }
    
       private fun updateHello() {
           binding.helloText.text = "Hello, $name"
       }
    }
    ```

    - 使用单向数据流

    ```kotlin
    class HelloCodelabViewModel: ViewModel() {
    
       private val _name = MutableLiveData("")
       val name: LiveData<String> = _name
    
       fun onNameChanged(newName: String) {
           _name.value = newName
       }
    }
    
    class HelloCodeLabActivityWithViewModel : AppCompatActivity() {
       private val helloViewModel by viewModels<HelloCodelabViewModel>()
    
       override fun onCreate(savedInstanceState: Bundle?) {
           /* ... */
    
           binding.textInput.doAfterTextChanged {
               helloViewModel.onNameChanged(it.toString()) 
           }
    
           helloViewModel.name.observe(this) { name ->
               binding.helloText.text = "Hello, $name"
           }
       }
    }
    ```
    
  - 单向数据流的意义：Activity将event告知ViewModel，ViewModel返还state给Activity（发布event，获得state）(events flow *up* and state flows *down*)

##### Compose的记忆memory

- localContentColor：为icon和typography设置颜色

- 复用组件也是一个recomposition的过程（如LazyColumn）

- Compose树

  - Compose会生成树：第一次运行时，Compose构造一棵树；重组时，更新树

- 副作用side-effect

  - 副作用是在 非composable运行过程中出现的视觉效果变化

    ```
    比如在第一个实验的LazyColumn中，被选中变红色背景的项目，在划动后不再为红色背景；
    又比如这个实验中，每次添加新的todo到列表时，icon的颜色都会变化
    ```

  - 如：在ViewModel中使用state、调用Random.nextInt()、或者进行数据库操作，这些都是副作用

  - 重组过程理应不出现side-effect

- 让composable拥有memory

  - remember提供了composable的memory
  
    ```kotlin
    remember(todo.id) { randomTint() }
    // 小括号内是关键参数，是给remember提供位置
    // 大括号内是lambda表达式，用于计算新的被remembered的值
    // 第一次运行时会记住该值，之后则会记住该值
    ```
  
  - 使用remember的composable 的remember值 会在移除时被遗忘（比如LazyColumn中划出了屏幕后）
  
  - 短暂的动画效果就适合在LazyColumn的子中使用remember，但其他就不适合

##### Compose的state

- 输入文字：在View系统中的EditText相当于Compose的TextField，但是TextField可以选择是有状态或者无状态的

- mutableStateOf

  - 自动为`MutableState`对象生成 getter 和 setter，是内置于 compose 中的可观察状态者，它会在更新时告诉 compose

  - 添加导入

    ```kotlin
    import androidx.compose.runtime.mutableStateOf
    import androidx.compose.runtime.getValue
    import androidx.compose.runtime.setValue
    ```

  - 声明MutableState的三种方式

    ```java
    val state = remember { mutableStateOf(default) }
    var value by remember { mutableStateOf(default) }
    val (value, setValue) = remember { mutableStateOf(default) }
    ```

- 有状态的TextField

  - 使用mutableStateOf为它保存状态，使用remember记住有这个状态
  - 虽然有状态的TextField能够重组ui，看到输入的文字，但是这个状态（输入的文字）不能被其他组件共享、获取

  ```kotlin
  @Composable
  fun SimpleFilledTextField() {
      var text by remember { mutableStateOf("Hello") }
  
      TextField(
          value = text,
          onValueChange = { text = it },
          label = { Text("Label") }
      )
  }
  ```

- 无状态的TextField

  - 使用状态提升（state hoisting）可以解决上述问题，状态提升至父Composable，使得子变成无状态的
  - 状态提升通常用到这两个参数：value、onValueChange
  - `by`是 Kotlin 中的属性委托语法
  - 状态提升带来的特性：单一来源、封装的（只有上层能够修改状态）、可共享、可接受或忽略变化、解耦（自动更新）

  ```kotlin
  @Composable
  fun TodoItemInput(onItemComplete: (TodoItem) -> Unit) {
     val (text, setText) = remember { mutableStateOf("") }
     Column {
         Row(Modifier
             .padding(horizontal = 16.dp)
             .padding(top = 16.dp)
         ) {
             TodoInputTextField(
                 text = text,
                 onTextChange = setText,
                 modifier = Modifier
                     .weight(1f)
                     .padding(end = 8.dp)
             )
             TodoEditButton(
                 onClick = { /* todo */ },
                 text = "Add",
                 modifier = Modifier.align(Alignment.CenterVertically)
             )
         }
     }
  }
  @Composable
  fun TodoInputTextField(text: String, onTextChange: (String) -> Unit, modifier: Modifier) {
     // ...
  }
  ```

- 设置button：发送event`onItemComplete()`、清空文本、文本非空时允许添加

  ```kotlin
  TodoEditButton(
     onClick = {
         onItemComplete(TodoItem(text)) // send onItemComplete event up
         setText("") // clear the internal text
     },
     text = "Add",
     modifier = Modifier.align(Alignment.CenterVertically),
     enabled = text.isNotBlank() // enable if text is not blank
  )
  ```

##### 基于state的动态数据

- 应用场景：TextField非空时展开Row，点击Row中的button以确认Todo属性

  - 新增Row是否可见的布尔型变量（而不是状态，两个状态任意造成不同步）

  - 新增状态icon用于保存当前选定的图标

    ```kotlin
       val (icon, setIcon) = remember { mutableStateOf(TodoIcon.Default)}
       val iconsVisible = text.isNotBlank()
    ```

- 重组可以根据数据变化改变树的结构

- 使用 imeAction 完成设计（输入法编辑器）

  - 键盘的submit按钮要和界面上的submit按钮发挥同一作用，可以通过复制代码完成，亦或是：使用lambda函数`submit`

    ```kotlin
    val submit = {
        // ...
    }
    onImeAction = submit
    // ...
    onClick = submit
    ```

- TextField提供对键盘输入的参数

  - `keyboardOptions` - 用于启用显示完成 IME 操作
  - `keyboardActions`- 用于指定响应触发的特定 IME 操作而触发的操作 - 在我们的例子中，一旦按下 Done，我们希望`submit`被调用并隐藏键盘
  - 控制软键盘：使用`LocalSoftwareKeyboardController.current`，这是个实验性API，还必须用`@OptIn(ExperimentalComposeUiApi::class)`

  ```kotlin
  @OptIn(ExperimentalComposeUiApi::class)
  @Composable
  fun TodoInputText(
      // ...
  ) {
      val keyboardController = LocalSoftwareKeyboardController.current
      TextField(
          // ...
          keyboardOptions = KeyboardOptions.Default.copy(imeAction = ImeAction.Done),
          keyboardActions = KeyboardActions(onDone = {
              onImeAction()
              keyboardController?.hideSoftwareKeyboard()
          }),
          modifier = modifier
      )
  }
  ```

##### 在ViewModel中使用State

> 这一小节包含了四步
>
> 状态分离》
>
> 在ViewModel中使用State（配置ViewModel）》
>
> 在ViewModel中测试State》
>
> 复用无状态组合》

- 应用场景：当点击Todo项目时可进入修改状态

- 状态分离》提取无状态的composables

  - 将有状态的Composable转换为无状态的Composable。无状态的Composable上层/入口都在有状态的Composable中

  ```kotlin
  @Composable
  fun TodoItemEntryInput(onItemComplete: (TodoItem) -> Unit) {
     val (text, setText) = remember { mutableStateOf("") }
     val (icon, setIcon) = remember { mutableStateOf(TodoIcon.Default)}
     val iconsVisible = text.isNotBlank()
     val submit = {
         onItemComplete(TodoItem(text, icon))
         setIcon(TodoIcon.Default)
         setText("")
     }
     TodoItemInput(
         text = text,
         onTextChange = setText,
         icon = icon,
         onIconChange = setIcon,
         submit = submit,
         iconsVisible = iconsVisible
     )
  }
  
  @Composable
  fun TodoItemInput(
     // ...
  ) {
     Column {
         // ...
     }
  }
  ```

- 配置ViewModel

  - ViewModel可以减轻很多作为Observer的工作量

  - 增删

    ```kotlin
    var todoItems: List<TodoItem> by mutableStateOf(listOf())
       private set
    
    // event: addItem
    fun addItem(item: TodoItem) {
        todoItems = todoItems + listOf(item)
    }
    
    // event: removeItem
    fun removeItem(item: TodoItem) {
       // toMutableList makes a mutable copy of the list we can edit, then
       // assign the new list to todoItems (which is still an immutable list)
       todoItems = todoItems.toMutableList().also {
           it.remove(item)
       }
    }
    ```

  - 在Activity中使用ViewModel

    ```kotlin
    @Composable
    private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
       TodoScreen(
           items = todoViewModel.todoItems,
           onAddItem = todoViewModel::addItem,
           onRemoveItem = todoViewModel::removeItem
       )
    }
    ```

  - 定义编辑器状态

    ```kotlin
    class TodoViewModel : ViewModel() {
    
       // private state
       private var currentEditPosition by mutableStateOf(-1)
    
       // state
       var todoItems by mutableStateOf(listOf<TodoItem>())
           private set
    
       // state
       val currentEditItem: TodoItem?
           get() = todoItems.getOrNull(currentEditPosition)
    
       // ..
    ```

  - 定义编辑器事件

    ```kotlin
    class TodoViewModel : ViewModel() {
       ...
    
       // event: onEditItemSelected
       fun onEditItemSelected(item: TodoItem) {
          currentEditPosition = todoItems.indexOf(item)
       }
    
       // event: onEditDone
       fun onEditDone() {
          currentEditPosition = -1
       }
    
       // event: onEditItemChange
       fun onEditItemChange(item: TodoItem) {
          val currentItem = requireNotNull(currentEditItem)
          require(currentItem.id == item.id) {
              "You can only change an item with the same id as currentEditItem"
          }
    
          todoItems = todoItems.toMutableList().also {
              it[currentEditPosition] = item
          }
       }
    }
    ```

  - 删除项目时结束编辑

    ```kotlin
    // event: removeItem
    fun removeItem(item: TodoItem) {
       todoItems = todoItems.toMutableList().also { it.remove(item) }
       onEditDone() // don't keep the editor open when removing items
    }
    ```

- 在ViewModel中测试状态

  - 在`test/`目录中打开`TodoViewModelTest.kt`并添加删除项目的测试
    - before：创建一个新的ViewModel，然后将两个项目添加到TodoItems
    - during：测试的方法
    - after：使用断言，断言列表仅包含第二项
  - 点击运行，若无报错即断言正确
  - 如果写入`MutableState<T>`是在另一个线程上执行的，它们将不会从您的测试中立即可见

  ```kotlin
  import com.example.statecodelab.util.generateRandomTodoItem
  import com.google.common.truth.Truth.assertThat
  import org.junit.Test
  
  class TodoViewModelTest {
  
     @Test
     fun whenRemovingItem_updatesList() {
         // before
         val viewModel = TodoViewModel()
         val item1 = generateRandomTodoItem()
         val item2 = generateRandomTodoItem()
         viewModel.addItem(item1)
         viewModel.addItem(item2)
  
         // during
         viewModel.removeItem(item1)
  
         // after
         assertThat(viewModel.todoItems).isEqualTo(listOf(item2))
     }
  }
  ```

- 重用无状态组合

  - 为Composable组件传入ViewModel的三个新事件以及当前正在编辑的项目（加上？）

    ```kotlin
    @Composable
    fun TodoScreen(
        items: List<TodoItem>,
        currentlyEditing: TodoItem?,
        onAddItem: (TodoItem) -> Unit,
        onRemoveItem: (TodoItem) -> Unit,
        onStartEdit: (TodoItem) -> Unit,
        onEditItemChange: (TodoItem) -> Unit,
        onEditDone: () -> Unit
    ) {
    	// ...
    }
    ```

  - 创建一个复用Composable：直接套用无状态的TodoItemInput

    - 注意：用的是等号

    ```kotlin
    @Composable
    fun TodoItemInlineEditor(
       item: TodoItem,
       onEditItemChange: (TodoItem) -> Unit,
       onEditDone: () -> Unit,
       onRemoveItem: () -> Unit
    ) = TodoItemInput(
       text = item.task,
       onTextChange = { onEditItemChange(item.copy(task = it)) },
       icon = item.icon,
       onIconChange = { onEditItemChange(item.copy(icon = it)) },
       submit = onEditDone,
       iconsVisible = true
    )
    ```

  - 直接在LazyColumn上使用：若有修改，则显示用于修改的Composable

    - `LazyColumn`不像`RecyclerView`，它不需要做任何回收，不可见即丢弃

    ```kotlin
    // fun TodoScreen()
    // ...
    LazyColumn(
       modifier = Modifier.weight(1f),
       contentPadding = PaddingValues(top = 8.dp)
    ) { 
     items(items) { todo ->
       if (currentlyEditing?.id == todo.id) {
           TodoItemInlineEditor(
               item = currentlyEditing,
               onEditItemChange = onEditItemChange,
               onEditDone = onEditDone,
               onRemoveItem = { onRemoveItem(todo) }
           )
       } else {
           TodoRow(
               todo,
               { onStartEdit(it) },
               Modifier.fillParentMaxWidth()
           )
       }
    }
    // ...
    ```

##### 使用插槽slot来传递Composable

- Slot可以作为composable的参数，相当于把一个Composable作为参数。调用方法形如 `@Composable () -> Unit`

- 调用方：

  ```kotlin
  @Composable
  fun TodoItemInput(
     // ...
     buttonSlot: @Composable() () -> Unit,
  ) {
     Column {
         // ...
         Box(Modifier.align(Alignment.CenterVertically)) { buttonSlot() }
     }
  }
  ```

- 传入方

  - 将slot写在 { } 内，这是尾随 lambda 语法
  - 亦或者按照平时的写法，写在（ ）内

  ```kotlin
  TodoItemInput(
     // other Parameters
  ) {
    // Slot
     TodoEditButton(onClick = submit, text = "Add", enabled = text.isNotBlank())
  }
  ```



[TOC]

### 动画

##### 简单的过渡动画

- 应用场景：简单属性切换

  - 使用`animate*AsState`API，委托变化。仅适用于简单变化，*可以为size、color、value等

  ```kotlin
  // no animation
  val backgroundColor = if (tabPage == TabPage.Home) Purple100 else Green300
  
  // with animation
  val backgroundColor by animateColorAsState(if (tabPage == TabPage.Home) Purple100 else Green300)
  ```

- 应用场景：组件的可见与否

  ```kotlin
  // no animation
  if (extended) {
  		// ...
  }
  
  // with animation
  AnimatedVisibility(extended) {
    	// ...
  }
  ```

- 应用场景：组件的可见与否》配置动画 的展开、收起方向

  - enter配置进场、exit配置退场
  - animationSpec指定了动画值如何随时间改变

  ```kotlin
  AnimatedVisibility(
      visible = shown,
      enter = slideInVertically(
          // Enters by sliding down from offset -fullHeight to 0.
          initialOffsetY = { fullHeight -> -fullHeight },
          animationSpec = tween(durationMillis = 150, easing = LinearOutSlowInEasing)
      ),
      exit = slideOutVertically(
          // Exits by sliding up from offset 0 to -fullHeight.
          targetOffsetY = { fullHeight -> -fullHeight },
          animationSpec = tween(durationMillis = 250, easing = FastOutLinearInEasing)
      )
  ) {
    // ...
  }
  ```

- 应用场景：组件长宽变化时的过度动画

  - 只需在modifier里加上`.animateContentSize()`即可

  ```kotlin
  Column(
      modifier = Modifier
          .fillMaxWidth()
          .padding(16.dp)
          .animateContentSize()
  ) {
      // ... the title and the body
  }
  ```


##### 多值动画

- 同时给多个值设置同一个动画，可以使用`Transition`

  ```kotlin
  // no animation
  val indicatorLeft = tabPositions[tabPage.ordinal].left
  val indicatorRight = tabPositions[tabPage.ordinal].right
  val color = if (tabPage == TabPage.Home) Purple700 else Green800
  
  // with animation
  val transition = updateTransition(tabPage)
  val indicatorLeft by transition.animateDp { page ->
      tabPositions[page.ordinal].left
  }
  val indicatorRight by transition.animateDp { page ->
      tabPositions[page.ordinal].right
  }
  val color by transition.animateColor { page ->
      if (page == TabPage.Home) Purple700 else Green800
  }
  ```

- 自定义动画行为

  - 靠近目的地的边比另一边移动得更快来实现指标的弹性效果
  - 可以使用 `isTransitioningTo`的lambda（`transitionSpec`中的中缀函数）来确定状态变化的方向

##### 重复动画

- 应用场景：刷新某个部件后的等待动画，通过修改不透明度实现重复动画

  ```kotlin
  // no animation
  val alpha = 1f
  
  // with repeated animation
  val infiniteTransition = rememberInfiniteTransition()
  val alpha by infiniteTransition.animateFloat(
      initialValue = 0f,
      targetValue = 1f,
      animationSpec = infiniteRepeatable(
          animation = keyframes {
              durationMillis = 1000
              0.7f at 500
          },
          repeatMode = RepeatMode.Reverse
      )
  )
  ```

##### 手势动画

- 应用场景：划动删除
  - 速度不够快时不能删

[TOC]

### 迁移到Jetpack Compose

- 在`build.gradle (Module:...`配置：

  - 添加上`compose true`

  ```
  android {
      ...
      kotlinOptions {
          jvmTarget = '1.8'
          useIR = true
      }
      buildFeatures {
          ...
          compose true
      }
      composeOptions {
          kotlinCompilerExtensionVersion rootProject.composeVersion
      }
  }
  ```

  