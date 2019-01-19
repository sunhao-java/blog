title: Linux下mysql中文乱码解决方法
tags: [Linux, mysql,database, 中文乱码]
date: 2016-03-15
categories: 数据库
---
## 解决MySQL数据库乱码问题总结：

1. 修改安装文件根目录下的my.ini文件：

	1. 搜索字段`default-character-set`，设置其值为`utf8/gbk`之一(注意设置utf8的时候不能设成utf-8)
	2. 再去重启MySQL服务器
	3. 如果还是出现乱码，接着执行下面操作
<!-- more -->

2. 修改数据库编码
	1. 在安装目录的`data `目录下找到你出现乱码的数据库对应的文件夹(这个文件夹即是你这个数据库存放数据的地方)，
	2. 进入找到db.opt文件(即此数据库的编码配置文件)，修改值为下面的

			gbk:default-character-set=gbk
				default-collation=gbk_chinese_ci

			utf8:default-character-set=utf8
				default-collation=utf8_general_ci
	3. 再去重启MySQL服务器
	4. 如果还是出现乱码，接着执行下面操作
3. 再不行，备份原数据库数据，直接drop掉这个数据库

   重新创建数据库并设置编码

		create database yourDB character set utf8;
		或者
		create database yourDB character set gbk;
	别忘了重启MySQL服务器

综上：如果还没有解决，我也没辙了。重装吧，重装的时候设置下编码三处的编码要一致

## 网上找到的
> 系统环境:suse linux server 10,mysql 5.0

> 安装mysql后，默认的字符集是latin1。在linux下安装mysql不像在windows上安装那像，可以选择字符集(即使当时使用了默认的字符集，安装后也可以在安装目录下修改my.ini文件），但是在linux就不太一样了。

1. 在shell输入mysql登陆后:

		mysql>show variables like '%char%';

	回车后显示:

		+---------------------－+-----－－--------------
		| Variable_name | Value
		+-----------------------+-－－------------------
		| character_set_client | latin1
		| character_set_connection | latin1
		| character_set_database | latin1
		| character_set_filesystem | binary
		| character_set_results | latin1
		| character_set_server | latin1
		| character_set_system | utf8
		| character_sets_dir | /usr/share/mysql/charsets/
		+---------------------+－－－-------------------

	这就是它默认的设置。

2. 接下来到/usr/share/mysql/目录下，将my-medium.cnf文件(使用其它实例配置文件也行)拷贝到/

	etc目录下：

		pds:~# cp /usr/share/mysql/my-medium.cnf /etc/my.cnf
		pds:~# vi /etc/my.cnf

	分别在如下几项中添加字符集：

		[client]
		default-character-set=gb2312
		[mysqld_safe]
		default-character-set=gb2312
		[mysqld]
		default-character-set=gb2312
		#default-table-type=innodb
		[mysql]
		default-character-set=gb2312

3. 再重启mysql让配置生效:

		pds:~# service mysql restart

		Shutting down MySQL done
		Starting MySQL done

4. 再次登陆mysql后，查看变量：

		mysql>show variables like '%char%';

		+---------------------－+-----－－--------------
		| Variable_name | Value
		+-----------------------+-－－------------------
		| character_set_client | gb2312
		| character_set_connection | gb2312
		| character_set_database | gb2312
		| character_set_filesystem | binary
		| character_set_results | gb2312
		| character_set_server | gb2312
		| character_set_system | utf8
		| character_sets_dir | /usr/share/mysql/charsets/
		+---------------------+－－－-------------------

5. 如此显示就完成了配置了，在表中插入一条含中文的记录，就不再出现乱码，但是原来插入的记录很可能还是乱码，因为原来的字符集与当前字符集不一致。






