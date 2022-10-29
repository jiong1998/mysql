 开始重新学习mysql数据库，这个文章将记录我学习mysql时的笔记以及遇到的问题

# 额外补充

## 1. mac/linux如何启动mysql
mac如何启动mysql：
1. 首先系统设置中启动mysql
2. 终端中输入mysql -u root -p

## 2.  MySQL8.0.26-Linux版安装


### 1. 准备一台Linux服务器

云服务器或者虚拟机都可以; 

Linux的版本为 CentOS7;



### 2. 下载Linux版MySQL安装包

https://downloads.mysql.com/archives/community/

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-mvQss4sT-1666859429547)(assets/image-20211031230239760.png)] 



### 3. 上传MySQL安装包

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-o6OYi8Vl-1666859429549)(assets/image-20211031231930205.png)] 



### 4. 创建目录,并解压

```
mkdir mysql

tar -xvf mysql-8.0.26-1.el7.x86_64.rpm-bundle.tar -C mysql
```



### 5. 安装mysql的安装包

```
cd mysql

rpm -ivh mysql-community-common-8.0.26-1.el7.x86_64.rpm 

rpm -ivh mysql-community-client-plugins-8.0.26-1.el7.x86_64.rpm 

rpm -ivh mysql-community-libs-8.0.26-1.el7.x86_64.rpm 

rpm -ivh mysql-community-libs-compat-8.0.26-1.el7.x86_64.rpm

yum install openssl-devel

rpm -ivh  mysql-community-devel-8.0.26-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-8.0.26-1.el7.x86_64.rpm

rpm -ivh  mysql-community-server-8.0.26-1.el7.x86_64.rpm

```



### 6. 启动MySQL服务

```
systemctl start mysqld
```

```
systemctl restart mysqld
```

```
systemctl stop mysqld
```



### 7. 查询自动生成的root用户密码

```
grep 'temporary password' /var/log/mysqld.log
```

命令行执行指令 :

```
mysql -u root -p
```

然后输入上述查询到的自动生成的密码, 完成登录 .



### 8. 修改root用户密码

登录到MySQL之后，需要将自动生成的不便记忆的密码修改了，修改成自己熟悉的便于记忆的密码。

```
ALTER  USER  'root'@'localhost'  IDENTIFIED BY '1234';
```

执行上述的SQL会报错，原因是因为设置的密码太简单，密码复杂度不够。我们可以设置密码的复杂度为简单类型，密码长度为4。

```
set global validate_password.policy = 0;
set global validate_password.length = 4;
```

降低密码的校验规则之后，再次执行上述修改密码的指令。



### 9. 创建用户

默认的root用户只能当前节点localhost访问，是无法远程访问的，我们还需要创建一个root账户，用户远程访问

```
create user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
```
注意：当你创建了一个可以远程连接的root用户以后，和本地root用户是同一个，只是登陆密码是可以不一样的。


### 10. 并给root用户分配权限

```
grant all on *.* to 'root'@'%';
```



### 11. 重新连接MySQL

```
mysql -u root -p
```

然后输入密码



### 12. 通过DataGrip远程连接MySQL






# 第一章：sql语句

sql注释：
- 单行注释：-- 注释内容 或 # 注释内容(MySQL特有)
- 多行注释： /* 注释内容 */

SQL又分为4类
- DDL ：定义语言，用来定义数据库对象(数据库，表，字段)
- DML：操作语言，用来对数据库表中的数据进行增删改
- DQL：查询语言，用来查询数据库中表的记录
- DCL：控制语言，管理用户、控制访问权限
## 1. DDL：定义数据库对象(数据库，表，字段)
### 1.1数据类型
MySQL中的数据类型有很多，主要分为三类：
- **数值类型（整型，浮点型）**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2eba5bd65b3c4f80a60d6ecbcbea77b2.png)

- **字符串类型**
	- char(10)，定长字符串：性能好，但是size固定
	- vachar(10)，变长字符串：性能差，但是size随着元素增加而增加
