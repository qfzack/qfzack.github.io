---
title: "学习笔记：MySQL数据库"

date: 2020-03-17T05:00:00Z
# image: "/images/placeholder.png"
categories: ["SQL"]
author: "Qingfeng Zhang"
---

# 1.MySQL的安装

这里是安装在ubuntu 18.04.3下的，首先更新apt，然后安装MySQL：

```shell
apt-get update  
apt-get install mysql-server  
```
  
安装完成之后，如果是root用户可以不输入密码，直接运行`mysql`进入MySQL，查看编码方式

```shell
show variables like 'character_set%';  
```
  
一般默认是`latin1`，将其修改为`utf8`：  
1.编辑`mysqld.cnf`:

```shell
vim /etc/mysql/mysql.conf.d/mysqld.cnf  
```
  
在`[mysqld]`下添加`set character_set_server=utf8;`;

2.编辑`mysql.cnf`：

```shell
vim /etc/mysql/conf.d/mysql.cnf  
```

添加`default-character-set=utf8`;

3.重启mysql服务：

```shell
service mysql restart  
```
  
# 2.MySQL常用命令

```sql
#列出数据库列表  
SHOW DATABASES;  
#选择数据库  
USE database_name;  
#显示当前数据库的所有列表  
SHOW TABLES;  
#显示数据表的属性  
SHOW COLUMNS FROM table_name;  
#显示数据表的详细索引信息，包括主键  
SHOW INDEX FROM table_name;  
#输出数据库管理系统的性能及统计信息(表名以"Data"开头的表)  
SHOW TABLE STATUS from database_name LIKE "Data%"\G;  
```
  
# 3.MySQL数据类型

**数值类型：**

类型 | 字节数 | 范围(有符号/无符号)  
---|---|---  
TINYINT | 1 | - 2^7 ~  2^7 -1 / 0 ~  2^8 -1  
SMALLINT | 2 | - 2^{15} ~  2^{15} -1 / 0 ~  2^{16} -1  
MEDIUMINT | 3 | - 2^{13} ~  2^{23} -1 / 0 ~  2^{24} -1  
INT | 4 | - 2^{31} ~  2^{31} -1 / 0 ~  2^{32} -1  
BIGINT | 8 | - 2^{63} ~  2^{63} -1 / 0 ~  2^{64} -1  
FLOAT[(M,D)] | 4 | (-3.402823466E+38,-1.175494351E-38),0,(1.175494351E-38,3.402823466351E+38)  
DOUBLE[(M,D)] | 8 | (-1.7976931348623157E+308,-2.2250738585072014E-308),0,(2.2250738585072014E-308,1.797693134 8623157E+308)  
DECIMAL[(M,D)] | 依赖M和D | 依赖M和D  
  
M表示数字的总位数，D表示小数点后的位数;  
无符号FLOAT的范围:0,(1.175494351E-38,3.402823466E+38);  
无符号DOUBLE的范围:0,(2.2250738585072014E-308,1.7976931348623157E+308)

> float和double是以二进制的形式存储的,存储可能会被截断导致精度丢失,所以保存的是一个近似值;  
>  decimal是唯一指定能精确存储的类型,比double范围更大,会将9位数字包装为4个字节,也就是用更大的范围来存储;

**日期/时间类型：**

类型 | 字节数 | 范围(有符号)  
---|---|---  
YEAR | 1 | 1901 ~ 2155  
TIME | 3 | -838:59:59 ~ 838:59:59  
DATE | 3 | 1000-01-01 ~ 9999-12-3  
DATETIME | 8 | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59  
TIMESTAMP | 4 | 1980-01-01 00:00:01 UTC ~ 2040-01-19 03:14:07 UTC  
  
**字符串类型：**

类型 | 存储需求  
---|---  
CHAR(M) | M字节固定长度,1<=M<255  
VARCHAR(M) | L+1变长,L< = M,1<=M<=65535  
TINYTEXT | L+1,L< 2^8  
TEXT | L+2,L< 2^{16}  
MEDIUMTEXT | L+3,L< 2^{24}  
LONGTEXT | L+4,L< 2^{32}  
ENUM | 枚举类型1或2个字节,取决于枚举值的数目（最多65535个）  
SET | 1,2,3,4或8个字节,取决于成员的数目（最多64个）  
  
