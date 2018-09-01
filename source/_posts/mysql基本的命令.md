**navicat快捷方式使用：**

~~~python
1.ctrl+q           打开查询窗口
2.ctrl+/            注释sql语句
3.ctrl+shift +/  解除注释
4.ctrl+r           运行查询窗口的sql语句
5.ctrl+shift+r   只运行选中的sql语句
6.F6               打开一个mysql命令行窗口
7.ctrl+l            删除一行
8.ctrl+n           打开一个新的查询窗口
9.ctrl+w          关闭一个查询窗口
10.ctrl+d     在查询表数据界面打开一个该表结构的窗口（我自己用的时候是复制这一行）
~~~





1.创建表：

~~~python
 CREATE TABLE `book2`(
    ->  `id` int(11) NOT NULL AUTO_INCREMENT,
    ->   `name` varchar(255) NOT NULL,
    ->   `author` varchar(255) DEFAULT NULL,
    ->   PRIMARY KEY (`id`)
    -> )
~~~

2.查看表结构：desc 表名。

如： desc t_book;

3.查看表格创建信息：show create table  表名。

如：show create table t_book;

4.更改表名：alter table 旧表名 rename 新表名。

alter table book rename books;

5.修改字段：alter table 表名 change 旧属性名 新属性名 新数据类型

6.增加字段：alter table 表名 add  属性名1 数据类型

7.删除字段： alter table 表名 drop 属性名

8.删除表名：drop table 表名。

**单表查询**

~~~python
select * from t_student;#查看所有字段
SELECT id,stuName,age from t_student;#查看指定字段
select * from t_student where id = 1;#查看id为的所有字段信息
select * from t_student WHERE age BETWEEN 22 and 25;#参看age在22-25的信息
select * from t_student where age in(21,23,25);#查看在in中的信息
select * from t_student WHERE stuName like '张三';#查看名字为张三的信息
select * from t_student WHERE stuName like '%张三';#查看名字张三结尾，%表示任意匹配
select * from t_student WHERE stuName like '张三_';#查看名字张三开头且后面接任意一个字符的信息
select * from t_student WHERE sex is NULL;#查找值为空
select * from t_student WHERE sex is NOT NULL;#查找值不为空
select * from t_student WHERE gradeName = '一年级' AND age = 23;#and查询
select * from t_student WHERE gradeName = '一年级' or age = 23;#or查询
select DISTINCT gradename from t_student #去重复
select * from t_student ORDER BY age Asc;#排序（升序）默认也是为升序
select * from t_student ORDER BY age DESC;#排序（降序）也可以直接在age前面加‘-’

分组查询
 select gradename,GROUP_CONCAT(stuname) from t_student GROUP BY gradeName#根据学生姓名分组
select gradeName,COUNT(stuName) from t_student GROUP BY gradeName#查询每个班级的学生人数 
select gradename,GROUP_CONCAT(stuname) from t_student GROUP BY gradeName with ROLLUP#在最后一行增加一行作为统计所有结果的和（文本是文本叠加，数字是和）
select gradeName,COUNT(stuName) from t_student GROUP BY gradeName HAVING COUNT(stuName)>3;#查询班级学生人数大于3的班级。

分页查询
SELECT * FROM t_student LIMIT 1,5#1代表从第1行开始查看，5代表每页5行。

SELECT COUNT(*) as total from t_grade#取别名
select stuname,sum(score) from t_grade where stuName = '张三'#查看张三总分
select stuname,sum(score) from t_grade GROUP BY stuname#查看所有学生总分
avg（），max（），min()同sum（）
~~~

**连接查询**（外连接！）

~~~python
select * from t_book,t_booktype#笛卡尔乘积两个表的数量乘积
将两个或两个以上的表按照某个条件连接起来选取需要的数据；

内连接：
SELECT * from t_book,t_booktype where t_book.bookTypeId = t_booktype.id
SELECT tb.bookName,tb.author,tby.bookTypeName from t_book tb,t_booktype tby where tb.bookTypeId = tby.id#取别名同时查看相应字段的信息

外连接：（左连接，右连接，
SELECT * from t_book LEFT JOIN t_booktype on t_book.bookTypeId = t_bookType.id
#左连接，左边表的数据全部显示出来，右边表有没匹配的数用null代替。
SELECT * from t_book RIGHT JOIN t_booktype on t_book.bookTypeId = t_bookType.id#与左连接相反
SELECT * from t_book,t_booktype where t_book.bookTypeId = t_booktype.id and price>70#多条件查询
SELECT * from t_book,t_booktype where t_book.bookTypeId = t_booktype.id and price#
~~~

**子查询**

