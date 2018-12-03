title: Linux_Ubuntu_安装rabbitmq
tags: [Linux, Ubuntu, rabbitmq]
date: 2016-03-15
---

1. 需要3个文件

		otp_src_17.3.tar.gz
		simplejson-3.6.4.tar.gz
		rabbitmq-server-generic-unix-3.3.5.tar.gz
<!-- more -->
2. 安装erlang
	1. tar -zxvf otp_src_17.3.tar.gz
	2. cd otp_src_17.3/
	3. ./configure
		> 备注：
			
		- [无碍]
			
				configure: WARNING:
						wxWidgets must be installed on your system.
		
				Please check that wx-config is in path, the directory
				where wxWidgets libraries are installed (returned by
				'wx-config --libs' or 'wx-config --static --libs' command)
				is in LD_LIBRARY_PATH or equivalent variable and
				wxWidgets version is 2.8.4 or above.
		- 需要解决
		  
				configure: error: No curses library functions found
				configure: error: /bin/bash '/opt/software/otp_src_17.3/erts/configure' failed for erts

		  解决：
		  
		  		apt-cache search ncurses
				apt-get install libncurses5-dev
	4. make
	5. make install

3. 安装simplejson
	- tar -zxvf simplejson-3.6.4.tar.gz
	- cd simplejson-3.6.4
	- python setup.py install

4. 安装rabbitmq-server
	- `tar -zxvf rabbitmq-server-generic-unix-3.3.5.tar.gz`
	- `cd rabbitmq_server-3.3.5/etc/rabbitmq`
	- `touch rabbitmq.config`
	- `vim rabbitmq.config`
		
		添加内容：
		
			[{rabbit, [{loopback_users, []}]}].
	- 启动：
		1. cd rabbitmq_server-3.3.5/sbin
		2. 启用监控台web:
			
				./rabbitmq-plugins enable rabbitmq_management
		3. 启动
				
				./rabbitmq-server
	- 外网访问：
		
			http://ip:15672

5. 制作开机启动
	1. vim /etc/rc.local
	2. 最后exit 0之前加上：
			
			#start rabbitMq
			/opt/rabbitmq_server-3.3.5/sbin/rabbitmq-server -detached > /opt/logs/rabbitmq.log