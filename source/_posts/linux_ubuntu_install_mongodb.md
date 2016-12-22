title: Linux_Ubuntu_安装mongoDB
tags: [Linux, Ubuntu,mongoDB]
date: 2016-03-15
---

1. 下载安装包(`mongodb-linux-x86_64-2.4.9.tgz`)

2. 解压

		tar -zxvf mongodb-linux-x86_64-2.4.9.tgz
	重命名，文件夹名称太长
	
		mv mongodb-linux-x86_64-2.4.9 mongodb
<!-- more -->
3. 新建数据库和日志文件夹，日志文件夹很重要，可以很快的定位问题出在哪里
	- 进入mongodb文件夹
  		
  			cd mongodb
	- 新建文件夹
		  
			mkdir data
			mkdir data/db
			mkdir data/log
			touch data/log/mongodb.log	--日志文件

4. 新建启动脚本
	- 进入系统文件夹
			
			cd /etc/init.d			--必须是此文件夹
	- 创建脚本
			
			cd /opt/shell
			touch mongodb
	- 文件内容
			
			!/bin/sh
			/opt/mongodb/bin/mongod --dbpath /opt/mongodb/data/db --logpath /opt/mongodb/data/log/mongodb.log
	- 设置可以运行的权限
			
			chmod u+x mongodb
5. 设置成系统服务，并随计算机启动而启动
	  
	  	update-rc.d mongodb defaults
6. 删除服务
	   
		update-rc.d -f mongodb remove
7. 设置开机启动
		
		vim /etc/rc.local
	最后一行加
		
		#start mongodb
		/opt/shell/mongodb > /opt/logs/mongodb.log

8. 以配置文件方式启动
	写配置文件
	  - 创建目录
	    
	    	mkdir /opt/mongodb/conf
	  - 创建配置文件
	    	
	    	touch /opt/mongodb/conf/mongodb.conf
	  - 配置文件
	    	
	    	# 端口
	    	port=27017							
			#数据目录
	    	dbpath=/opt/mongodb/data/db
	    	#日志文件
	    	logpath=/opt/mongodb/data/log/mongodb.log
	    	#日志是否以append模式添加
	    	logappend=true
	    	#是否静默启动
	    	fork=true									
	  - 回到第3步，文件内容
		    
		    !/bin/sh
		    /opt/mongodb/bin/mongod -f /opt/mongodb/conf/mongodb.conf

9. 设置别名

	mongodb启动之后，在命令行进入数据库是通过命令$MONGO_HOME/bin/mongo。每次这么写很麻烦，是否可以把这一段变成一条命令，就像cd一样
		
		这里用到alias设置别名
		alias mongo='/opt/mongodb/bin/mongo'
		alias mongodb='/etc/init.d/mongodb'

	alias命令永久生效：
		
		在用户主目录下，vim .bashrc，写入上面两行代码，source .bashrc


10. 安装rockmongo
	- `apt-get install php5-dev`
	- `pecl install mongo`
	- `cd /etc/php5/mods-available/`
	- `touch mongo.ini`
	- 写入`extension=mongo.so`
	- `ln -s /etc/php5/mods-available/mongo.ini /etc/php5/apache2/conf.d/mongo.ini`