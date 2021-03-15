---
title: MySQL
date: 2020-10-01 19:29:25
tags:
- MySQL
top: 1
---



# MySQL

## Mysql的四种隔离级别

SQL标准定义了4类隔离级别，包括了一些具体规则，用来限定事务内外的哪些改变是可见的，哪些是不可见的。低级别的隔离级一般支持更高的并发处理，并拥有更低的系统开销。

### Read Uncommitted（读取未提交内容)

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。

### Read Committed（读取提交内容）

这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。

### Repeatable Read（可重读）

这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。

### Serializable（串行化）

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。



1. **脏读** ：脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
    e.g.
       1.Mary的原工资为1000, 财务人员将Mary的工资改为了8000(但未提交事务)
       2.Mary读取自己的工资 ,发现自己的工资变为了8000，欢天喜地！
       3.而财务发现操作有误，回滚了事务,Mary的工资又变为了1000

   像这样,Mary记取的工资数8000是一个脏数据。

2. **不可重复读** ：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
     e.g.
     1.在事务1中，Mary 读取了自己的工资为1000,操作并没有完成
     2.在事务2中，这时财务人员修改了Mary的工资为2000,并提交了事务.
     3.在事务1中，Mary 再次读取自己的工资时，工资变为了2000

 **解决办法：如果只有在修改事务完全提交之后才可以读取数据，则可以避免该问题。**

3. **幻读** : 是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。
     e.g.
     目前工资为1000的员工有10人。
     1.事务1,读取所有工资为1000的员工。
     2.这时事务2向employee表插入了一条员工记录，工资也为1000
     3.事务1再次读取所有工资为1000的员工 共读取到了11条记录，

    解决办法：如果在操作事务完成数据处理之前，任何其他事务都不可以添加新数据，则可避免该问题

**不可重复读的重点是修改** : 同样的条件, 你读取过的数据,再次读取出来发现**值**不一样了
**幻读的重点在于新增或者删除**: 同样的条件, 第 1 次和第 2 次读出来的**记录数**不一样

<!--more-->

## 联表查询

**内连接**：

说明：组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。

![](mysql\内连接.png)

**左连接（左外连接）**:

left join 是left outer join的简写，它的全称是左外连接，是外连接中的一种。

左(外)连接，左表(a_table)的记录将会全部表示出来，而右表(b_table)只会显示符合搜索条件的记录。右表记录不足的地方均为NULL。

![img](mysql\左外连接.png)



## MySQL的存储引擎--MyISAM和InnoDB的区别

1. InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；

2. InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败；

3. InnoDB 是聚集索引，MyISAM 是非聚集索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

   ![Innodb索引方式](F:\学习笔记\mysql\Innodb索引方式.png)

   ![MyISAM索引方式](F:\学习笔记\mysql\MyISAM索引方式.png)

   

4. InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快；

5. InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一； 

**常规使用操作**

* MYISAM 节约空间，速度较快
* INNODB 安全性高，事务的处理，多表多用户操作

> 在物理空间存储的位置：本质是文件存储，一个文件夹对应一个数据库

**MySQL引擎在物理文件上的区别**

* InnoDB在数据库表中只有一个*.frm文件，以及上级目录下的ibdata1文件
* MYSIAM
  * *.frm - 表结构的定义文件
  * *.MYD- 数据文件(data)
  * *.MYI  - 索引文件(index)

## Ubuntu下的MySQL布局详情

```sql
/usr/bin                客户端程序和mysql_install_db
/var/lib/mysql          数据库和日志文件
/var/run/mysqld         服务器
/etc/mysql              配置文件my.cnf
/usr/share/mysql        字符集，基准程序和错误消息
/etc/init.d/mysql       启动mysql服务器
/etc/mysql/mysql.conf.d 真正的配置文件
```

## Mysql复习

编写顺序

```SQL
SELECT DISTINCT
	<select list>
FROM
	<left_table> <join_type>
JOIN
	<right_table> ON <join_condition>
WHERE
	<where_condition>
GROUP BY
	<group_by_list>
HAVING
	<having_condition>
ORDER BY
	<order_by_condition>
LIMIT
	<limit_params>
```

执行顺序

