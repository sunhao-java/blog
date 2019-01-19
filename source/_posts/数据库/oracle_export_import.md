title: Oracle_关于oracle导入导出数据库表结构和数据
tags: [Oracle,database, 导入,导出]
date: 2016-03-18
categories: 数据库
---

1. 首先在系统环境变量中新建环境变量`ORACLE_HOME=F:\study\oracle\product\10.1.0\db_1`(视自己数据安装路径而定)
2. 在path环境变量最后添加`%ORACLE_HOME%/BIN`

<!-- more -->

3. 在命令行中输入`exp username/password@dbname file=backup file path`
	- username: 导出数据库用户名
	- password: 导出数据库密码
	- backup file path: 数据库备份文件存放路径

	如：

		exp scott/123456@sunhao file=d:\1.dmp
4. 在命令行中输入`imp tousername/password@dbname file=backup file path fromuser=fromusername touser=tousername`

	- tousername:		要导入的数据库用户名
	- password:		要导入的数据库密码
	- backup file path:	数据库备份文件存放路径
	- fromusername:		要导入的数据库用户名
	- tousername:		导入的那个数据库用户名

	如：

		imp usr_oa_sys/usr_oa_sys@SUNHAO file=d:\oa_show.dmp fromuser=usr_oa_show touser=usr_oa_sys
5、导入导出单张表：

	在exp或者imp语句后面加上`tables(table1,table2,...)`