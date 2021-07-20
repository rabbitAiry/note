# 对Jetpack Compose的第一次试验》SimpleColorSelector

[TOC]

##### 初衷

​	参加了android的Study Jam活动，看了好几天的CodeLab，最后是一个小测验以及完成一个作品，要求要有一个输入框。

​	虽然英语不算太烂，但是想到CodeLab里面到处都是英文的指南，而且到处都是术语，终究是死活看不下去，硬是耗了很长时间，最后就着chrome的全部翻译与原文比对读了几篇。关于作品，要求的输入框必然需要输入点东西然后在界面中能够有反应，想了下以我现在的能力，要求用户输入一串颜色值，然后让整个界面变成输入的颜色，做出这样的app还是可以的，所以app的名字就定了SimpleColorSelector了

​	Compose是安卓推出、用于代替View的ui系统，只能用kotlin编写



##### v 0.1

​	第一个版本做了大部分的工作：整体的ui界面，还有一排点击后无反应的推荐颜色按钮。

》img

​	Compose的默认界面居然连actionBar都没有，但这里直接使用compose提供的material组件topAppBar即可。中心内容中使用Column占满整个屏幕，然后使用权重`modifier = Modifier.weight(1f)`分配界面：上方的文字和按钮下有一列颜色块，是用LazyColumn实现的，中间的颜色块是card，底部是一个能把TextField中的文本清空的FAB按钮。值得一提的是，在Compose里面的属性定义、能力和View相比还是很丰富的，比如：Card的四周圆角不用再先写个drawable了，直接在属性里写个shape就可以了

```kotlin
Card(
    modifier = Modifier
        .fillMaxWidth()
        .height(250.dp)
        .padding(horizontal = 24.dp)
        .weight(3f),
    shape = RoundedCornerShape(12.dp),
    elevation = 4.dp,
//                backgroundColor = Color(0xFF000000+Integer.parseInt("00FFFF",16))
    backgroundColor = resultColor
){
	// ...
}
```

​	又比如，倘若提交的是颜色值不能够正确地解析为颜色，card中会显示“Unable to resolve”。就像vue一样，这些东西可以一并写在ui里了

```kotlin
Text(
    text = if(selectedColor.isNotBlank())"#$selectedColor" else "Unable to resolve",
    style = MaterialTheme.typography.h6,
    color = oppositeColor
)
```

​	compose给我带来最大的感受是对ui的定义，以前我觉得xml写的是ui，java写的是逻辑，ui是不会变化的，ui中包括条件语句等的判断都是逻辑，应该在java中去完成。但现在有了compose后，我觉得只要是用户能够看见的部分都是ui。

​	接着该提到的是TextField，两个参数是必须的：value和onValueChange，前者是TextField上会显示的值，后者是TextField对获取到的值做出的反应，在这里是把值赋给value。因为从输入框获得的值会对整个界面的颜色都做出反应，所以需要状态提升至最高层，也就是和topAppBar同一层。因为这个值会有很多种可能值，所以使用了`mutableStateOf()`，至于括号内的默认值，这个看心情就好~

```kotlin
val baseColor = Color.White
val baseOppositeColor = Color.Black
var selectedColor by remember { mutableStateOf("ffffff") }
var resultColor by remember { mutableStateOf(baseColor) }
var oppositeColor by remember { mutableStateOf(baseOppositeColor)}
```

​	selectedColor的值就是用户输入的值了，用户输入的值有很多种可能，但是只有输入的值是十六进制的颜色时才能够正确的显示出来，所以我的想法是当用户输入完后点击旁边的按钮时，就会尝试能不能够正确解析

```kotlin
onSetColorButtonClick = {
    try {
        var data = Integer.parseInt(selectedColor,16)
        resultColor = Color(0xFF000000+data)
        oppositeColor = Color(0xFFFFFFFF-data)
    }catch (e:NumberFormatException){
        selectedColor = ""
    }
}
```

​	oppositeColor的产生是因为设置不同颜色时，Card中的文字可能会看不见，所以这是要给那些有可能看不清的文字使用的。事后我意识到了这是个错误的想法，当data=0x888888时，resultColor=oppositeColor，这样还是导致了看不清，而且设置文字反色后的观感也不太好。这个问题在v 0.2中解决了。

​	最下方的刷新按钮做的事很简单，单纯是把selectedColor的值清空而已。

​	完成这些后，看到程序中card的值、textfielde的值都会随我的输入改变，这种感觉是挺神奇的，以我目前的能力我都不知道应该怎么在view中实现



##### v 0.2

​	接下来做的事重点就在那一行颜色块了，我把它叫做推荐颜色。这个版本的推荐颜色按钮按下后能够直接把selectedColor和resultColor改为对应值

```kotlin
onGetSuggestColor = {
    selectedColor = it
    val data = Integer.parseInt(selectedColor,16)
    resultColor = Color(0xFF000000+data)
}
```

​	做到这里，有两件事我是困惑的：

1. TextButton要求的onClick()是没有参数的，但在推荐颜色行中，每当我点击颜色，都应该返回一个颜色值，我应该怎么返回这个值呢？

   ```kotlin
   @Composable
   fun TextButton(
       onClick: () -> Unit,
       modifier: Modifier = Modifier,
       // ...
   )
   ```

2. 为resultColor、oppositeColor添加state是必须的吗？明明resultColor仅仅是根据selectedColor而来的，而oppositeColor也是根据resultColor而变化的

   

​	为了解决第一个问题，瞄了下codelab给的代码，才发现方法很简单：上层传下来的方法应该是这种类型的`onGetSuggestColor: (String) -> Unit`，然后在每个颜色块中直接传递参数即可

   ```kotlin
   MyTextButton(
       onClickHere = { onGetSuggestColor(color) },
       // ...
   )
   ```

   	至于第二个问题，oppositeColor确实没必要添加state，但用户输入的值并非每时每刻一定能够转为颜色，所以也resultColor应该还是有自己的状态的

​	顺便也找到了能够让文字不会因为resultColor的变化而看不清的方法了：让文字非黑即白，而决定条件是resultColor的颜色亮度，低于0.5就是偏暗的颜色，这是就该给上白色文字了。这个功能是翻了Color包（`import androidx.compose.ui.graphics.Color`）的属性发现的，不用自己折腾真是太爽了。

```kotlin
val oppositeColor = if(resultColor.luminance()>0.5) Color.Black else Color.White
```

​	最后是把oppositeColor前面的`var`改为`val`。这是在IDE抛出warning后才发现的，改了之后试了下能够正常运行（见v0.2a），才发现这样的oppositeColor已经不算是变量了，它的来源是固定的，没有其他方式，这样就称不得上是变量了。



##### 总结

​	这就是制作这个app的全部流程了，第一次使用compose，最大的感受就是比View要更强大了，能添加逻辑、方便设置背景、而且特别容易能够自制Composable并当做组件复用，还有一点是数据的变化，数据都是自己改的，我什么都没做。

​	但是我连Java都没搞熟啊，用什么kotlin~[doge]

链接https://github.com/rabbitAiry/SimpleColorSelector