``` sql
FROM	<left_table>

ON 		<join_condition>

<join_type>		JOIN	<right_table>

WHERE		<where_condition>

GROUP BY 	<group_by_list>

HAVING		<having_condition>

SELECT DISTINCT		<select list>

ORDER BY	<order_by_condition>

LIMIT		<limit_params>
```

### 1.操作数据库

#### 1.1 创建数据库

`create database [if not exists] westos`

#### 1.2 删除数据库

`drop database [if exists] westos`

#### 1.3 使用数据库

`use school` //如果一个表名或者字段名是一个特殊字符，则需要加``

#### 1.4 查看数据库

`show databases`

### 2. 数据库的列类型

> 数值

* tinyint 			1个字节大小
* smallint           2个字节大小
* mediumint     3个字节
* **int                    4个字节大小**
* bigint               8个字节
* float                  浮点数，4个字节
* double             浮点数， 8个字节
* decimal           字符串形式的浮点数

> 字符串

* char  字符串固定大小的 0~255
* varchar 可变字符串 0~65535 常用的  String
* tinytext  微型文本 2^8-1
* text  文本串  2^16-1

> 时间日期

* date YYYY-MM-DD,日期
* time HH:mm:ss 时间格式
* datetime YYYY-MM-DD HH:mm:ss 最常用的时间格式
* timestamp  时间戳 , 1970.1.1 到现在的毫秒数
* year  年份表示

> null

* 没有值，未知
* 不要使用NULL进行运算，结果为NULL

###  3. 数据库的字段属性（重点）

==Unsigned==:

* 无符号的整数
* 声明了该列不能为负数

==zerofill==:

* 0填充的
* 不足的位数，使用0来填充，int(3), 5->005

==自增（auto_increment）==：

* 通常理解为自增，自动在上一条记录的基础上+1
* 通常用来设计唯一的主键~ index，必须是整数类型
* 可以自定设计主键自增的起始值和步长

==非空（not Null）==

### 4.创建表

```sql
 create table if not exists `student` (
	`id` int(4) not null auto_increment comment '学号',
	`name` varchar(30) not null default '匿名' comment '姓名',
	`pwd` varchar(20) not null default '123456' comment '密码',
	`sex` varchar(2) not null default '女' comment '性别',
	`birthday` datetime default NULL comment '出生日期',
	`address`  varchar(100) default null comment '家庭住址',
	`email` varchar(50) default null comment '邮箱',
	PRIMARY key (`id`)
)engine=innodb default charset=utf8;
```

``` SQL
CREATE TABLE [IF NOT EXISTS] `表名` (
	`字段名` 类型 [属性] [索引] [注释],
	`字段名` 类型 [属性] [索引] [注释],
    `字段名` 类型 [属性] [索引] [注释],
)[表类型][字符集设置][注释]
```

### 5.修改删除表

--- 修改表名

```sql
alter table student rename as student1;
```

--- 增加表的字段

```sql
alter table student1 add age int(11);
```

-- 修改表的字段

```sql
alter table student1 modify age varchar(11);  // 无法重命名
alter table student1 change age age1 int(11);
```

-- 删除表的字段

```sql
alter table student1 drop age1
```

-- 删除表

```sql
drop table if exists teacher1;
```

### 6.MySQL数据管理

#### 6.1 外键

```sql
create table if not exists `student` (
	`id` int(4) not null auto_increment comment '学号',
	`name` varchar(30) not null default '匿名' comment '姓名',
	`pwd` varchar(20) not null default '123456' comment '密码',
	`sex` varchar(2) not null default '女' comment '性别',
	`birthday` datetime default NULL comment '出生日期',
	`gradeid` int(10) not null comment '学生的年级',
	`address`  varchar(100) default null comment '家庭住址',
	`email` varchar(50) default null comment '邮箱',
	PRIMARY key (`id`),
	key `fk_gradeid` (`gradeid`),
	constraint `fk_gradeid` foreign key (`gradeid`) references `grade` (`gradeid`)
)engine=innodb default charset=utf8;
```

```sql
mysql> drop table if exists grade;
ERROR 1217 (23000): Cannot delete or update a parent row: a foreign key constraint fails
删除有外键关系的表的时候，需要先删除引用别人的表（从表），再删除被引用的表（主表）
```

