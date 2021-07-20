## 日期

##### 切记不要用字符串存储日期

- 字符串占用的空间更大
- 字符串存储的日期比较效率比较低（逐个字符进行比对），无法用日期相关的 API 进行计算和比较。



##### DateTime 类型与Timestamp类型相比

- 没有时区信息



##### 关于时区的命令

- 查看当前会话时区
  SELECT @@session.time_zone;

- 设置当前会话时区
  SET time_zone = 'Europe/Helsinki';
  SET time_zone = " 00:00";

- 数据库全局时区设置
  SELECT @@global.time_zone;

- 设置全局时区

  SET GLOBAL time_zone = ' 8:00';
  SET GLOBAL time_zone = 'Europe/Helsinki';