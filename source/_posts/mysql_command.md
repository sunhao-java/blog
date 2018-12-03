title: MySQL常用命令
tags: [database, MySQL]
date: 2016-03-18
---

1. 显示当前数据库服务器中的数据库列表：

		mysql> SHOW DATABASES;
	注意：mysql库里面有MYSQL的系统信息，我们改密码和新增用户，实际上就是用这个库进行操作。

<!-- more -->
2. 显示数据库中的数据表：

		mysql> USE 库名；
		mysql> SHOW TABLES;
3. 显示数据表的结构：

		mysql> DESCRIBE 表名;
4. 建立数据库：

		mysql> CREATE DATABASE 库名;
5. 建立数据表：

		mysql> USE 库名;
		mysql> CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));
6. 删除数据库：

		mysql> DROP DATABASE 库名;
7. 删除数据表：

		mysql> DROP TABLE 表名；
8. 将表中记录清空：

		mysql> DELETE FROM 表名;
9. 显示表中的记录：

		mysql> SELECT * FROM 表名;
10. 往表中插入记录：

		mysql> INSERT INTO 表名 VALUES (”hyq”,”M”);
11. 更新表中数据：

		mysql-> UPDATE 表名 SET 字段名1=’a',字段名2=’b’ WHERE 字段名3=’c';
12. 用文本方式将数据装入数据表中：

		mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt” INTO TABLE 表名;
13. 导入.sql文件命令：

		mysql> USE 数据库名;
		mysql> SOURCE d:/mysql.sql;
14. 命令行修改root密码：

		mysql> UPDATE mysql.user SET password=PASSWORD(’新密码’) WHERE User=’root’;
		mysql> FLUSH PRIVILEGES;
15. 显示use的数据库名：

		mysql> SELECT DATABASE();
16. 显示当前的user：

		mysql> SELECT USER();
17. 一个建库和建表以及插入数据的实例

		drop database if exists school; //如果存在SCHOOL则删除
		create database school; //建立库SCHOOL
		use school; //打开库SCHOOL
		create table teacher //建立表TEACHER
		(
			id int(3) auto_increment not null primary key,
			name char(10) not null,
			address varchar(50) default ‘深圳’,
			year date
		); //建表结束
		//以下为插入字段
		insert into teacher values(”,’allen’,'大连一中’,'1976-10-10′);
		insert into teacher values(”,’jack’,'大连二中’,'1975-12-23′);
 如果你在mysql提示符键入上面的命令也可以，但不方便调试。

	1. 你可以将以上命令原样写入一个文本文件中，假设为school.sql，然后复制到c:\\下，然后键入以下命令：

			mysql -uroot -p密码 < c:\\school.sql
	如果成功，空出一行无任何显示；如有错误，会有提示。（以上命令已经调试，你只要将//的注释去掉即可使用）。
	2. 或者进入命令行后使用 `mysql> source c:\\school.sql`; 也可以将`school.sql`文件导入数据库中。
18. 将文本数据转到数据库中
	1. 文本数据应符合的格式：字段数据之间用tab键隔开，null值用空格来代替.例：

			3 rose 大连二中 1976-10-10
			4 mike 大连一中 1975-12-23
		假设你把这两组数据存为school.txt文件，放在c盘根目录下。
	2. 数据传入命令 `load data local infile “c:\\school.txt” into table 表名`;

		注意：你最好将文件复制到`\\mysql\\bin`目录下，并且要先用use命令打表所在的库。
19. 备份数据库
	1. 导出整个数据库

		导出文件默认是存在mysql\bin目录下

			mysqldump -u 用户名 -p 数据库名 > 导出的文件名
			mysqldump -u user_name -p123456 database_name > outfile_name.sql
	2. 导出一个表

			mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名
			mysqldump -u user_name -p database_name table_name > outfile_name.sql
	3. 导出一个数据库结构

			mysqldump -u user_name -p -d –add-drop-table database_name > outfile_name.sql
		- -d 没有数据
		- –add-drop-table 在每个create语句之前增加一个drop table
	4. 带语言参数导出

			mysqldump -uroot -p –default-character-set=latin1 –set-charset=gbk –skip-opt database_name > outfile_name.sql