```sql
alter table `student` 
add constraint `fk_gradeid` foreign key (`gradeid`) references `grade` (`gradeid`);
--- alter table 表 add constraint 约束名 foreign key (作为外键的列) references 哪个表(哪个字段)
```

==最佳实践==

* 数据库就是单纯的表，只用来存数据，只有行（数据）和列（字段）
* 外键通过逻辑去实现

#### 6.2 DML语言

* insert

  ```sql
  insert into 表名([字段名1, 字段名2, ...]) values ('值1', '值2',...)
  ```

* update

* delete

#### 6.3 添加

```sql
insert into grade(`gradename`) values ('大四');

insert into grade(`gradename`) values 
('大三'), 
('大二'), 
('大一');

insert into `student`(`name`) values ('张三');

insert into `student`(`name`, `pwd`, `sex`) values 
('张三', 'aaaaa', '男'),
('李四', 'bbbbb', '女');
```

语法：``insert into 表名([字段名1, 字段名2, ...]) values ('值1', '值2',...), ('值1', '值2',...);``

注意事项：

* 字段和字段之间使用英文逗号隔开
* 字段是可以省略的，但是必须一一对应
* 可以同时插入多条数据

#### 6.4 修改

```sql
update 表名 set colnum_name=value where [条件]
```

```sql
 update `student` set name='baobaobao123', birthday=current_time  where name='baobaobao123' and sex='女';
```

#### 6.5  删除

语法:``delete from 表名 where[条件]``

```sql
delete from `student` where id=1
```

> TRUNCATE

作用：清空一个数据库，表的索引和约束不会变

```sql
-- 清空 student表
TRUNCATE `student`
```

delete和truncate的不同点：

* TRUNCATE 重新设置自增列 计数器会归零
* TRUNCATE 不会影响事务

> delete删除的问题：重启数据库，现象
>
> * InnoDB 自增列会重新从1开始 （存在内存中）
> * MyISAM 继续从上一个自增量开始 （存在文件中）

### 7. 查询数据(DQL)

#### 7.1 指定查询字段

```sql
select * from student;
select `StudentNo`, `StudentName`  from student;

-- 字段起别名
select `StudentNo` as 学号, `StudentName` as 学生姓名 from student;

select CONCAT('姓名: ', StudentName) as 新名字 from student;
```

> 语法: select 字段 from 表



> 去重 distinct

```sql
select distinct `StudentNo` from result;
```

```sql
select VERSION()  -- 查询系统版本（函数）
select 100*3-1 as 计算结果 

```

#### 7.2 where条件子句

作用： 检索数据中`符合条件`的子句

```sql
select studentNo, `StudentResult` from result;

select studentNo, `StudentResult` from result where Studentresult >= 95 and StudentResult <= 100;

select studentNo, `StudentResult` from result where Studentresult between 80 and 95;


select studentNo, `StudentResult` from result where StudentNo != 1000;

select studentNo, `StudentResult` from result where not studentNo = 100; 
```

#### 7.3 模糊查询

```sql
--- %表示0到多个字符，_表示一个字符
select studentNo, `StudentName` from `student` where studentName like '张%';

select studentNo, `studentname` from `student` where studentName like '张_';

select studentNo, `StudentName` from `student` where address in ('上海浦东');

select studentNo, `studentname` from `student` where `borndate` is not null and address in ('上海浦东');
```

#### 7.4 联表查询

> JOIN

![img](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=4057843062,2771292381&fm=26&gp=0.jpg)



| 操作       |                                            |
| ---------- | ------------------------------------------ |
| inner join | 如果表中至少有一个匹配，就返回行           |
| left join  | 会从左表中返回所有的值，即使右表没有匹配   |
| right join | 会从右表中返回所有的值，即使左表中没有匹配 |

