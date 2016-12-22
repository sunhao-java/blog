title: Windows下创建软连接
tags: [Windows]
date: 2016-03-18
---

# 要求:
1. 文件类型为NFS
2. 使用junction

	`下载junction(http://technet.microsoft.com/en-us/sysinternals/bb896768)`

<!-- more -->

# 创建软连接
语法:

	junction [-s] destDir srcDir
	举例:
	junction -s message-test\src\main\webapp\js\easyjs message-easyjs\easyjs

# 删除软连接
语法:
	
	junction -d linkPath
	举例
	junction -d message-test\src\main\webapp\js\easyjs