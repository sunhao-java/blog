title: Linux_oracle管理
tags: [Linux,database, oracle]
date: 2016-03-18
---

## 启动

1. `su - oracle` 切换到oracle用户且切换到它的环境
2. `lsnrctl status` 查看监听及数据库状态
3. `lsnrctl start` 启动监听
4. `sqlplus /nolog` 进入sqlplus
5. `SQL> conn / as sysdba` 以DBA身份登录
6. `SQL> startup` 启动db

<!-- more -->

## 停止

1. `su - oracle` 切换到oracle用户且切换到它的环境
2. `lsnrctl stop` 停止监听
3. `sqlplus /nolog` 进入sqlplus
4. `SQL>conn / as sysdba` 以DBA身份登录
5. `SQL>SHUTDOWN IMMEDIATE` 关闭db

 	其中startup和shutdowm还有其他一些可选参数，有兴趣可以另行查阅

## 查看初始化参数及修改

1. `su - oracle` 切换到oracle用户且切换到它的环境
2. `sqlplus / as sysdba` 进入sqlplus
3. `SQL>conn / as sysdba` 以DBA身份登录
4. `SQL>show parameter session;` 查看所接受的session数量
5. `SQL>alter system set shared_servers=10;` 将shared_servers的数量设置为10

## 数据库连接数目

1. 其中一个数据库连接需要一个session,它的值由processes决定，session与processes通常有以下关系：

		session = 1.1 * processes + 5

2. 以`sysdba `身份登陆PL/SQL 或者 Worksheet
 	- 查询目前连接数

     		show parameter processes;
 	- 更改系统连接数

     		alter system set processes=1000 scope=spfile;
 	- 创建pfile

     		create pfile from spfile;
 	- 重启Oracle服务或重启Oracle服务器

	不过这也不是绝对的，还要受到CPU和内存等硬件条件的限制。另外 processes和session不可以通过alter system语句直接修改，只可以修改服务器参数文件来更改(Server Parameter File)。如果存在一个server parameter file，通过alter system语句所作的更改将会被持久化到文件中。

## 查询Oracle游标使用情况的方法

   	select * from v$open_cursor where user_name = 'TRAFFIC'；

## 查询Oracle会话的方法

   	select * from v$session