```sql
-- 查询了参加了考试的同学（学号， 姓名，科目编号，分数）
select s.studentno, studentName, subjectNo, studentresult
from student s
inner join result r
on s.studentnO = r.studentno;

--- 查询缺考的同学
select s.studentno, studentName, subjectNo, studentresult
from student s
left join result r
on s.studentno = r.studentNo
where studentresult is null;

-- join(连接的表) on(判断条件) 连接查询
-- where 等值查询

-- 思考题（查询了参加考试的同学信息：学号，学生姓名，科目名称，分数名称）
select s.studentno, studentName, subjectname, studentresult
from student s
right join result r
on r.studentno = s.studentno
inner join subject sub
on r.subjectno = sub.subjectno;
```



> 自连接

自己的表和自己的表连接，核心：一张表拆违两张一样的表即可

```sql
select a.`categoryName` as '父栏目', b.`categoryName` as '子栏目'
FROM `category` as a, `category` as b
where a.`categoryid` = b.`pid`;
```

> 练习

```sql
-- 查询学员所属的年级（学号，学生的姓名， 年级的名称）
select studentNo, studentName, GradeName
from student s 
inner join `grade` g
on s.`gradeid` = g.`gradeid`;

-- 查询科目所属的年级（科目名称，年级名称）
select subjectname, `gradename`
from subject sub
inner join grade g
on sub.gradeid = g.gradeid;

-- 查询了参加数据库结构-1考试的同学信息： 学号，学生姓名，科目名，分数
select s.`studentno`, `studentname`, `subjectname`, `studentresult`
from student s
inner join result r
on s.studentno = r.studentno
inner join subject sub
on r.subjectno = sub.subjectno
where subjectName='高等数学-1';
```

#### 7.5 分页和排序

```sql
-- 升序
select s.`studentno`, `studentname`, `subjectname`, `studentresult`
from student s
inner join result r
on s.studentno = r.studentno
inner join subject sub
on r.subjectno = sub.subjectno
where subjectName='高等数学-1'
order by studentresult asc;

-- 分页
select s.`studentno`, `studentname`, `subjectname`, `studentresult`
from student s
inner join result r
on s.studentno = r.studentno
inner join subject sub
on r.subjectno = sub.subjectno
where subjectName='高等数学-1'
order by studentresult asc
limit 5, 5

-- 第N页  limit 0,5  (n-1) * pageSize, pageSize
-- [pagesize：页面大小]
-- [(n-1)*pagesize：起始值]
-- [n: 当前页]
-- [数据总数/页面大小 = 总页数]
```

语法：`limit(起始页面，偏移量)`

#### 7.6 嵌套查询

```sql
-- 分数不小于80分的学号和姓名
select DISTINCT s.studentno, studentname
from student s
inner join result r
on r.studentno = s.studentno
where `studentresult` >= 80 and subjectno = (select subjectno
from subject where subjectname = '高等数学-1')
```

### 8. MySQL函数

=== 常用函数 ===

-- 数学运算

```sql
SELECT ABC(-8)    -- 绝对值
SELECT celling(9.4)  -- 向上取整
SELECT floor(9.4) -- 向下取整
SELECT RAND() -- 返回一个0~1之间的随机数
SELECT sign() -- 判断数的符号
```

-- 字符串函数

```sql
SELECT  CHAR_LENGTH('即使再小的帆也能远航') 
SELECT  CONCAT('我', '爱', '你们')
SELECT  insert('我爱编程helloworld', 1, 2, '超级热爱')
SELECT  LOWER('kuangshen')
SELECT  UPPER('kuangshen')
SELECT  INSERT('kuangshen', 'h')
SELECT  Replace('坚持就能成功', '坚持', '努力');
```



=== 聚合函数 === 

| 函数名称 | 描述   |
| -------- | ------ |
| COUNT()  | 计数   |
| SUM()    | 求和   |
| AVG()    | 平均值 |
| MAX()    | 最大值 |
| MIN()    | 最小值 |

```sql
-- 查询不同课程的平均分，最高分，最低分
select Subjectname, avg(studentresult), max(studentresult), min(studentresult)
from result r
inner join `subject` sub
on r.`subjectno` = sub.`subjectno`
group by r.subjectno
having avg(studentresult) >= 80;
```

```sql
select [DISDINCT] 查询的字段 
from 表
xxx join
on
where
group by 
having
order by ..
limit startindex, pagesize
```

### 9.事务

```sql
ACID -- 原子性（atomicity）、一致性（Consistency）、隔离性（isolation）、持续性（durability）
```

