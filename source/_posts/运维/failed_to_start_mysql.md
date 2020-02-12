title: 记一次MySQL启动失败原因的分析
tags: [MySQL]
date: 2020-02-12
categories: 运维
---
> 今天在我的Windows电脑上准备启动MySQL服务，测试一点东西，但是发现每次都启动不成功，所以就引出本次的分析

<!-- more -->

## 排查启动失败的原因
1. 首先，我的MySQL服务已经注册到windows的服务中了，所以需要查看windows服务的启动日志
2. 打开windows的`事件查看器`(windows开始图标上右键-事件查看器)
3. 左边菜单树上选择`Windows 日志 - 应用程序`
![](/imgs/mysql/1.png)

4. 右边操作栏选择`筛选当前日志...`，设置事件级别为`错误`如下图：
![](/imgs/mysql/2.png)

5. 通过筛选查看，可以发现如下信息，MySQL的端口被占用了
![](/imgs/mysql/3.png)

## 排查端口占用情况
1. 在Windows下查看端口被占用的情况（被哪个进程占用了）

	```
	C:\Users\sunha>netstat -ano|findstr 3306
	  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       4564
	```
2. 记住最右一列的值，此处是`4564`，这个就是占用进程的`pid`
3. 根据上面pid查改进程跑的是哪个应用程序

	```
	C:\Users\sunha>tasklist|findstr 4564
vmnat.exe                     4564 Services                   0      5,364 K
	```
4. 到此时已经查到哪个应用程序在作祟，就是`vmnat.exe`
5. 这个到底是什么玩意？根据名称`vm`大概能猜到是虚拟机，再根据`nat`能猜到网络
6. 所以此处应该是我电脑里安装的虚拟机网络造成的

## 解决问题
1. 此时我已经回忆起来了，我电脑里安装了`VMware`
2. 而且我的网络策略是`nat模式`
3. 为了将虚拟机里的MySQL给同事使用，就做了一个映射出来
4. 一查，果然是的了
![](/imgs/mysql/4.png)

5. 删除这个配置，Windows上的MySQL就可以顺利启动了。