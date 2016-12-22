title: Linux_检查和收集Linux硬件信息的7个命令
tags: [Linux, 检查, 收集, 硬件]
date: 2016-03-14
---

1. lscpu用于查询CPU信息

2. lshw显示硬件信息表

    > 这个命令应用普遍，它可通过个人需求而列出多种不同的硬件参数：CPU、内存、硬盘、USB控制器、lshw卡片等等，本质上就是从/proc目录不同文件中中提取对应的硬件信息。
    
	    按照下面的步骤去安装lshw工具，然后就可以使用了。
	    wget http://ezix.org/software/files/lshw-B.02.14.tar.gz
	    tar -zxvf lshw-B.02.14.tar.gz
	    cd lshw-B.02.14
	    make && make install
	    
<!-- more -->

3. hwinfo-硬件信息

    > hwinfo类似于lshw，也能查询硬件信息，且应用广泛。它也能输出多个硬件部分的详细或者简要信息，但是不同的是有时hwinfo比lshw的信息更详细。
	    
	    默认情况下，Linux系统没有安装hwinfo工具，所以你需要按照以下步骤自己安装：
	    CentOS 6
	        #rpm -Uvh http://mirror.symnds.com/distributions/gf/el/6/gf/x86_64/gf-release-6-6.gf.el6.noarch.rpm
	        #yum list hwinfo
	        #yum install hwinfo
	    CentOS 5
	        #rpm -Uvh http://mirror.symnds.com/distributions/gf/el/5/gf/x86_64/gf-release-5-6.gf.el5.noarch.rpm
	        #yum list hwinfo
	        #yum install hwinfo

4. lspci

    > lsppci命令可列出PCI总线的信息以及连接到PCI总线上的设备信息，比如VGA适配器、SATA控制器、其他模块等等。lspci工具是pciutils包的一部分，所以在安装lspci之前，你需要安装pciutils包。
    
	    安装pciutils包使用下面的命令：
	    yum install pciutils

5. lsusb-列出USB总线信息
6. 
	    这个命令可列出USB控制器的设备信息。
	    lsusb工具是usbutils包的一部分，所以你需要按照如下命令安装：
	    #yum install usbutils

6. lsblk-列出块设备的信息

    	[root@devops tmp]# lsblk

7. lsscsi-列出SCSI的设备信息

	    列出SCSI/SDAT设备的信息，比如硬盘驱动器、光盘驱动器。
	    [root@devops tmp]# lsscsi