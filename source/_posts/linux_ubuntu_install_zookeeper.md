title: Linux_Ubuntu_安装zookeeper
tags: [Linux, Ubuntu, zookeeper]
date: 2016-03-15
---

1. 下载zookeeper，http://archive.apache.org/dist/zookeeper/
2. 解压到: /opt/zookeeper-3.4.6下载
3. 把conf下的`zoo_sample.cfg` copy一份后重命名为: `zoo.cfg`.
<!-- more -->
		文件中内容如下:
		syncLimit=5
		initLimit=10
		tickTime=2000
		clientPort=2181
		dataDir=/opt/zookeeper-3.4.6/data
		dataLogDir=/opt/zookeeper-3.4.6/logs

4. 新建zookeeper-3.4.6下的data,log目录,然后
		
		chmod u+x /opt/zookeeper-3.4.6/bin/*
5. 新建shell脚本`startZookeeper.sh`在`/opt/shell`文件夹下，
		
		chmod u+x /opt/shell/startZookeeper.sh
	内容：
		
		#!/bin/sh
		#to start zookeeper
		#set environment
		export ZOOKEEPER_INSTALL=/opt/zookeeper-3.4.6
		export PATH=$PATH:$ZOOKEEPER_INSTALL/bin
		PID_FILE=$ZOOKEEPER_INSTALL/data/zookeeper_server.pid
		FLAG=$1
	
		if [ "$FLAG" = "os_start" ] && [ -f "$PID_FILE" ]
		then
				rm $PID_FILE
		fi
	
		#start zookeeper
		$ZOOKEEPER_INSTALL/bin/zkServer.sh start
6. 设置zookeeper开机启动
	
		vim /etc/rc.local
		在最后一行添加如下两句话
		#start zookeeper
		/opt/shell/startZookeeper.sh