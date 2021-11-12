# 连接服务端--Firebase

[TOC]

- 使用服务端推送而不是客户端轮询
  - 每隔30s查询一次对电池消耗大
  - 每个1h查询一次不及时
  - 在代码上，应该根据推送得到的消息做出反应。教程使用了FCM处理这些推送
- FCM
  - 可以选择不同类型的message
    - Notification Message：在后台收到信息时会自动发出通知，并存为intent
    - Data Message：收到消息，但不做什么。不能从Firebase Console中发送



#### 代码部分

##### #1 连接FCM

- 在Firebase网站中
  - 创建新的项目、配置
  - 选择：将Firebase连接到数据库
  - 下载配置文件并保存到app目录下
- 在app中
  - 添加gradle依赖项
  - 添加SDK

##### #2 推送发送到特定设备

- 创建一个服务类继承自FirebaseInstanceIdService
  - 重写方法
- 在MainActivity中
  - 调用FirebaseInstanceId.getInstance().getToken()，以获取token
- 在Manifest中
  - 注册上述服务
  - 该服务中设置intent-filter，action为com.google.firebase.INSTANCE_ID_EVENT
- 在Firebase Console中发送，用于验证是否成功配置

##### #3 创建firebase消息服务

> 用于接收Data Message

- 创建一个服务类，继承自FirebaseMessagingService
  - 重写方法
- 在Manifest中
  - 注册上述服务
  - 该服务中设置intent-filter，action为com.google.firebase.MESSAGE_EVENT

##### #4 通过订阅的方式推送

> 让推送发送到多台设备，可以使用订阅作区分

- 在设置fragment中
  - 继承自PreferenceFragmentCompat，实现了SharedPreferences.OnSharedPreferenceChangeListener
  - 在实现的方法`onSharedPreferenceChanged()`中作出设置项被修改后做的事。这里调用了Firebase的`FirebaseMessaging.getInstance().(un)subscribeToTopic(key)`作为告知订阅与否的响应
  - 在实现的方法`onCreate()`和`onDestroy()`中注册和取消注册监听器



