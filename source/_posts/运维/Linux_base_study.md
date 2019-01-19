title: Linux_基本学习
tags: [Linux, 学习]
date: 2016-03-14
categories: 运维
---

1. 快捷操作：
	- 切换控制台，由图形转换到控制台模式：ctrl+alt+f1(同时按下3秒钟不要马上松开)。由控制台转向图形模式是：alt+f7 
	- 命令行--->图形界面 startx
	- 修改默认的语言项（在控制台下）：vi /etc/sysconfig/i18n中的LANG=zh_CN.GB18030(注意大小写，然后重启系统即可) 
	- 修改默认启动级别：vi /etc/inittab 修改id:3:initdefault中的3为想要的级别
	- 翻页：shift+pageup/pagedown
	- 关机：shutdown
	- 重启：reboot

<!-- more -->

2. 我的Linux配置

		用户名root		密码sunhao
		用户名sunhao	密码1314520
		IP地址 192.168.220.129	
		广播地址 192.168.220.255	
		子网掩码255.255.255.0

3. Linux中的命令：

	- pwd	查看当前路径名
	- whoami	查看当前登录的用户名
	- ls	列出当前路径下所有的文件和目录
		- ls -l	 按照一定格式(实验)<br/>
				 以d开头的都是目录<br/>
				 以-开头的都是文件<br/>
				 以l开头的都是链接
		- ls -R	按照树状结构列出目录
	- cd	进入一个目录
		- cd ..	返回上一层目录
	- mkdir	 创建一个目录
	- touch	 创建一个文件
	- rmdir	 删除一个目录(只能是空目录)
		- rm -rf 目录	强制删除一个目录(不管是否为空)
		- rm -r 目录	删除一个非空的目录(会询问是否删除此目录的子目录)
	- mount	 挂载
		- mount /dev/cdrom /mnt/cdr	把dev下的cdrom(光驱)挂载在mnt下的cdr目录下
		- umount (设备名/挂载点名)	取消挂载(注意要推出访问设备)
	- cp	拷贝
		- cp 1 2	把文件名为1的文件拷贝一份为文件2
		- cp	-r	s1 s2	把目录s1复制到目录s2下
	- mv	移动文件
		- mv 1 s1	把文件1移动到s1目录下
		- mv -r	同cp -r
		- mv sunhaoInfo sunhao	把名为sunhaoInfo的文件夹改名为sunhao
	- clear	清屏
	- vi	(文本编辑器)往创建好的文件中写入
		- vi 3.txt	如何有则直接写入，如果没有则创建
			- a(回车)	开始编辑模式
			- ESC	退出编辑模式
			- :	返回命令行模式
			- w	存盘
			- q	退出
			- qw	存盘退出
			- q!	不存盘退出
			- dd	删除一行
			- dw	删除一个单词
			- o	往下插入一行
			- O(大写)	往上插入一行
	- more	查看文件内容
		- more 3.txt	查看文件的内容
	- cat		同上(顺序显示)
	- tac		同上(逆序显示)
	- head	同上
		- head -n 3.txt	显示3.txt的前n行内容
	- tail	同上
		- tail -n 3.txt	显示3.txt的后n行内容
	- find	查找文件在哪里
		- find /sunhao -name *.txt	从/sunhao路径下开始查找符合*.txt的文件(查文档)
	- grep	查找字符
		- grep sunhao 1.txt	查找1.txt中字符串sunhao所在的一行
	- whereis		查看命令在哪里
		- whereis ls	查看ls命令所在的位置
	- echo	查看环境变量
		- echo $PATH	查看Linux下的PATH环境变量值(大写)
	- ln		链接命令符
		- ln 3.txt 4	创建3.txt的一个链接4(硬链接，复制了一个文件并在文件中建立一个链接)
		- ln -s 3.txt 5	同上，但是软链接(软链接、符号链接，相当于Windows中的快捷方式)
		- 区别：删掉源文件后硬链接依然存在，而软链接就失去了链接
	- groupadd	添加组
		- groupadd testgroup	添加组testgroup
	- useradd/adduser		添加用户(如果没有指定组就创建一个跟用户名相同的组)
		- useradd testuser	添加testuser用户
		- useradd testuser -g testgroup	添加用户testuser到testgroup用户组
	- passwd	为用户指定密码
		- passwd testuser	为testuser添加密码
		
				(当以普通用户身份登录的时候，直接输入passwd
					1、输入当前密码
					2、输入新密码
					3、再次输入新密码
				)
	- groupdel	删除组(没有用户)
		- groupdel testgroup	删除组testgroup
	- userdel		删除用户(注意要删除用户的主目录，home下)
		- userdel test	删除用户test
	- usermod		为用户更改组
		- usermod -g testgroup testuser	把用户testuser的组改成testgroup
	- su			切换用户

