title: Linux下远程拷贝文件夹
tags: [Linux,拷贝]
date: 2016-03-15
categories: 运维
---

`scp -r /opt/test root@202.119.200.201:/opt/`


> 远程从一台机器拷贝文件或者文件夹到另一台机器
<!-- more -->
	scp [-r] dir name@ip:dir

	1. scp		远程拷贝命令
	2. -r		如果是拷贝文件夹，需要这个参数，表示向子目录递归；如果是拷贝文件，则不需要
	3. dir		源机器上的源文件路径
	4. name		目标机器的用户
	5. ip		目标机器IP
	6. dir		拷贝到目标机器哪个目录