- **日期时间类型**
![在这里插入图片描述](https://img-blog.csdnimg.cn/19693e17bbc148e6ab83ccbe164b3f76.png)

### 1.2 数据库操作
- 查询
	- 查询所有数据库 
		- show databases;
	- 查询当前数据库
		- select databases();
- 创建
	- create database  数据库名;
- 删除
	- drop database [IF EXISTS]数据库名;
- 使用
	- use 数据库名;

注意：实际代码不需要加中括号[]，[]只是表示 可选
### 1.3 数据表操作 
#### 1.3.1 表查询：
- 查询当前库的所有表： show tables;
-  查询表结构：desc 表名;
- 查询指定表的建表语句：show create table 表名


#### 1.3.2 表创建：
```sql
CREATE TABLE 表名(
	字段1 字段1类型[COMMENT 字段1注释]，
	字段2 字段2类型[COMMENT 字段2注释]，
	字段3 字段3类型[COMMENT 宇段了注释]，
	...
	字段n 字段n类型[COMMENT 字段n注释]
)[COMMENT 表注释]；
注意 ：[ ... ] 为可选参数， 最后一个字段后面没有逗号
```
#### 1.3.3 表修改字段:
**添加字段：**
	alter table 表名 add 字段名 类型 [comment 注释]  [约束];

**修改**：
- 单纯修改数据类型：
	- alter table 表名 modify 字段名 新数据类型;
- 修改数据类型和字段名
	- alter table 表名 change 旧名 新名 数据类型;

**表删除字段：**
- alter table 表名 drop 字段名

**表名修改:**
- alter table 表名 rename to 新表名
#### 1.3.4 表删除:



表删除：
- drop table 表名；

## 2 DML：对数据库表中的数据进行增删改
- 增加数据：insert into
	- 给指定字段添加数据：insert into 表名(字段1,字段2) values(值1,值2);
	- 给全部字段添加数据：insert into 表名 values (值1,值2,...);
	- 批量插入多条数据：
		- insert into 表名(字段1,字段2) values(值1,值2),(值1,值2),(值1,值2) ; 
		- insert into 表名 values (值1,值2,...),(值1,值2,...),(值1,值2,...);
- 修改数据：update
	-  update 表名 set 字段名1 = 值1 , 字段名2 = 值2 , ....[WHERE 条件] 注意：如果没有条件则默认修改整张表所有数据
- 删除数据：delete
	- delete from 表名 [where 条件]

## 3 DQL：查询数据库中表的记录
编写顺序
```
SELECT
    字段列表
FROM
    表名字段
WHERE
    条件列表
GROUP BY
    分组字段列表
HAVING
    分组后的条件列表
ORDER BY
    排序字段列表 			升序：asc， 降序desc
LIMIT
    分页参数
```

### 3.1 基本查询：select
select：查询哪些字段

1. 查询多个字段
	- select 字段1, 字段2 from 表名;
	- select * from 表名;
2. 设置别名
	- select 字段1 [as 别名1] , 字段2 [as 别名2 ] ... from 表名;
	select name '名字' from emp;//as可以省略
3. 去除重复记录
	- select distinct 字段列表 from 表名;

### 3.2 条件查询: where
语法：SELECT 字段列表 FROM 表名 WHERE 条件列表;
条件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f5f5bf381334da0b709033631324775.png)
例子：
```sql
-- 年龄等于30
select * from employee where age = 30;
-- 年龄小于30
select * from employee where age < 30;
-- 小于等于
select * from employee where age <= 30;
-- 没有身份证
select * from employee where idcard is null or idcard = '';
-- 有身份证
select * from employee where idcard;
select * from employee where idcard is not null;
-- 不等于
select * from employee where age != 30;
-- 年龄在20到30之间
select * from employee where age between 20 and 30;
select * from employee where age >= 20 and age <= 30;
-- 下面语句不报错，但查不到任何信息
select * from employee where age between 30 and 20;
-- 性别为女且年龄小于30
select * from employee where age < 30 and gender = '女';
-- 年龄等于25或30或35
select * from employee where age = 25 or age = 30 or age = 35;
select * from employee where age in (25, 30, 35);
-- 姓名为两个字
select * from employee where name like '__';
-- 身份证最后为X
select * from employee where idcard like '%X';
```

### 3.3 聚合查询（聚合函数）
常见聚合函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ea1b368b2664c7d91f38ebcc2279e54.png)
语法：
```
SELECT 聚合函数(字段列表) FROM 表名;
```
例：
```
SELECT count(id) from employee where workaddress = "广东省";
```
注意：null不参与聚合函数的运算

### 3.4 分组查询：group by
语法：
```
SELECT 字段列表 FROM 表名 [ WHERE 条件 ] GROUP BY 分组字段名 [ HAVING 分组后的过滤条件 ];
```
**分组之前的过滤用where，分组之后的过滤用having。**

where 和 having 的区别：
- 执行时机不同：where是分组之前进行过滤，不满足where条件不参与分组；having是分组后对结果进行过滤。
- 判断条件不同：where不能对聚合函数进行判断，而having可以。

例子：
```sql
-- 根据性别分组，统计男性和女性数量（只显示分组数量，不显示哪个是男哪个是女）
select count(*) from employee group by gender;
-- 根据性别分组，统计男性和女性数量
select gender, count(*) from employee group by gender;
-- 根据性别分组，统计男性和女性的平均年龄
select gender, avg(age) from employee group by gender;
-- 年龄小于45，并根据工作地址分组
select workaddress, count(*) from employee where age < 45 group by workaddress;
-- 年龄小于45，并根据工作地址分组，获取员工数量大于等于3的工作地址
select workaddress, count(*) address_count from employee where age < 45 group by workaddress having address_count >= 3;
```