```sql
SET autocommit = 0 /* 关闭 */
SET atuocommit = 1 /* 开启 */

-- 手动处理事务
-- 事务开始
START TRANSACTION -- 标记一个事务的开始
INSERT 	xx
INSERT  xx
-- 提交：持久化
COMMIT
-- 回滚： 回到原来的样子
ROLLBACK
-- 事务结束
SET autocommit = 1 -- 开启自动提交

-- 了解
savepoint 保存点名称
ROLLBACK TO SAVEPOINT 保存点
```

### 10.索引

#### **篇外：B+树**

BTree又叫多路平衡搜索树，一颗m叉的BTree特性如下：

- 树中每个节点最多包含m个孩子。
- 除根节点与叶子节点外，每个节点至少有[ceil(m/2)]个孩子。
- 若根节点不是叶子节点，则至少有两个孩子。
- 所有的叶子节点都在同一层。
- 每个非叶子节点由n个key与n+1个指针组成，其中[ceil(m/2)-1] <= n <= m-1 

**B+Tree为BTree的变种，B+Tree与BTree的区别为：**

1). n叉B+Tree最多含有n个key，而BTree最多含有n-1个key。

2). B+Tree的叶子节点保存所有的key信息，依key大小顺序排列。

3). 所有的非叶子节点都可以看作是key的索引部分。

![00001](F:\学习笔记\mysql\00001.JPG)

由于B+Tree只有叶子节点保存key信息，查询任何key都要从root走到叶子。所以B+Tree的查询效率更加稳定。

```sql 
MySQL官方对索引的定义为：索引（index）是帮助MySQL高效获取数据的数据结构。
```

**索引的优势和劣势**

优势

1. 类似于书籍的目录索引，提高数据检索的效率，降低数据库的IO成本
2. 通过索引列队数据进行排序，降低数据排序的成本，降低CPU的消耗

劣势

1. 实际上索引也是一张表，该表中保存了主键和索引字段，并指向实体类的记录，所以索引也是要占用空间的
2. 虽然索引大大提高了查询效率，但同时也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE. 因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息.

#### 为什么MySQL选择B+树做索引

1、 **B+树的磁盘读写代价更低**：B+树的内部节点并没有指向关键字具体信息的指针，因此其内部节点相对B树更小，如果把所有同一内部节点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多，一次性读入内存的需要查找的关键字也就越多，相对IO读写次数就降低了。

2、**B+树的查询效率更加稳定**：由于非终结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

3、**B+树更便于遍历**：由于B+树的数据都存储在叶子结点中，分支结点均为索引，方便扫库，只需要扫一遍叶子结点即可，但是B树因为其分支结点同样存储着数据，我们要找到具体的数据，需要进行一次中序遍历按序来扫，所以B+树更加适合在区间查询的情况，所以通常B+树用于数据库索引。

4、**B+树更适合基于范围的查询**：B树在提高了IO性能的同时并没有解决元素遍历的我效率低下的问题，正是为了解决这个问题，B+树应用而生。B+树只需要去遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作或者说效率太低。

#### 10.1 索引分类

* 主键索引（PRIMARY KEY）
  * 唯一的标识，主键不可重复，只能有一个主键
* 唯一索引（UNIQUE KEY）
  * 避免重复的列出现，唯一索引可以重复，多个列都可以标识为 唯一索引
* 常规索引（KEY/INDEX）
  * 默认的，index/key关键字来设置

* 全文索引（FULLTEXT）
  * 在特定的数据库引擎下才有，MyISAM

```sql
show index from student;

-- 增加一个全文索引（索引名）列名
ALTER TABLE student add FULLTEXT INDEX `studentname` (`studentname`);

--
```

#### 10.2 索引结构

| 索引      | InnoDB引擎      | MyISAM | Memory |
| --------- | --------------- | ------ | ------ |
| BTREE     | 支持            | 支持   | 支持   |
| HASH      | 不支持          | 不支持 | 支持   |
| R-tree    | 不支持          | 支持   | 不支持 |
| Full-text | 5.6版本之后支持 | 支持   | 不支持 |