> CHAR(M)不足M补空格,读取时会删去末尾的空格,VARCHAR(M)是实际长度加一个字节存放长度值;

**二进制类型：**  
包括BIT,BINARY,VARBINARY,TINYBLOB,BLOB,MEDIUMBLOB,LONGBLOB;

# 4.MySQL基本语法

### 1.数据库的操作

```sql    
#数据库的创建  
CREATE DATABASE [IF NOT EXISTS] <数据库名称> [[DEFAULT] CHARACTER SET <字符集名称>] [[DEFAULT] COLLATE <校对规则>]  
#数据库的修改  
ALTER DATABSE [数据库名称] {[DEFAULT] CHARACTER <字符集名称> [DEFAULT] COLLATE <校对规则>}  
#数据库的删除  
DROP DATABASE [IF EXISTS] <数据库名称>  
```
  
示例：`CREATE DATABASE IF NOT EXISTS db1 DEFAULT CHARACTER SET utf8 DEFAULT
COLLATE utf8_chinese_ci;`

### 2.数据表的操作

**2.1 数据表的创建、修改与删除**

```sql
#数据表的创建  
CREATE TABLE <表名> ([<列名1> <类型1>,...,<列名n> <类型n>]) [表选项] [分区选项]  
    
#数据表的修改：  
#添加字段  
ALTER <表名> ADD <字段名> <数据类型> [约束条件] [FIRST|AFTER <字段名>]  
#修改数据表字段类型  
ALTER TABLE <表名> MODIFY <字段名> <数据类型>  
#删除数据表的字段  
ALTER TABLE <表名> DROP <字段名>  
#修改数据表字段名  
ALTER TABLE <表名> CHANGE <旧字段名> <新字段名> <新数据类型>  
#修改数据表名  
ALTER TABLE <旧表名> RENAME [TO] <新表名>  
    
#数据表的删除  
DROP TABLE [IF EXISTS] <表名1> [,<表名2>,<表名3>...]  
```
  
示例：  
创建数据表：

```sql
CREATE TABLE tb1(  
    id INT(11),  
    name VARCHAR(25),  
    deptId INT(11),  
    salary FLOAT  
);  
```
  
修改数据表的字段类型：`ALTER TABLE tb1 MODIFY name VARCHAR(30);`  
修改数据表字段名：`ALTER TABLE tb1 CHANGE name newName VARCHAR(28);`

**2.2 数据表的查询**

```sql
#查询结果去重  
SELECT DISTINCT <字段名> FROM <表名>  
#设置别名  
<表名/字段名> [AS] <别名>  
#限制查询结果  
<LINIT> [<偏移量>,] <行数>  
#查询结果排序  
ORDER BY {<列名>|<表达式>|<位置>} [ASC|DESC]  
#查询的WHERE条件  
WHERE <查询条件> {<判定运算1>,<判定运算2>,...}  
```
  
判定运算的语法：

  * <表达式1> {=|<=|>|>=|<=>|<>|!=} <表达式2>
  * <表达式1> [NOT] LIKE <表达式2>
  * <表达式1> [NOT] [REGEXP|RLIKE] <表达式2>
  * <表达式1> [NOT] BETWEEN <表达式2> AND <表达式3>
  * <表达式1> IS [NOT] NULL

示例：  
查询并通过两个字段排序：`SELECT * FROM tb1 ORDER BY age,name;`  
通过多个字段值条件查询：`SELECT name,age FROM tb1 WHERE age<24 AND height>175;`  
通过字段名模糊查询：`SELECT name FROM tb1 WHERE name LIKE 'Z%';`

**2.3 数据表的连接查询**