### 3.5 排序查询: order by
语法：
```sql
SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1, 字段 2 排序方式2;
```

排序方式：

- ASC: 升序（默认）
- DESC: 降序

例子：
```sql
-- 根据年龄升序排序
SELECT * FROM employee ORDER BY age ASC;
SELECT * FROM employee ORDER BY age;
-- 两字段排序，根据年龄升序排序，入职时间降序排序
SELECT * FROM employee ORDER BY age ASC, entrydate DESC;
```

注意事项

如果是多字段排序，当第一个字段值相同时，才会根据第二个字段进行排序
### 3.6 分页查询 limit
语法：
```
SELECT 字段列表 FROM 表名 LIMIT 起始索引, 查询记录数;
```

例子：
```sql
-- 查询第一页数据，展示10条
SELECT * FROM employee LIMIT 0, 10;
-- 查询第二页
SELECT * FROM employee LIMIT 10, 10;
```

注意事项
- 起始索引从0开始，起始索引 = （查询页码 - 1） * 每页显示记录数
- 分页查询不同数据库函数不同，MySQL是LIMIT
- 如果查询的是第一页数据，起始索引可以省略，直接简写 LIMIT 10### 3.
## 4 DCL：管理用户、控制访问权限
### 4.1 管理用户
查询用户：
```sql
USE mysql;
SELECT * FROM user;
```
创建用户:
```sql
CREATE USER '用户名'@'主机名' identified by '密码';
```
修改用户密码：
```sql
alter user '用户名'@'主机名' identified WITH mysql_native_password BY '新密码';
```
删除用户：
```
DROP USER '用户名'@'主机名';
```
例子：
```sql
-- 创建用户test，只能在当前主机localhost访问
create user 'test'@'localhost' identified by '123456';
-- 创建用户test，能在任意主机访问
create user 'test'@'%' identified by '123456';
-- 修改密码
alter user 'test'@'localhost' identified with mysql_native_password by '1234';
-- 删除用户
drop user 'test'@'localhost';
```

注意事项
- 主机名可以使用 % 通配
### 4.2 权限控制
常用权限
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7aa6adba78142739ffceeac2881d09e.png)
查询权限：
```sql
show grants FOR '用户名'@'主机名';
```
授予权限：
```sql
grant 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';
```
撤销权限：
```sql
revoke 权限列表 ON 数据库名.表名 FROM '用户名'@'主机名';
```
注意事项
- 多个权限用逗号分隔
- 授权时，数据库名和表名可以用 * 进行通配，代表所有
# 第二章：函数
## 1 字符串函数
常用函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/149c61b312fd400d9731bc0e7145d871.png)
例子
```sql
-- 拼接
SELECT CONCAT('Hello', 'World');
-- 小写
SELECT LOWER('Hello');
-- 大写
SELECT UPPER('Hello');
-- 左填充
SELECT LPAD('01', 5, '-');
-- 右填充
SELECT RPAD('01', 5, '-');
-- 去除空格
SELECT TRIM(' Hello World ');
-- 切片（起始索引为1）
SELECT SUBSTRING('Hello World', 1, 5);
```
## 2 数值函数
常见函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/419b68131aca430081146158385f8707.png)
```sql
--生成六位随机验证码
select lpad(ceil(rand()*1000000),6,0);
```
## 3 日期函数
常用函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/91bc63a6f8124ea9b877cfcdfcaf9a88.png)
例子：
```sql
-- DATE_ADD
SELECT DATE_ADD(NOW(), INTERVAL 70 YEAR);
```
## 4 流程函数
常用函数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/61bde01ed4e64ece8356beb62e0cfb80.png)
例子：
```sql
select
    name,
    (case when age > 30 then '中年' else '青年' end)
from employee;

select
    name,
    (case workaddress when '北京市' then '一线城市' when '上海市' then '一线城市' else '二线城市' end) as '工作地址'
from employee;
```
# 第三章：约束
## 1. 约束概念
概念：约束是**作用于表中字段上的规则**，用于**限制存储**在表中的数据。

目的：保证数据库中数据的正确、有效性和完整性。

分类：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e145a11b4f9544679a7cdef44d00ce0e.png)

<font color='red'>注意： 约束是作用于表中**字段**上的，可以在创建表/修改表的时候添加约束。 </font>

