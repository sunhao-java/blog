title: Linux_Ubuntu_安装Oracle
tags: [Linux, Ubuntu, database, Oracle]
date: 2016-03-15
---

1. 安装必需的包

		apt-get install gcc make binutils lesstif2 libc6 libc6dev rpm libmotif3 libaio1 libstdc++6 alien

2. 创建用户

		adduser oracle
<!-- more -->
3. 设置swap区

	Oracle10g至少需要500M的内存和400M的交换空间，要查看swap区是否足够大小，用 `fdisk l` 命令去查，如果小于400M的空间，那么就要增加swap的大小

	重设交换分区可以使用如下操作:

		dd if=/dev/zero of=tmp_swap bs=1k count=900000
		chmod 600 tmp_swap
		mkswap tmp_swap
		swapon tmp_swap
		完成安装以后,可以释放这个空间:
		swapoff tmp_swap
		rm tmp_swap

4. 修改 `sysctl.conf`

		kernel.shmmax = 3147483648
		kernel.shmmni = 4096
		kernel.shmall = 2097152
		kernel.sem = 250 32000 100 128
		fs.filemax = 65536
		net.ipv4.ip_local_port_range = 1024 65000
6. 修改 `limits.conf`

		* soft nproc 2407
		* hard nproc 16384
		* soft nofile 1024
		* hard nofile 65536

6. 让修改生效

	修改了以上文件后,必须让其生效,或重启系统,或切换到 root 用户下用以下的方式改变内核运行参数: `sysctl p `

7. 产生相应的软连接

	创建一个文件如 kk，内容如下：

		#!/bin/bash
		ln s /usr/bin/awk /bin/awk
		ln s /usr/bin/rpm /bin/rpm
		ln s /usr/bin/basename /bin/basename
		mkdir /etc/rc.d
		ln s /etc/rc0.d /etc/rc.d/rc0.d
		ln s /etc/rc2.d /etc/rc.d/rc2.d
		ln s /etc/rc3.d /etc/rc.d/rc3.d
		ln s /etc/rc4.d /etc/rc.d/rc4.d
		ln s /etc/rc5.d /etc/rc.d/rc5.d
		ln s /etc/rc6.d /etc/rc.d/rc6.d
		ln s /etc/init.d /etc/rc.d/init.d

	创建后，切换到 root 用户去执行一下。

8. 创建RedHat的版本声明文件

	在/etc/redhatrelease中添加以下语句，以使安装程序认为正在一个RedHat的系统上安装：

		Red Hat Linux release 3.1 (drupal)

9. 修改环境变量

	编辑 /home/oracle/.bashrc，增加以下export 的内容。
(注意，在Ubnutu 7.04中用户的profile文件已改名为~/.profile，有很多安装教程都是用 ~/.bash_profile，在7.04中不行的)

		export ORACLE_HOME=/opt/oracle
		export ORACLE_OWNER=oracle
		export ORACLE_SID=oracle
		export ORACLE_TERM=xterm
		export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/Apache/Apache/bin:$PATH

10. 开始安装

	注销原来的用户，改用oracle用户登录。用env查看一下环境变量是否生效。 然后进行`/ora_ins_disk`中进行安装，执行安装脚本时还需要以root权限创建目录`/opt/oracle`

		sudo mkdir /opt/oracle
		sudo chown R oracle:oracle /opt/oracle
		sudo chmod R 770 /opt/oracle
		cd/ora_ins_disk
		./runInstaller

	在安装过程中，请使用 Advanced Installation，然后一路按默认的设置进行往下设置，到窗单名为 “Specify Database Configuration Options”的时候，要修改以下设置： Database Character Set 中选择 Simplified Chinese ZHS16GBK 在安装的后期，系统提示需要用 root 用户去运行两个脚本文件orainstRoot.sh和root.sh，安装完毕后，Oracle是正常启动着的，你可以试一下连接数据库，同时也可以使用浏览器去设置一下Oracle，（url:http: //localhost:1158/em/）（Oracle 10g与之前的版本都不一样，使用WEB页的企业管理器来代替以前的C/S版JAVA企业管理器）

11. 启动服务,一般采用手动:

	Ubuntu下启动Oracle，启动oracle必须在你安装oracle的那个账户上进行的.

	手动启动oracle:

	    1.先在命令的模式下启动监听
		lsnrctl start
	    2.然后使用sqlplus来启动oracle
		sqlplus / as sysdba
		startup
		exit

	能看到oracle启动成功的消息就ok了。


> 安装过程中,可能会出错,解决方案:

1. 问题：

	调用makefile `．．/sqlplus/lib/ins_sqlplus.mk` 的目标`install` 时出错。请参阅`/home/oracle/oraInventory/logs/installActions20111206_110318AM.log`以了解详细信息。  

  	解决办法：

		$ORACLE_HOME/ /sqlplus/lib/env_sqlplus.mk添加一行：EXPDLIBS=lclntsh ，然后点击“重试”按钮， ok.

2. 问题：

	调用makefile `．．/sysman/lib/ins_sysman.mk` 的目标`agent nmo nmb` 时出错。请参阅`../oraInventory/logs/installActions20111206_110318AM.log` 以了解详细信息。 

  	解决办法：

		1. 降低gcc的版本,oracle10g的gcc是3.4左右的版本.使用gcc3.4_3.4.66ubuntu3_i386.deb.
		2. 在ubuntu中有可能我们的gcc版本过高或者过低，需要改变到合适的版本，，，
		3. 在/usr/bin/目录下，我们可以看到一些gcc开头的文件，其中有一个是gcc，用ls命令看一下，他是个链接文件，链接到当前的gcc文件，也即是说，
		4. 他是连接到当前使用的gcc上的，所以改变他的链接源文件就可以了，假如我们现在的gcc是gcc4.6,我们要降级到gcc3.4，我们先下载一个gcc3.4安装，
		5. 会在/usr/bin目录下看到gcc3.4这个文件，然后在/usr/bin目录下删除(备份)gcc这个文件，然后执行
		6. ln s gcc3.4 gcc ,这样之后，执行：gcc v

3. 问题:

	`libstdc++.so.5`找不到`No such file or directory`

  	解决办法:

	1. 下载安装包：

	  	请到ubuntu的官方网站的packages栏目`http://packages.ubuntu.com/precise/amd64/libstdc++5/download`选择一个可用的链接来下载.deb文件。

	  	我已下载:`libstdc++5_3.3.625ubuntu1_amd64.deb`
	2. 解包为libstdc++5：

		`dpkg x libstdc++5_3.3.625ubuntu1_amd64.deb libstdc++5`
	3. 复制到系统lib目录：

		`sudo cp libstdc++5/usr/lib/x86_64linuxgnu/libstdc++.so.5.0.7 /usr/lib`
	4. 进入系统lib目录建libstdc++5链接：

		`cd /usr/lib`<br/>
		`sudo ln s libstdc++.so.5.0.7 libstdc++.so.5`

4. 问题:

	`libgcc_s.so.1`找不到

  	解决方案: 不予理会