```sql
SELECT <列名1,列名2...> FROM <表名1> INNER JOIN <表名2> [ON子句]  
```
  
  * 内连接查询：INNER JOIN ON
  * 左连接查询：LEFT JOIN ON/ LEFT OUTER JOIN ON
  * 右连接查询：RIGHT JOIN ON/ RIGHT OUTER JOIN ON

示例：`SELECT id,name,dept_name FROM tb1 INNER KOIN tb2 on
tb1.dept_id=tb2.dept_id;`

**2.4 数据表的子查询**

```sql
<表达式> [NOT] IN <子查询>  
<表达式> {=|<|>|>=|<=|<=>|<>|!=} {ALL|SOME|ANY} <子查询>  
```

示例：`SELECT name FROM tb1 WHERE id IN (SELECT id FROM tb2 WHERE grad='A');`

**2.5 数据表的分组查询**

```sql
GROUP BY {<列名>|<表达式>|<位置>} [ASC|DESC]  
#过滤分组  
HAVING <条件>  
```

示例：  
通过id分组并显示大于1的分组内容：`SELECT id,GROUP_CONCAT(name) AS names FROM tb1 GROUP BY id
HAVING COUNT(name)>1;`

**总结MySQL的查询语句结构如下：**

```sql    
SELECT DISTINCT column,…  
FROM mytable  
    JOIN another_table ON mytable.column = another_table.column  
    WHERE expression1  
    GROUP BY column  
    HAVING expression2  
    ORDER BY column ASC/DESC  
    LIMIT count OFFSET COUNT;  
```
  
### 3.记录的操作

```sql
#记录的增加  
INSERT INTO <表名> [<列名n> [,...列名n]] VALUES (值1) [...,(值n)]  
INSERT INTO <表名> SET <列名1> = <值1>, <列名2> = <值2>, ...  
    
#记录的修改  
UPDATE <表名> SET 字段1 = 值1 [,字段2 = 值2...] [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]  
    
#记录的删除  
DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]  
```
  
示例：添加记录：  
`INSERT INTO tb_course (name, info) VALUES ('DataBase','MySQL');`  
`INSERT INTO tb_course VALUES (2,'Java','JavaEE');`  
`INSERT INTO tb_course (id,name,info) SELECT course_id,course_name.course_info
FROM tb_course2;`  
示例：更新记录：  
`UPDATE tb_course SET name='DB',info='DataBaseCourse' WHERE id=1;`  
示例：删除记录：  
`DELETE FROM tb_course WHERE id=2;`

### 4.主键约束

```sql
#设置主键约束  
<字段名> <数据类型> PRIMARY KEY [默认值]  
#修改表添加主键约束：  
ALTER TABLE <表名> ADD PRIMARY KEY(<列名>)  
```
  
示例：`ALTER TABLE tb1 ADD PRIMARY KEY(id);`

### 5.外键约束

```sql
#设置外键约束  
[CONSTRAINT <外键名>] FOREIGN KEY 列名 [,列名2,...] REFERENCES <主表名> 列名1 [,列名2,...]  
#修改表添加外键约束：  
ALTER TABLE <表名> ADD CONSTRAINT <索引名> FOREIGN KEY(<列名>) REFERENCES <主表名>(<列名>)  
#删除表的外键约束：  
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>  
```
  
示例：将tb2上的depId作为外键关联到tb1的主键id：  
`CONSTRAINT cons FOREIGN KEY(depId) REFERENCES tb1(id);`

### 6.唯一约束和非空约束

```sql
#设置唯一约束  
<字段名> <数据类型> UNIQUE  
#修改表添加唯一约束  
ALTER TABLE <数据表名> ADD CONSTRAINT <唯一约束名> UNIQUE(<列名>)  
#删除字段的唯一约束  
ALTER TABLE <表名> DROP INDEX <唯一约束名>  
    
#设置非空约束  
<字段名> <数据类型> NOT NULL;  
```
  
示例：  
`ALTER TABLE tb_course ADD CONSTRAINT unique_name UNIQUE(name);`  
`ALTER TABLE tb_course DROP INDEX unique_name;`