## 2. 常用约束关键字
![在这里插入图片描述](https://img-blog.csdnimg.cn/8628c639561d4aa4b3c297b5fb988750.png)

例子：
```sql
create table user(
    id int primary key auto_increment,
    name varchar(10) not null unique,
    age int check(age > 0 and age < 120),
    status char(1) default '1',
    gender char(1)
);
```

## 3. 外键约束
概念 ：外键用来让**两张表**的数据之间建立连接，从而保证数据的一致性和完整性。

例如，如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/e33561a572c84b6698acaf4cfb895032.png)
emp表的最后一个字段关联了另一张表的主键，则最后一个字段称为外键。通过外键让两张表数据产生连接。

语法：
```sql
--创建时设置
CREATE TABLE 表名(
    字段名 字段类型,
    ...
    [constraint] [外键名称] foreign key(外键字段名) references 主表(主表列名)
);
--修改
alter table 表名 add constraint 外键名称 foreign key (外键字段名) references 主表(主表列名);

--删除 
ALTER TABLE 表名 DROP FOREIGN KEY 外键名;
```
例子：
```sql
alter table emp add constraint fk_emp_dept_id foreign key(dept_id) references dept(id);
```
可选用的删除/更新行为的参数：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e051c9a8b6e0439fb6ea0fa239fb058c.png)
```sql
alter table 表名 add constraint 外键名称 foreign key (外键字段名) references 主表(主表列名) on update 行为 on delete 行为;
```

# 第四章：多表查询
## 1. 多表关系
多表关系:
- 一对多（多对一）
- 多对多
- 一对一

**一对多：**
- 例子：部门与员工
- 关系：一个部门对应多个员工，一个员工对应一个部门
- 实现：在多的一方建立**外键**，指向一的一方的主键

**多对多：**
- 例子：学生与课程
- 关系：一个学生可以选多门课程，一门课程也可以供多个学生选修
- 实现：建立一张**中间表**，中间表至少包含两个外键，分别关联两方主键

**一对一：**
- 例子：用户与用户详情
- 关系：一对一关系，多用于**单表拆分**，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升操作效率
- 实现：**在任意一方加入外键**，关联另外一方的主键，并且设置**外键为唯一的（UNIQUE）**
- 一对一其实就是一张表，只是为了效率，拆分成两张表
## 2. 多表查询----处理笛卡尔积
直接合并查询，会产生**笛卡尔积**，即会展示所有组合结果：
select * from employee, dept;
如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ca52f94fcd54783be5a2eba198ae4ca.png)
因此，在多表查询时，需要**消除无效的笛卡尔积**

消除无效笛卡尔积：只需要让子表中的外键=父表中的主键即可
```sql
select * from employee, dept where employee.dept = dept.id;
```
### 2.1 多表查询的分类
多表查询的分类
- 连接查询
	- 内连接：查询A、B表**交集**部分数据
	- 外连接
		- 左外连接：左外连接：查询**左表**所有数据，以及两张表交集部分数据
		- 右外连接：右外连接：查询**右表**所有数据，以及两张表交集部分数据
	- 自连接：当前表与自身的连接查询（可内可外），自连接必须使用表别名
- 子查询：SQL语句中**嵌套**SELECT语句，称为嵌套查询，又称子查询。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7f9cae565fb045afb8535797ebd41416.png)
## 3. 连接查询
### 3.1 内连接
内连接查询的是两张表**交集**的部分

**隐式内连接：**
SELECT 字段列表 FROM 表1, 表2 WHERE 条件 ...;

**显式内连接：**
SELECT 字段列表 FROM 表1 inner join 表2 on 连接条件 ...; //inner可省略

显式性能比隐式高

```sql
-- 查询员工姓名，及关联的部门的名称
-- 隐式
select e.name, d.name from employee as e, dept as d where e.dept = d.id;
-- 显式
select e.name, d.name from employee as e inner join dept as d on e.dept = d.id where e.age < 30;
```
### 3.2 外连接
**左外连接：**
**查询左表所有数据，以及两张表交集部分数据**

SELECT 字段列表 FROM 表1 left outer join 表2 on 条件 ...; //outer可省略
相当于查询表1的所有数据，包含表1和表2交集部分数据

在实际开发中，右外也可以改成左外，所以只用了解左外即可。
### 3.3 自连接
子连接：
自身的表与自身的表连接查询

当前表与自身的连接查询，**自连接必须使用表别名**

自连接查询，可以是内连接查询，也可以是外连接查询

语法：
- 内连接:
select 字段列表 from 表A 别名A join 表A 别名B on 条件 ...;
- 外连接:
select 字段列表 from 表A 别名A left join 表A 别名B on 条件 ...;

例子：
```sql
-- 查询员工及其所属领导的名字
select a.name, b.name from employee a, employee b where a.manager = b.id;
-- 没有领导的也查询出来
select a.name, b.name from employee a left join employee b on a.manager = b.id;
```