> 不明确说明的前提下聚集索引、复合索引、前缀索引、唯一索引默认都是使用B+Tree树索引，统称为索引

#### 10.3 索引语法

```sql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name
[using index_tyep]
on tb1_name(index_col_name,...)

index_col_name : column[(length)][ASC | DESC]

-- 删除索引
drop index idx_city_name on city;

-- ALTER添加索引
Alter table tb_name add [primary|unique|fulltext|index] key(column_list);
```

![image-20201112110936892](C:\Users\bao\AppData\Roaming\Typora\typora-user-images\image-20201112110936892.png)

#### 10.4 索引设计原则

* 对查询频次较高，且数据量比较大的表建立索引
* 索引字段的选择，最佳候选列应当从where子句的条件中提取，如果where子句中的组合比较多，那么应当挑选最常用、过滤效果最好的列的组合
* 使用唯一索引，区分度越高，使用索引的效率越高
* 索引并不是越多越好
* 使用短索引
* 利用最左前缀，N个列组合而成的组合索引，那么相当于是创建了N个索引，如果查询时where子句使用了组成该索引的前几个字段，那么这条查询SQL可以利用组合索引来提升查询效率。

#### 10.5 索引失效

1. **全值匹配 ，对索引中所有列都指定具体值**

```sql
explain select * from tb_seller where name='小米科技' and status='1' and address='北京市'\G;
```

![1556170997921](F:\hexo\docs\source\_posts\mysql\1556170997921.png)

2. **最左前缀法则**

   如果索引了多列，要遵守最左前缀法则。指的是**查询从索引的最左前列开始，并且不跳过索引中的列**。

3. **范围查询右边的列，不能使用索引** 
4. **不要在索引列上进行运算操作， 索引将失效**
5. **字符串不加单引号，造成索引失效**
6. **尽量使用覆盖索引(只查询有索引的列)，避免select ***
7. **用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到**

 ```sql
explain select * from tb_seller where name='黑马程序员' or createtime = '2088-01-01 12:00:00'\G;	
 ```

![1556174994440](F:\hexo\docs\source\_posts\mysql\1556174994440.png)

8. **以%开头的Like模糊查询**，索引失效

   如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

![1556175114369](F:\hexo\docs\source\_posts\mysql\1556175114369.png)

   解决方案 ： 

   通过覆盖索引来解决 

![1556247686483](F:\hexo\docs\source\_posts\mysql\1556247686483.png)

9. **如果MySQL评估使用索引比全表更慢，则不使用索引**
10. **in 走索引， not in 索引失效**
11. **单列索引和复合索引**

 尽量使用复合索引，而少使用单列索引 。

### 11.视图

#### 11.1视图的语法

```sql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW view_name [(column_list)]
AsS select_statement
[with [CASCADED | LOCAL] CHECK OPTION]
```

![image-20201113091417536](C:\Users\bao\AppData\Roaming\Typora\typora-user-images\image-20201113091417536.png)

```sql
选项：
	with [cascaded | local] check option 决定了是否允许更新数据使记录不再满足视图的条件。
	Local： 只要满足本是图的条件就可以更新
	CASCADED： 必须满足所有针对该视图的所有视图的条件才可以更新，默认值
```

### 12.存储过程

```sql
create PRODUCEDURE procedure_name
begin
	--- SQL语句
end ;
```

```sql 
delimiter & // 声明分隔符

create PRODUCEDURE procedure_name
begin
	--- SQL语句
end ;

```

```sql
--- 查看存储过程
show procedure statu\G;

--- 查看某个存储过程的定义
show create procedure test.pro_test1 \G;

--- 删除存储过程
drop procedure pro_test1;

--- 声明变量
declare num int default 5;
select  ... into var_num

--- 直接赋值使用set
set var_name = expr [, var_name = expr]...
```

### 13.SQL优化

#### 13.1show profile分析SQL

Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。

通过 have_profiling 参数，能够看到当前MySQL是否支持profile：

​	![1552488372405](F:\hexo\docs\source\_posts\mysql\1552488372405.png)

默认profiling是关闭的，可以通过set语句在Session级别开启profiling：

![1552488401999](F:\hexo\docs\source\_posts\mysql\1552488401999.png)

