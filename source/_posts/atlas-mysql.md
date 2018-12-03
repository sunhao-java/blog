title: 通过Atlas实现MySQL读写分离
tags: [MySQL, Atlas, database]
date: 2017-05-23
---
> 最近公司项目要求MySQL高可用，加上以前公司听过QiHoo360的Atlas，所以就尝试搭建了一个MySQL读写分离，并且高可用的。

<!-- more -->

## 前期准备
1. 准备4台机器，系统为`CentOS release 6.6`
2. Ip分别为`192.168.20.121`、`192.168.20.122`、`192.168.20.123`、`192.168.20.124`
3. 4台机器分别作为`Atlas代理服务`，`master MySQL`，`slave MySQL 1`，`slave MySQL 2`
4. 下载QiHoo360的Atlas [地址](https://github.com/Qihoo360/Atlas/releases)

## 安装Atlas
1. 下载得到`Atlas-XX.el6.x86_64.rpm`安装文件
2. `sudo rpm –i Atlas-XX.el6.x86_64.rpm`安装
3. 安装在`/usr/local/mysql-proxy`
4. 安装目录分析
	- `bin`
		- 可执行文件
		- `encrypt`用来加密密码，后面会用到
		- `mysql-proxy`是MySQL自己的读写分离代理
		- `mysql-proxyd`操作Atlas
		- `VERSION`
	- `conf`
		- `test.cnf`配置文件
		- 一个文件为一个实例，实例名即文件名，启动需要带上这个实例名
	- `lib`依赖包
	- `log`记录日志
5. 启动命令：`/usr/local/mysql-proxy/bin/mysql-proxyd [实例名] start`
6. 停止命令：`/usr/local/mysql-proxy/bin/mysql-proxyd [实例名] stop`
7. 同理，`restart`为重启，`status`为查看状态
8. 配置文件解释

	请查看[官方文档](https://github.com/Qihoo360/Atlas/wiki/Atlas%E7%9A%84%E5%AE%89%E8%A3%85)
	
## 数据库配置
1. 1台master2台slave，都要配置相同的用户名密码，且都要可以远程访问
2. 分别进入3台服务器，创建相同的用户名密码，创建数据库`test`，设置权限

		CREATE USER 'test'@'%' IDENTIFIED BY 'test123';
		CREATE USER 'test'@'localhost' IDENTIFIED BY 'test123';
		grant all privileges on test.* to 'test'@'%' identified by 'test123';
		grant all privileges on test.* to 'test'@'localhost' identified by 'test123';
		flush privileges;
3. 主从数据库配置
	1. 配置master服务器
		- 找到MySQL配置文件`my.cnf`，一般在`etc`目录下
		- 修改配置文件

				[mysqld]
				# 一些其他配置
				... 
				  
				#主从复制配置  
				innodb_flush_log_at_trx_commit=1  
				sync_binlog=1  
				#需要备份的数据库
				binlog-do-db=test
				#不需要备份的数据库
				binlog-ignore-db=mysql  
				  
				#启动二进制文件  
				log-bin=mysql-bin  
				  
				#服务器ID  
				server-id=1  
				  
				# Disabling symbolic-links is recommended to prevent assorted security risks  
				symbolic-links=0
		- 重启数据库`service mysql restart`
		- 进入数据库，配置主从复制的权限

				mysql -uroot -p123456
				grant replication slave on *.* to 'test'@'127.0.0.1' identified by 'test123';
		- 查看主数据库信息，记住下面的`File`与`Position`的信息，它们是用来配置从数据库的关键信息。

				mysql> show master status;
				+------------------+----------+--------------+------------------+
				| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
				+------------------+----------+--------------+------------------+
				| mysql-bin.000002 | 17620976 | test         | mysql            |
				+------------------+----------+--------------+------------------+
				1 row in set (0.00 sec)
	2. 配置两台salve服务器
		- 找到配置文件`my.cnf`
		- 修改配置文件如下

				[mysqld]
				# 一些其他配置
				... 
				
				# 几台服务器不能一样
				server-id=2
				
				# Disabling symbolic-links is recommended to prevent assorted security risks
				symbolic-links=0
		- 进入数据库，配置从数据库的信息，这里输入刚才记录下来的`File`与`Position`的信息，并且在从服务器上执行：
				
						# master数据库的ip
				mysql> change master to master_host='192.168.20.122',
						# master的用户名
				    -> master_user='buck',
				    	# 密码
				    -> master_password='hello',
				    	# 端口
				    -> master_port=3306,
				    	# master数据库的`File `
				    -> master_log_file='mysql-bin.000002',
				    	# master数据库的`Position`
				    -> master_log_pos=17620976,
				    -> master_connect_retry=10;
		- 启动进程

				mysql> start slave;
				Query OK, 0 rows affected (0.00 sec)
		- 检查主从复制状态，要看到下列`Slave_IO_Running`、`Slave_SQL_Running `的信息中，两个都是`Yes`，才说明主从连接正确，如果有一个是No，需要重新确定刚才记录的日志信息，停掉“stop slave”重新进行配置主从连接。

				mysql> show slave status \G;
				*************************** 1. row ***************************
				               Slave_IO_State: Waiting for master to send event
				                  Master_Host: 192.168.246.134
				                  Master_User: buck
				                  Master_Port: 3306
				                Connect_Retry: 10
				              Master_Log_File: mysql-bin.000002
				          Read_Master_Log_Pos: 17620976
				               Relay_Log_File: mysqld-relay-bin.000002
				                Relay_Log_Pos: 251
				        Relay_Master_Log_File: mysql-bin.000002
				             Slave_IO_Running: Yes
				            Slave_SQL_Running: Yes
				              Replicate_Do_DB: 
				          Replicate_Ignore_DB: 
				           Replicate_Do_Table: 
				       Replicate_Ignore_Table: 
				      Replicate_Wild_Do_Table: 
				  Replicate_Wild_Ignore_Table: 
				                   Last_Errno: 0
				                   Last_Error: 
				                 Skip_Counter: 0
				          Exec_Master_Log_Pos: 17620976
				              Relay_Log_Space: 407
				              Until_Condition: None
				               Until_Log_File: 
				                Until_Log_Pos: 0
				           Master_SSL_Allowed: No
				           Master_SSL_CA_File: 
				           Master_SSL_CA_Path: 
				              Master_SSL_Cert: 
				            Master_SSL_Cipher: 
				               Master_SSL_Key: 
				        Seconds_Behind_Master: 0
				Master_SSL_Verify_Server_Cert: No
				                Last_IO_Errno: 0
				                Last_IO_Error: 
				               Last_SQL_Errno: 0
				               Last_SQL_Error: 
				1 row in set (0.00 sec)
				
				ERROR: 
				No query specified
				
## Atlas配置
1. 使用Atlas的加密工具对上面用户的密码进行加密

		/usr/local/mysql-proxy/bin/encrypt test123
		29uENYYsKLo=
2. 配置atlas
	- 这是用来登录到Atlas的管理员的账号与密码，与之对应的是`Atlas监听的管理接口IP和端口`，也就是说需要设置管理员登录的端口，才能进入管理员界面，默认端口是`2345`，也可以指定IP登录，指定IP后，其他的IP无法访问管理员的命令界面。方便测试，我这里没有指定IP和端口登录。
	- 配置主数据的地址与从数据库的地址，这里配置的主数据库是122，从数据库是123、124

			#Atlas后端连接的MySQL主库的IP和端口，可设置多项，用逗号分隔
			proxy-backend-addresses = 192.168.20.122:3306
			#Atlas后端连接的MySQL从库的IP和端口，@后面的数字代表权重，用来作负载均衡，若省略则默认为1，可设置多项，用逗号分隔
			proxy-read-only-backend-addresses = 192.168.20.123:3306@1,192.168.20.124:3306@2
	- 这个是用来配置MySQL的账户与密码的，就是上面创建的用户，用户名是`test`,密码是`test123`,刚刚使用Atlas提供的工具生成了对应的加密密码

			pwds = buck:RePBqJ+5gI4=
3. 启动Atlas

		[root@localhost /usr/local/mysql-proxy/bin]# ./mysql-proxyd test start
		OK: MySQL-Proxy of test is started
		
## 测试
1. 进入atlas的管理界面
		
		[root@localhost ~]#mysql -h127.0.0.1 -P2345 -uuser -ppwd
		Welcome to the MySQL monitor.  Commands end with ; or \g.
		Your MySQL connection id is 1
		Server version: 5.0.99-agent-admin
		
		Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.
		
		Oracle is a registered trademark of Oracle Corporation and/or its
		affiliates. Other names may be trademarks of their respective
		owners.
		
		Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
		
		mysql> select * from help;
		+----------------------------+---------------------------------------------------------+
		| command                    | description                                             |
		+----------------------------+---------------------------------------------------------+
		| SELECT * FROM help         | shows this help                                         |
		| SELECT * FROM backends     | lists the backends and their state                      |
		| SET OFFLINE $backend_id    | offline backend server, $backend_id is backend_ndx's id |
		| SET ONLINE $backend_id     | online backend server, ...                              |
		| ADD MASTER $backend        | example: "add master 127.0.0.1:3306", ...               |
		| ADD SLAVE $backend         | example: "add slave 127.0.0.1:3306", ...                |
		| REMOVE BACKEND $backend_id | example: "remove backend 1", ...                        |
		| SELECT * FROM clients      | lists the clients                                       |
		| ADD CLIENT $client         | example: "add client 192.168.1.2", ...                  |
		| REMOVE CLIENT $client      | example: "remove client 192.168.1.2", ...               |
		| SELECT * FROM pwds         | lists the pwds                                          |
		| ADD PWD $pwd               | example: "add pwd user:raw_password", ...               |
		| ADD ENPWD $pwd             | example: "add enpwd user:encrypted_password", ...       |
		| REMOVE PWD $pwd            | example: "remove pwd user", ...                         |
		| SAVE CONFIG                | save the backends to config file                        |
		| SELECT VERSION             | display the version of Atlas                            |
		+----------------------------+---------------------------------------------------------+
		16 rows in set (0.00 sec)
		
		mysql> 
2. 使用工作接口来访问

		[root@localhost ~]#mysql -h127.0.0.1 -P1234 -utest -ptest123
		Welcome to the MySQL monitor.  Commands end with ; or \g.
		Your MySQL connection id is 34
		Server version: 5.0.81-log MySQL Community Server (GPL)
		
		Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.
		
		Oracle is a registered trademark of Oracle Corporation and/or its
		affiliates. Other names may be trademarks of their respective
		owners.
		
		Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
		
		mysql> show databases;
		+--------------------+
		| Database           |
		+--------------------+
		| information_schema |
		| mysql              |
		| performance_schema |
		| test               |
		+--------------------+
		4 rows in set (0.00 sec)
		
		mysql> 
		
## 使用可视化管理工具`Navicat`登录
使用用户名`test`、密码`test123`、端口`1234`、地址`192.168.20.121`正常登录。注意，这里登录的是atlas服务器，不再是任何一个MySQL服务器