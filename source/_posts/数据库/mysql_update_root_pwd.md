title: MySQL_修改root帐号密码
tags: [MySQL, database, root, 密码]
date: 2016-03-18
categories: 数据库
---

> 如果你忘了 MySQL 的 root 帐号密码，别担心，使用下面步骤就可以重设一个新密码：

1. 首先停止 MySQL 服务 `/etc/init.d/mysql stop`
2. 启动 MySQL 服务并屏蔽用户权限检查，可通过如下命令：
3.
<!-- more -->

		mysqld_safe --skip-grant-tables
    记住，当你使用这个参数启动服务时，任何人无需密码即可连接到 MySQL 并拥有最高权限，因此你需要确定你在做这个的过程中，别让其他人访问数据库。可以使用如下命令来拒绝网络连接：

    	mysqld_safe --skip-grant-tables --skip-networking
    这样就只能通过本机连接到 MySQL 服务器
3. 然后使用 `mysql -u anything` 命令来连接到 MySQL
4. 执行 SQL 语句：

		update mysql.user set Password=PASSWORD(‘new-password’) WHERE User=’root’
5. 用 `mysqladmin -u qualquer_coisa shutdown` 命令停止 MySQL 服务，然后以正常模式启动 MySQL 即可