~~~python
SELECT * from t_book where bookTypeId in (SELECT id from t_booktype）
SELECT * from t_book where price >=(select price from t_pricelevel where priceLevel=1)#通过一个表查看另外一个表的信息
SELECT * from t_book where EXISTS(select * from t_booktype)#查询是否存在  
select * from t_book where price >= ANY(SELECT price from t_pricelevel)#any满足任意一个
select * from t_book where price >= all(SELECT price from t_pricelevel)#全部都需要满足                        
~~~

**合并查询**

~~~python
union#将查的东西合并在一起，相同的只留一个
select id from t_book UNION SELECT id FROM t_booktype
union all#将查找的两个东西全部合并在一起
select id from t_book UNION ALL SELECT id FROM t_booktype
~~~

**插入数据**

~~~python
#插入
insert into t_book VALUES (NULL,'性暗示的',20,'张三',1)#null为默认值
insert into t_book(id,bookname,price,author,booktypeid) VALUES(NULL,'sad',21,'asd',2),(NULL,'aaad',21,'bbsd',2)#插入多条数据
#更新
UPDATE t_book set bookname = 'python' where id = 1
#删除
DELETE from t_book where bookname = 'sad'
~~~

**提高数据查询速度（索引查询）**

~~~python
索引：是由数据库表中的一列或者多列组合而成，其作用是提高对表数据的查询速度类似图书目录
优点：提高查询速度
缺点：创建和维护索引的时间增加

索引分类：
普通索引：创建在任何数据类型中
唯一索引：使用unique参数设置，在创建唯一索引是，限制该索引是唯一的
全文索引：使用fulltext，该索引创建在char，varchar，text类型的字段上，提高查询大字符串类型的速度，只有myIsam引擎支持该索引，mysql默认引擎不支持
单列索引：在表中可以给单个字段创建索引，单列索引可以是普通索引，唯一索引，全文索引
多列索引：在表的多个字段上创建索引
空间索引：使用spatial参数可以设置空间索引，只能建立在空间数据类型上，提高获取空间书觉得效率
create table t_user1(id int,
                     username VARCHAR(20),
                     PASSWORD varchar(20),
                     INDEX(username));#普通字段索引
create table t_user1(id int,
                     username VARCHAR(20),
                     PASSWORD varchar(20),
                     unique INDEX(username));#唯一索引
create UNIQUE index index_username on t_user1(username)#给已存在表创建索引起索引别名
~~~

**视图**

~~~python
视图是虚拟的表，数据库只存放视图的定义，而并没有存放视图的数据，这些数据存放在原来的表中，使用视图查询数据是数据库系统会从原来的表取出对应的数据
作用：
使操作简化，增加数据的安全性，提高表的逻辑独立性
CREATE VIEW v1 as SELECT *from t_book;#取出t_book全部数据作为视图下操作
create view v2 as select bookName,price from t_book#只取出两个字段进行操作，保证其他数据的安全性
create view v3 as select bookname,booktypename from t_book ,t_booktype where t_book.bookTypeId = t_booktype.id#多表的视图操作
~~~

**触发器triggers**

~~~python
由事件的操作来触发（增删改）
SHOW TRIGGERs;#显示所有触发器
drop TRIGGER trig_book;#删除触发器
create TRIGGER trig_books after insert #触发器名
	on t_book for each ROW
	UPDATE t_booktype set booknum = booknum+1 where new.booktypeid= t_booktype.id;
#创建一个触发器
insert into t_book VALUES(null,'jjj',100,'aaa',1);#事件启动触发器

create trigger trig_bookk after DELETE#触发器多表操作
	on t_book for each ROW
	BEGIN
			update t_booktype set booknum = booknum-1 where old.booktypeid = t_booktype.id;
			INSERT into t_log VALUES(null,NOW(),'adadda');
			DELETE from t_test where old.booktypeid=t_test.id;
	END
DELETE FROM t_book where bookname='java dd';#事件启动触发器
~~~

**函数**

~~~python
abs（x）求绝对值
sqrt（x）求平方根
mod（x，y）求余

password(str)对用户密码加密，不可逆
md5（str）普通加密，不可逆
encode（str,pswd_str）加密函数，结果是二进制，必须使用blob类型的字段来保存
decide(crypt_str,pswd_str)解密函数
~~~

**创建存储过程和函数引入**

~~~python
减少相同sql语句的书写

游标
~~~

**数据备份和数据还原**

~~~python
备份：
确保数据库安全，定期备份
1.使用mysqldump备份
mysqldump -u username -p dbname table table2...> 路径
2.使用工具备份

还原：
1.mysql -u root -p 文件名 <路径
~~~