4. Linux中有四种权限：`rwx-	---> read write execute none(读、写、执行、无)`

		权限：-rwxr-xr-x  ---> 分成四组(1、表示文件类型；2、文件所有者的权限；3、文件所有者所在组的人的权限；4、其他人权限)
		chmod	修改权限(+:添加权限 -:取消权限)(按照rwx顺序排列)
		chmod +x 4	给4文件添加权限x(全部的人)
		chmod u+x 4		给当前用户添加4文件的x权限
		chmod g+x 4		给文件所有者所在组的人添加文件4的权限x
		chmod o+x 4		给所有不在同组的人添加文件4的权限x
		chmod 777	特殊的方式修改权限
				9位权限分成3组(所有者，同组人，其他人)
				每一组用3位二进制表示，即752--->111101010(rwxr-x-w-)
		chown	修改文件所有者
		chown sunhao 1.txt	修改1.txt的所有者为sunhao
		
5. 管道	把命令以|隔开，表示依次执行

		ls -l | grep "^d"	先执行列出目录，然后执行查找以d开头的行(即只列出目录)
		ls -l | grep "^d" | wc -l 	输出目录的个数
		wall	警告所有登录的人
		wall date	所有登录的用户都会显示all
		wall ・date・		警告所有登录的人执行date之后的结果(显示当前系统日期时间)
	
6. 重定向	把命令执行的结果写在一个文件中(输出)

		ls > 2.txt	把ls执行结果写在2.txt中(替换)
		ls >> 2.txt		把执行结果写在2.txt中文本的最后(不替换)
		lsss 2> 2.txt	把执行命令的错误信息写入到2.txt中	
7. 重定向	把文本中的信息取出然后输入

		wall < 2.txt	把2.txt中的文本取出警告所有登录的人
		
8. Linux FTP Service

		启动：service vsftpd start	d表示后台启动
		停止：service vsftpd stop
		重启：service vsftpd restart
		登录：ftp localhost
		查IP：ifconfig	查询Linux的IP地址
9. 关闭Linux防火墙	
	
		service iptables stop
10. 允许匿名登录

		cmd连接Linux的FTP： ftp 192.168.94.129
		匿名用户名：		anonymous
		密码：				空
11. Linux中FTP配置文件：/etc/vsftpd/vsftpd.conf
12. Linux中允许root用户登录FTP(记得要重启FTP service)：
		
		修改文件/etc/vsftpd.user_list，把root给注释掉
		修改文件/etc/vsftpd.ftpusers,把root给注释掉
13. Linux中设置vsftp开机自启动：
		
		方法1：vi /ect/rc.local加入以下一句:/user/local/bin/vsftpd &(表示以后台程序启动)
		方法2：chkconfig vsftpd on(查ckconfig的用法)
14. ssh		远程登录
		
		启动：service sshd start(基本与vsftpd相同)
		
15. Linux下安装tomcat
		
	1. 解压文件：先gzip -d 文件名
			  
			  再tar -xvf 上面解压后的文件名
			  tar -zxvf ??.tar.gz
	2. 需要指定JAVA_HOME环境变量(要执行export JAVA_HOME,别人才能使用)
	3. ps -ef		查看所有的进程(重要参数是id)
		
			ps -ef tomcat	查看tomcat进程
	4. 关闭tomcat进程		`kill tomcat进程id`
	5. 设置tomcat开机启动
			
			vi /etc/rc.local
			加上JAVA_HOME=/usr/java/jdk1.6.0_25
			export JAVA_HOME
			/sunhao/tomcat/bin/startup.sh

16. rpm 
	- rpm-qa	查看装了哪些软件
		
			rpm -qa | grep jdk		查看有没有装jdk
	- rpm	-e	卸载(查强制卸载)
			
			rpm -e 安装包名
	- rpm -ivh	安装
			
			rpm -ivh 文件名(rpm文件)

17. 设置全局环境变量
	
		/etc/profile文件	vi /etc/profile	最后一行加上JAVA_HOME=jdk目录

18. Linux下MySQL	用户名：root	密码：1314520
		
		设置密码	mysqladmin -uroot password '1314520'

19. 要是发现我们的应用和数据库服务器系统时间不对可以使用这几个命令修改：
	- 先设置日期
		
			date -s 20080103
	- 设置时间

			date -s 18:24:00
	- 如果要同时更改BIOS时间
			
			再执行clock -w

> export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin