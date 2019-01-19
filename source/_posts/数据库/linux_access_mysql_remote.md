title: Linux_远程访问Linux下的mysql
tags: [Linux, database, mysql]
date: 2016-03-14
categories: 数据库
---

1. 首先尝试在mysql内赋予该账户从远程IP访问的权限。

		mysql -u root -p
		use mysql;
		登陆以后运行以下命令，给予远程访问客户端权限..
		grant all on *.* to 'ray7hu'@'192.168.1.5' identified by 'password';
		其中：ray7hu表示用户名.  '192.168.1.5' 是需要赋予访问权限的Iip地址；password表示远程登陆密码.
		mysql> grant all on *.* to user_name@'%'identified by'user_password';  mysql> grant all on *.* to user_name@'%' identified by 'user_password';
		上面的命令授予的用户权限可以访问mysql中的任意数据库和表.

<!-- more -->

2. 取消mysql本机绑定

		也就是取消mysql默认的只能本机访问的限制。
		编辑/etc/mysql/my.cnf（ubuntu下默认mysql的话就是my.cnf，不然就有可能不是这个文件名）；找到
		# Instead of skip-networking the defaultis now to listen only on
		# localhost which is more compatible and is not less secure.
		bind-address = 127.0.0.1
		将"bind-address = 127.0.0.1"注释掉（前面加上个#即可）；如果有需要，可以将127.0.0.1修改成特定的IP地址。
		修改之后需要重启一下mysql服务才能生效。命令：
		sudo /etc/init.d/mysql restart
		至此已经能够远程访问MySQL，但是你也许会发现能访问但是看不到数据库。这是因为还没有授予访问特定数据库的权限。

3. 给以存在的数据库授权

		如果用户test 经常在远程IP地址192.168.1.5的客户端访问webdb数据库，那么在服务器端执行的命令应该为（改表法）：
		mysql> use mysql
		mysql> update db set Host='192.168.1.5'where Db='webdb';
		mysql> update user set Host='192.168.1.5'where user='test';
		至此，如果没有意外的话，应该就能远程访问数据库了。