### 3.4 联合查询
**把多次查询的结果合并**，形成一个新的查询集

语法：
SELECT 字段列表 FROM 表A ...
union [all]
SELECT 字段列表 FROM 表B ...

注意事项
- UNION ALL 会有重复结果，UNION 不会
- 多张表的查询结果的列数量、类型必须一样。
- 联合查询比使用or效率高，不会使索引失效

## 4. 子查询
子查询：
**SQL语句中嵌套SELECT语句**，那个被嵌套的select的查询称为嵌套查询，又称子查询。
SELECT * FROM t1 WHERE column1 = ( SELECT column1 FROM t2);
<font color='red'> 子查询外部的语句可以是 insert / update / delete / select /where/ from的任何一个 </font>

根据子查询结果可以分为：
- 标量子查询（子查询结果为单个值）
- 列子查询（子查询结果为一列）
- 行子查询（子查询结果为一行）
- 表子查询（子查询结果为多行多列）

根据子查询位置可分为：
- WHERE 之后
- FROM 之后
- SELECT 之后

### 4.1 标量子查询：子查询结果为单个值
**子查询**返回的结果是单个值（数字、字符串、日期等）。

其实标量子查询就是编程的思想，把这个查询的语句当作返回的值直接当结果输入到另一个查询的语句中使用。

常用操作符：- < > > >= < <=
```sql
-- 查询销售部所有员工
select id from dept where name = '销售部';
-- 根据销售部部门ID，查询员工信息
select * from employee where dept = 4;
-- 合并（子查询）
select * from employee where dept = (select id from dept where name = '销售部');
-- 查询xxx入职之后的员工信息
select * from employee where entrydate > (select entrydate from employee where name = 'xxx');
```
### 4.2 列子查询：子查询结果为一列
子查询返回的结果是一列（可以是多行）。

常用操作符：
![在这里插入图片描述](https://img-blog.csdnimg.cn/03ad0470818a4b52be27252ac1dc8470.png)

例子：
```sql
-- 查询销售部和市场部的所有员工信息
select * from employee where dept in (select id from dept where name = '销售部' or name = '市场部');
-- 查询比财务部所有人工资都高的员工信息
select * from employee where salary > all(select salary from employee where dept = (select id from dept where name = '财务部'));
-- 查询比研发部任意一人工资高的员工信息
select * from employee where salary > any (select salary from employee where dept = (select id from dept where name = '研发部'));
```
### 4.3 行子查询：子查询结果为一行
返回的结果是一行（可以是多列）。
常用操作符：=, <, >, IN, NOT IN

例子：
```sql
-- 查询与xxx的薪资及直属领导相同的员工信息
select * from employee where (salary, manager) = (12500, 1);
select * from employee where (salary, manager) = (select salary, manager from employee where name = 'xxx');
 ```

### 4.4 表子查询：子查询结果为多行多列
返回的结果是多行多列
常用操作符：IN

例子：
```sql
-- 查询与xxx1，xxx2的职位和薪资相同的员工
select * from employee where (job, salary) in (select job, salary from employee where name = 'xxx1' or name = 'xxx2');
-- 查询入职日期是2006-01-01之后的员工，及其部门信息
select e.*, d.* from (select * from employee where entrydate > '2006-01-01') as e left join dept as d on e.dept = d.id;
```
# 第五章：事务
事务是一组操作的集合，事务会把所有操作作为**一个整体**一起向系统提交或撤销操作请求，即**这些操作要么一起成功，要么一起失败。**

想象一个例子：
张三要转账给李四，在数据库来说有三个操作：
- 查询张三余额
- 张三账号-1000
- 李四账号+1000

上述三个操作需要看作一个事务，不然就错了。

## 1. 事务基本操作
利用事务的操作方式一：
```sql
-- 查看事务提交方式
SELECT @@AUTOCOMMIT;
-- 设置事务提交方式，1为自动提交，0为手动提交，该设置只对当前会话有效
SET @@AUTOCOMMIT = 0;
-- 提交事务
commit;
-- 回滚事务
rollback;
```

操作实例:
```sql
-- 设置手动提交
SELECT @@AUTOCOMMIT;
SET @@AUTOCOMMIT = 1;
-- 操作数据
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
-- 提交事务：将数据真正提交到数据库
commit;
```

如果设置成手动提交事务的话，执行sql语句后，数据不会更新到数据库本身，只有输入commit才会真正更新数据库

利用事务的操作方式二：
```sql
开启事务：
start transaction 或 begin transaction;
提交事务：
commit;
回滚事务：
rollback;
```
操作实例:
```sql
-- 开启事务
start transaction;
-- 操作数据
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
-- 提交事务
commit;
```

## 2. 事务的四大特性
四大特性ACID：
- 原子性(Atomicity)：事务是不可分割的**最小操作单元**，要么全部成功，要么全部失败
- 一致性(Consistency)：事务完成时，必须使所有数据都保持**一致**状态
- 隔离性(Isolation)：数据库系统提供的**隔离**机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性(Durability)：事务一旦提交或回滚，它对数据库中的数据的改变就是**永久**的

## 3. 并发事务
**多个**并发事务会产生一系列问题，最常见的是如下三个：
- 脏读：一个事务读到另一个事务还没提交的数据
- 不可重复读：一个事务先后读取同一条记录，但两次读取的数据不同
- 幻读：一个事务按照条件查询数据时，没有对应的数据行，但是再插入数据时，又发现这行数据已经存在, 但是查询的时候又存在

## 4. 事务的隔离级别
上述三个并发事务的问题可以通过事务的**隔离级别**来解决

隔离级别    | 脏读 | 不可重复读  |幻读
-------- | ----- |---- |------ |
Read uncommitted  | 存在| 存在 | 存在|
Read committed	  | 不存在 | 存在| 存在 | 存在
Repeatable Read(默认)  | 不存在 | 不存在 | 存在
Serializable  | 不存在 | 不存在 | 不存在

从上到下数据的安全性越来越高，但是性能越来越差。

查看事务隔离级别：
SELECT @@transaction_isolation;

设置事务隔离级别：
set [ session | global ] transaction isolation level {Read uncommitted | Read committed | Repeatable Read | Serializable };


SESSION 是会话级别，表示只针对当前会话有效，GLOBAL 表示对所有会话有效

# 第六章：存储引擎
## 1. 介绍
MySQL体系结构：
- 连接层
	- 最上层是一些**客户端和链接服务**，主要完成一些类似于连接处理、授权认证、及相关的安全方案。服务器也会为安全接入的每个客户
端验证它所具有的操作权限。
- 服务层
	- 第二层架构主要**完成大多数的核心服务功能**，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存
储引擎的功能也在这一层实现，如过程、函数等
- 引擎层
	- 存储引擎真正的**负责了MySQL中数据的存储和提取**，服务器通过APl和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。
- 存储层
	- 主要是将数据存**储在文件系统之上**，并完成与存储引擎的交互。

如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/30bae57e489947d5ae67d26bb6d417f6.png)
**存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式**。存储引擎是**基于表**而不是基于库的，所以存储引擎也可以被称为**表引擎**。

