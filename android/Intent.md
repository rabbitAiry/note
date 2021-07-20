# 4、Intent

- 构建intent的方法之一
  - `new Intent(context,class)` 其中的class指的是显式intent将要启动的页面的class（如`MainActivity.class`）
  
  - `new Intent(Intent.ACTION_VIEW,uri)`
  
  - ```
    `new Intent(Intent.ACTION_VIEW)`
    
    `intent.setData(uri)`
    ```
  
  - 每个activity都扩展自上下文，所以可以使用mainActivity.this来获取context
  
  - 使用startActivity(intent)来切换页面
  
- 向intent添加内容 
  - 使用putExtra（）
  - `intent.putExtra(Intent.EXTRA_TEXT,String)`键 - 值（name - value）
  - 键名允许自定义
  
- 获取intent
  - 获取 - 打开
  - 获取 
  - `Intent intent = getIntent();`
  - 打开(需要检查是否有)
  - `intent.hasExtra(Intent.键)`返回Boolean型
  - `String string = intent.getStringExtra(Intent.键)`
  
- 隐式intent：发送intent等待打开(非指定)
  - 检查是否有app能够打开
  - `if(intent.resolveActivity(getPackageManager())!=null`
  - `{startActivity(intent)}`
  
- Uri
  
  - URI：统一资源标识符（Identifier）
  - URL：统一资源定位符（Loactor）（网址）
  - URI完整形式：
  - `scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]`
    - []内为可选部分
    - scheme描述指向的资源类型：http、ftp、https、geo……
    - authorty部分（第一个大括号）表示登录用户名、主机（站点）
    - 资源路径（path）
    - query部分（？q=）以问号开头
    - #指示路径资源将使用的辅助数据
  - String转Uri：Uri.parse(string)
  - `Uri uri = Uri.parse(string)`
  
- 构建地图Uri(scheme为geo时)

  - 初始化Uri.builder
  - `Uri.Builder = new Uri.Builder()`
  - 添加参数
  - `builder.scheme("geo").path("0,0").appendQueryParameter("q","仲恺农业工程学院")`
  
- 分享文本、图片等媒体类型

  - 媒体类型字符串由类型、子类型、可选参数组成，如：text/html; chaeset=UTF-8

  - 使用ShareCompat.IntentBuilder来共享

  - ```
    String mimeType = "text/plain";
    String title = "标题;
    ShareCompat.IntentBuilder
        .from(this)
        .setType(mimeType)
        .setChooserTitle(title)
        .setText("分享内容")
        .startChooser();
    ```

    