```sql
set profiling=1; //开启profiling 开关；
```

通过profile，我们能够更清楚地了解SQL执行的过程。

首先，我们可以执行一系列的操作，如下图所示：

```sql
show databases;

use db01;

show tables;

select * from tb_item where id < 5;

select count(*) from tb_item;
```

执行完上述命令之后，再执行show profiles 指令， 来查看SQL语句执行的耗时：

![1552489017940](F:\hexo\docs\source\_posts\mysql\1552489017940.png)

通过``show  profile for  query  query_id ``语句可以查看到该SQL执行过程中每个线程的状态和消耗的时间

![1552489053763](F:\hexo\docs\source\_posts\mysql\1552489053763.png)

> TIP ：
> 	Sending data 状态表示MySQL线程开始访问数据行并把结果返回给客户端，而不仅仅是返回给客户端。由于在Sending data状态下，MySQL线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。

在获取到最消耗时间的线程状态后，MySQL支持进一步选择all、cpu、block io 、context switch、page faults等明细类型类查看MySQL在使用什么资源上耗费了过高的时间。例如，选择查看CPU的耗费时间  ：

![1552489671119](F:\hexo\docs\source\_posts\mysql\1552489671119.png)

| 字段       | 含义                           |
| ---------- | ------------------------------ |
| Status     | sql 语句执行的状态             |
| Duration   | sql 执行过程中每一个步骤的耗时 |
| CPU_user   | 当前用户占有的cpu              |
| CPU_system | 系统占有的cpu                  |

#### 13.1 优化insert语句

* 如果需要同时对同一张表插入多行数据时，应该尽量使用多个值表的insert语句，这种方式将大大的缩减客户端与数据库服务器质检的连接、关闭等消耗。使得效率比分开执行的单个insert语句块；

  ```sql
  --- 不要使用
  insert into tb_test values (1, 'Tom');
  insert into tb_test values (2, 'Jack');
  insert into tb_test values (3, 'Tonny');
  --- 尽量使用
  insert into tb_test values (1, 'Tom'), (2, 'Jack'), (3, 'Tonny');
  ```

* 在事务中进行数据插入

  ```sql
  start transaction;
  insert into tb_test values (1, 'Tom');
  insert into tb_test values (2, 'Jack');
  insert into tb_test values (3, 'Tonny');
  commit;
  ```

* 数据有序的插入

  ```sql
  --- 尽量将数据按主键升序或者降序插入
  insert into tb_test values(1,'Tom');
  insert into tb_test values(2,'Cat');
  insert into tb_test values(3,'Jerry');
  insert into tb_test values(4,'Tim');
  insert into tb_test values(5,'Rose');
  ```


#### 13.2 优化Order by和Group by

> Order by：有两种排序方式

1.  第一种是通过对返回数据进行排序，也就是通常所说的filesort，所有不是通过索引直接返回排序结果的排序都叫FileSort
2. 第二种是通过有序索引顺序扫描返回有序的数据

**所以优化Order by的方式即：尽量减少额外的排序，通过索引直接返回有序的数据**，并且where条件和order by使用相同的索引，并且order by的顺序和索引

顺序相同，同时order by的字段都是升序，或者都是降序。否则肯定需要额外操作，这样就会出现FileSort。

**FileSort的优化**

1. 两次扫描算法 ：MySQL4.1 之前，使用该方式排序。首先根据条件取出排序字段和行指针信息，然后在排序区 sort buffer 中排序，如果sort buffer不够，则在临时表 temporary table 中存储排序结果。完成排序之后，再根据行指针回表读取记录，该操作可能会导致大量随机I/O操作。
2. 一次扫描算法：一次性取出满足条件的所有字段，然后在排序区 sort  buffer 中排序后直接输出结果集。排序时内存开销较大，但是排序效率比两次扫描算法要高。

MySQL 通过比较系统变量 **max_length_for_sort_data** 的大小和Query语句取出的字段总大小， 来判定是否那种排序算法，如果**max_length_for_sort_data** 更大，那么使用第二种优化之后的算法；否则使用第一种。

可以适当提高 **sort_buffer_size**  和 **max_length_for_sort_data**  系统变量，来增大排序区的大小，提高排序的效率。