注意：索引是在存储引擎层实现的，所以不同的存储引擎的索引结构是不同的。


建表时如何指定存储引擎：
```sql
-- 建表时指定存储引擎
CREATE TABLE 表名(
    ...
) ENGINE=INNODB;
```

## 2. innoDB
innoDB 是一种兼顾**高可靠性**和**高性能**的通用存储引擎，在 MySQL 5.5 之后，InnoDB 是默认的 MySQL 引擎。

特点：
- DML 操作遵循 ACID 模型，支持**事务**
- **行级锁**，提高并发访问性能
- 支持**外键**约束，保证数据的完整性和正确性

文件形式：
- xxx.ibd: xxx代表表名，InnoDB 引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引。

存储引擎特点：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fcb880637f8246a8888aaffe22465ee0.png)
在innodb中，page页是磁盘操作的最小单元。

## 3. Myisam
 mysql早期的默认存储引擎。
 

特点：
- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

文件：
- xxx.sdi: 存储表结构信息
- xxx.MYD: 存储数据
- xxx.MYI: 存储索引
## 4. Memory
Memory 引擎的表数据是**存储在内存**中的，受硬件问题、断电问题的影响，只能将这些表作为临时表或缓存使用。

特点：
- 存放在内存中，速度快
- hash索引（默认）

