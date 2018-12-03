title: MongoDB_安装
tags: [MongoDB]
date: 2016-03-18
---

1. 已管理员身份运行命令行
2. 输入命令
<!-- more -->
		mongod --dbpath "F:\mongodb\data\db" --logpath "F:\mongodb\data\log\MongoDB.log" --install --serviceName "MongoDB"

	其中：
	- dbpath: 数据库数据文件路径
	- logpath:	数据库日志文件路径
	- serviceName:	注册的服务名称