> 优化Group by

由于GROUP BY 实际上也同样会进行排序操作，而且与ORDER BY 相比，GROUP BY 主要只是多了排序之后的分组操作。当然，如果在分组的时候还使用了其他的一些聚合函数，那么还需要一些聚合函数的计算。所以，在GROUP BY 的实现过程中，与 ORDER BY 一样也可以利用到索引。

**如果查询包含 group by 但是用户想要避免排序结果的消耗， 则可以执行order by null 禁止排序。如下 ：**

```sql
drop index idx_emp_age_salary on emp;

select age,count(*) from emp group by age;

select age,count(*) from emp group by age order by null;
```

#### 13.3 优化嵌套查询（用联表查询代替子查询）

Mysql4.1版本之后，开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另一个查询中。使用子查询可以一次性的完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也很容易。但是，有些情况下，子查询是可以被更高效的连接（JOIN）替代。

#### 13.4 优化OR条件

对于包含OR的查询子句，如果要利用索引，则OR之间的每个条件列都必须用到索引 ， 而且不能使用到复合索引； 如果没有索引，则应该考虑增加索引。

**建议使用 union 替换 or **

### 14.应用优化

**应用优化包括几种方法：1.使用连接池。2.减少对MySQL的访问（1.避免对数据进行重复检索2.增加Cache层）3.负载均衡**

### 15.MySQL锁

MyISAM：**表锁**

1. 对MyISAM表的读操作，不会阻塞其他线程对该表的读操作，但会阻塞其他线程对该表的写操作；
2. 对MyISAM表的写操作，会阻塞其他线程对该表的读写操作。

InnoDB:  **行锁**

InnoDB  实现了以下两种类型的行锁。

- 共享锁（S）：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- 排他锁（X）：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。

对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)；

对于普通SELECT语句，InnoDB不会加任何锁；

##### 无索引行锁升级为表锁

如果不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，实际效果跟表锁一样。

##### 间隙锁危害

当我们用范围条件，而不是使用相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据进行加锁； 对于键值在条件范围内但并不存在的记录，叫做 "间隙（GAP）" ， InnoDB也会对这个 "间隙" 加锁，这种锁机制就是所谓的 间隙锁（Next-Key锁） 。

优化建议：

- 尽可能让所有数据检索都能通过索引来完成，避免无索引行锁升级为表锁。
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少索引条件，及索引范围，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可使用低级别事务隔离（但是需要业务层面满足需求）



## MySQL探究

>  MySQL支持的存储引擎包括：1. InnoDB 2. MyISAM 3.NDB 4.memory等七种常用的引擎

### 1. 连接MySQL

#### 1.1 TCP/IP

TCP/IP套接字方式是MySQL在任何平台下都提供的连接方式，也是网络中使用得最多的一种方式。这种方式在TCP/IP连接上建立一个基于网络的连接请求，一般情况下客户端在一台服务器上，而MySQL实例在另一台服务器上，这两台机器通过一个TCP/IP网络连接。

```sql
-- 远程连接必须授权 
-- 修改/etc/mysql/mysql.conf.d/mysqld.cnf 中的bind_address = 127.0.0.1为bind_address = 0.0.0.0
```

#### 1.2 命名管道和共享内存

命名管道：在MySQL数据库中，需在配置文件中启用--enable-named-pipe选项

共享内存：配置文件中添加--shared-memory，共享内存的方式还需要在客户端使用-protocol=memory选项

#### 1.3 Unix域套接字

### 2.InnoDB存储引擎

> InnoDB存储引擎是第一个**完整支持ACID事务的MySQL引擎, 行设计锁,支持MVCC，提供类似Oracle风格的一致性非毒锁定读，支持外键，被设计用来最有效利用内存和CPU**

#### 2.1线程

> InnoDB存储引擎后台线程有7个——4个IO，1个主线程，一个锁监控线程，1个错误监控线程

#### 2.2 内存

InnoDB存储引擎内存由**缓冲池**、**重做日志缓冲池**、以及**额外的内存池**

缓冲池中缓存的数据页类型有：**索引页，数据页，undo页，插入缓冲，自适应哈希索引，InnoDB存储的锁信息，数据字典信息**