文件：
- xxx.sdi: 存储表结构信息
## 5. 三种存储引擎的特点总结
![在这里插入图片描述](https://img-blog.csdnimg.cn/cef51375195448ceb0f3fe5657571cc3.png)
## 6.存储引擎的选择
在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

- InnoDB: 如果应用对**事务的完整性有比较高**的要求，在**并发条件**下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，则 InnoDB 是比较合适的选择
- MyISAM: **如果应用是以读操作和插入操作为主**，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不高，那这个存储引擎是非常合适的。（日志、电商评论/足迹）
- Memory: 将所有数据保存在内存中，访问速度快，**通常用于临时表及缓存**。Memory 的缺陷是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。（现在一般用redis代替memory）

# 第七章：索引
 ## 1. 索引概述：
索引是帮助 **MySQL 高效获取数据**的**数据结构**（**有序**）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查询算法，这种数据结构就是索引。

优缺点：
- 优点：
	- **提高数据检索效率**，降低数据库的IO成本
	- 通过索引列对数据进行**排序**，降低数据排序的成本，降低CPU的消耗
- 缺点：
	- 索引列也是要**占用空间**的
	- 索引大大提高了查询效率，但**降低了更新的速度**，比如 INSERT、UPDATE、DELETE
## 2.索引结构
索引是在第三层存储引擎层实现的，所以存储引擎的不同，索引也不同。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ccc76144e08a47e3a43f6038732d39a5.png)
索引在不同存储引擎的支持情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1906d09e67df479abaee21d28b0685d8.png)
我们平常所说的索引，如果没有特别指明，都是指B+树结构组织的索引。
## 2.1 B树
![在这里插入图片描述](https://img-blog.csdnimg.cn/9fde22110eb24cedb1c536795b3a1140.png)
就是二叉查找树的多叉版。
注意：B树的分叉是在节点的左右两边的。B树的叶子节点是最下面的空节点

那么如何保证m阶的B树的查找效率呢？（如何维护B树，B树的条件）
- 每个节点最多有m个子树（m个孩子节点），即m-1个关键字
- 假设B树有m阶（阶：所有节点的孩子节点个数的最大值称为B树的阶，比如上图的阶就是5），那么除了根节点外，任何节点的**分叉**至少要有⌈m/2⌉，即**关键字**至少要有⌈m/2⌉-1
- 所有子树的高度相同，所以，所有失败节点都在同一层，即图里最下面的空节点
- 所有非叶节点的关键字是有序的

## 2.2 B+树
![在这里插入图片描述](https://img-blog.csdnimg.cn/6825fbf24d5d491fbd0631f7a8ca0224.png)

B+树是应对数据库所需而出现的一种B树的变形树。

B树和B+树的区别：
- B树的分叉是在节点的左右两边，而B+树的分叉是在节点本身，即有多少个节点就有多少个孩子

一棵m阶的B+树的条件：
- 每个分支节点最多有m棵子树（m个孩子节点）
- 如果一个根节点不是叶子节点，则必须要有两棵子树，其他每个分支节点至少有⌈m/2⌉棵子树。
- 节点的子树（孩子）个数=关键字个数
- 所有的叶子节点包含了所有的关键字以及指向相应记录的指针，叶子节点的关键字按大小顺序排序，相邻叶子节点按大小顺序相互连接。

B+树有两种查找方式：
- 从根节点向下查找，只有找到最下面的一个节点，才能找到该节点实际对应的记录。
- 直接从叶子节点从左往右进行顺序查找

为什么B+树更适合存储数据？
&emsp;&emsp;首先明确，在B/B+树中，一个节点存放在一个磁盘块中。在B树中，非叶节点存放了该关键字对应的存储地址，而在**B+树中，只有叶子节点才会从存放关键字对应的存储地址**，所以可以使一个磁盘块可以包含更多的关键字，使得B+树的阶更大，树更矮，读取磁盘的次数更少，查找更快。 
# 第六章：Mysql API库
在使用mysql的库的时候 需要使用链接库：
```
gcc hello.c -o hello -I/usr/include/mysql/ -L/usr/lib64/mysql/ -lmysqlclient 
```

总体使用的函数有：
```
a. 初始化：MYSQL *mysql_init(MYSQL *mysql)

b. 错误处理	：unsigned int mysql_errno(MYSQL *mysql) 

					char *mysql_error(MYSQL *mysql);
c. 建立连接：
MYSQL *mysql_real_connect(MYSQL *mysql, const char *host, const char *user, const char
	*passwd,const char *db, unsigned int port, const char *unix_socket, unsigned long client_flag);
	
d. 执行SQL语句：int mysql_query(MYSQL *mysql, const char *stmt_str)

e. 获取结果：	MYSQL_RES *mysql_store_result(MYSQL *mysql)
		MYSQL_ROW mysql_fetch_row(MYSQL_RES *result)
		
f. 释放内存：	void mysql_free_result(MYSQL_RES *result)
```
## 1. 初始化----mysql_init
MYSQL *mysql_init(MYSQL *mysql)
```cpp
MYSQL * mysql = mysql_init(NULL);
    if(mysql == NULL)
    {
        printf("mysql init error \n");
        return -1;
    }
printf("mysql init ok\n");
```

## 2. 连接与关闭数据库----mysql_real_connect
```
//参数依次为 mysql通道,
//ip,用户名(如用户名为主机，则填"localhost"),
//密码,数据库名,
//端口号,socket(设置为NULL的时候表示不使用socket)，客户端标识符（一般设置为0）
//mysql_real_connect函数若返回NULL表示则表示连接失败
MYSQL *mysql_real_connect(MYSQL *mysql, const char *host, const char *user, const char *passwd
, const char *db, unsigned int port, const char *unix_socket, un signed long client_flag);
```
```cpp
//初始化
MYSQL * mysql = mysql_init(NULL);
if(mysql == NULL)
{
    printf("mysql init error ! \n");
    return -1;
}
//连接数据库
MYSQL * conn = mysql_real_connect(mysql, "120.25.144.39", "root", "1234", "heima", 0, NULL, 0);
if(conn == NULL)
{
    printf("mysql connect error ! \n");
    return -1;
}
printf("connect mysql OK! \n");
```
## 3.  执行sql语句----mysql_query
当连接处于活动状态时，客户端或许会使用**mysql_query()**或**mysql_real_query()**向服务器发出SQL查询。两者的差别在于，mysql_query()预期的查询为指定的、由Null终结的字符串，而mysql_real_query()预期的是计数字符串。

int mysql_query(MYSQL *mysql, const char *query)

int mysql_real_query(MYSQL *mysql, const char *query, unsigned long length)

```cpp
//执行sql语句
int ret = mysql_query(conn, "insert into student values (11, '小路')");
if( ret == 0)
    printf("成功！\n");
```
## 4. 获取结果集----mysql_store_result
获取结果集就是对数据库进行SELECT、SHOW等获取操作。

一种方式是通过**mysql_store_result()**将整个结果集全部取回来。另一种方式则是调用**mysql_use_result()**初始化获取操作，但暂时不取回任何记录。视结果集的条目数选择获取结果集的函数。两种方法均通过**mysql_fetch_row()**来访问每一条记录。

**函数原型：**
- MYSQL_RES *mysql_store_result(MYSQL *mysql) 

**返回值：**
- 成功返回MYSQL_RES结果集指针
- 失败返回NULL。

整体获取的结果集，保存在 MYSQL_RES 结构体指针中，通过检查mysql_store_result()是否返回NULL，可检测函数执行是否成功：
```
MYSQL_RES *result = mysql_store_result(mysql);
if (result == NULL)
{
	ret = mysql_errno(mysql);
	printf("查询失败%s\n", mysql_error(mysql));
	return ret;	
}
```
该函数调用成功，则SQL查询的结果被保存在result中，但我们不清楚有多少条数据。

**所以应使用mysql_fetch_row的方式将结果集中的数据逐条取出**:

**函数原型：**
- MYSQL_ROW mysql_fetch_row(MYSQL_RES *result)

**返回值：**
- 成功返回下一行的MYSQL_ROW结构。
- 如果没有更多要检索的行或出现了错误，返回NULL。

```
MYSQL_ROW row = NULL;				//typedef char **MYSQL_ROW;	
while ((row = mysql_fetch_row(result))) 
{
	printf("%s\t%s\t%s\t%s\t%s\t%s\t%s\t%s\n", 
	row[0],row[1],row[2],row[3],row[4],row[5],row[6],row[7]);
}

```
**获取列数、获取行数**
查看帮助手册可以看到，有两个函数具备获取列数的功能：
- unsigned int mysql_field_count(MYSQL *mysql) 			从mysql句柄中获取有多少列。
- unsigned int mysql_num_fields(MYSQL_RES *result) 		从返回的结果集中获取有多少列。

```cpp
//获取列的个数
int rows = mysql_num_rows(result);
//获取行的个数
int fields = mysql_num_fields(result);
```
示例：
```cpp
//获取结果集
MYSQL_RES *results = mysql_store_result(conn);
if(results==NULL)
{
	printf("mysql_store_result error, [%s]\n",mysql_error(mysql));
	return -1;
}
printf("mysql_store_result ok\n");

//获取列数
unsigned int num = mysql_num_rows(result);

//获取结果集中每一行记录
MYSQL_ROW row;
while((row=mysql_fetch_row(results)))
{
	for(i=0; i<num; i++)
	{
		printf("%s\t", row[i]);	
	}
	printf("\n");
}
```

**获取表头信息：**
获取表头的API函数有两个：
- MYSQL_FIELD *mysql_fetch_fields(MYSQL_RES *result) 	全部获取
- MYSQL_FIELD *mysql_fetch_field(MYSQL_RES *result) 	获取单个

```cpp
MYSQL_FIELD *fields = NULL;
fields = mysql_fetch_fields(result);	//得到表头的结构体数组
for (i = 0; i < num; i++) //假设已通过 mysql_num_rows 获取了总列数	num
{		
	 
	printf("%s\t", fields[i].name);	//每一列的列名保存在name成员中 
}
printf("\n");

```

**释放结果集**：
结果集处理完成，应调用对应的函数释放所占用的内存。	
	
**函数原型：**
- void mysql_free_result(MYSQL_RES *result); 

**返回值：**
- 成功释放参数传递的结果集。没有失败情况。
```cpp
mysql_free_result(result);
```
