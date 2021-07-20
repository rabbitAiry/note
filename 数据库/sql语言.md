1#单表查询 SELECT语句
SELECT *** FROM *** WHERE （***） GROUP BY *** 从某表选择某列，其中符合某关系(σ操作)，按照某关系分组或排序
例   SELECT Sno,Sname FROM Student Where Smajor In('a','b')
SELECT * FROM ... *表示全部

函数-计算出生日期：今年-    YEAR（GETDATE（））-SAGE

SELECT DISTINCT *** FROM *** 去重复
SELECT *** FROM *** WHERE Sname LIKE '_晓%' 模糊查询名字中有“晓”的人
WHERE ISNULL 为空值时不用等于号

2#使用聚集函数 对数据的运算
COUNT 统计个数（如：男女、元组个数）
SELECT COUNT(***) as "newPage" FROM *** 建表统计，表名为newPage
SUM 求和
AVG 平均值
MAX 找最大
MIN 找最小

3#GROUP BY语句
GROUP BY排序
SELECT Smajor, COUNT(Sno)As '学生数' FROM Student GROUP By Smajor ORDER BY Smajor DESC 
以专业分组，先分组再按组count,以拼音降序排列
HAVING (***) 新的筛选限制，优先级低于where、group by、count（括号外），作用于被分开的各组中

查询选修两门以上（含两门）且各门课程均及格的学生的学号和其总成绩，查询结果按总成绩降序排列。
SELECT sno,sum(grade) AS 'Total_grade' FROM SC WHERE grade>=60 GROUP BY sno HAVING count(*)>=2 ORDER BY SUM(grade)

ALL ANY比大小中的范围是任意还是全部
4#连接查询:涉及两表以上的查询

最后一句Student.Sno = SC.Sno表示两表连接
SELECT Student.*,SC.* FROM Student,SC WHERE Student.Smajor='网络工程' AND Student.Sno = SC.Sno

自身连接：自己就是两个表     //如：查询先修课
自己的课程号与自己的先修课程号连接，所以C1.Cpno=C2.Cno
SELECT C1.Cname AS '课程名',C1.Credit AS '学分',C2.Cname AS '先修课程 课程号'
FROM Course AS C1,Course AS C2 WHERE C1.Cpno=C2.Cno



5#嵌套查询

IN语句:也可以改为 NOT IN
括号里面的皆为某个值
SELECT Sno, Sname, Smajor    //查询内容在同一表时最方便
FROM Student
WHERE Sno IN 
    ( SELECT Sno
       FROM SC
        WHERE Cno IN 
                (  SELECT Cno
                 FROM Course
                 WHERE Cname='数据结构' )
     )

6# EXISTS语句
只返回true或false
SELECT Sno, Sname, Smajor
FROM Student
WHERE  NOT  EXISTS 
(SELECT *
 FROM SC,Course
 WHERE SC.Cno = Course.Cno  AND
         Cname='操作系统'   AND  SC.Sno = Student. Sno )


7#数据更新
INSERT INTO *** VALUES()

复制表
INSERT 
INTO Stud(no, name, major)
SELECT Sno, Sname, Smajor
FROM Student

大量更新
UPDATE *** SET ***
UPDATE SC SET Grade = 0
UPDATE Course SET Ccredit = Ccredit +1 WHERE Cno='1'

删除数据
DELETE FROM *** WHERE ***
清空表格
DELETE FROM SC