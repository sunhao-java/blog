title: keepalived + nginx 实现高可用集群方案
tags: [nginx]
date: 2017-06-05 20:43
---

## 准备工作
1. 两台虚拟机如：
	- 10.211.55.8
	- 10.211.55.10
2. 准备安装文件
	- nginx-1.13.1.tar.gz
	- pcre-8.36.zip
	- keepalived-1.2.22.tar.gz

## 安装`nginx`
1. 参考[安装Linux](http://www.lodsve.com/2016/03/15/linux_ubuntu_install_nginx/)一文，在两台服务器安装nginx
	
	`10.211.55.8` 称为 `nginx1`<br/>
	`10.211.55.10` 称为 `nginx2`
	
## 安装`keepalived`
1. 解压文件

		cd /opt/software
		tar -zxvf keepalived-1.2.22.tar.gz
		cd keepalived-1.2.22
2. 安装

		./configure --prefix=/opt/keepalived
		make && make install
3. 处理配置文件和可执行文件

		cp /opt/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
		cp /opt/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
		mkdir /etc/keepalived
		cp /opt/keepalived/etc/keepalived/keepalived.conf /etc/keepalived
		ln -s /opt/keepalived/sbin/keepalived /usr/sbin/
4. 启动服务

		/etc/init.d/keepalived start
		
5. 按照相同的步骤在另一台服务器安装`keepalived`

## 修改配置文件
`vim /etc/keepalived/keepalived.conf`

	! Configuration File for keepalived
	
	global_defs {
	   # keepalived 自带的邮件提醒需要开启 sendmail 服务。建议用独立的监控或第三方 SMTP,也可配置邮件发送
	   router_id 10.211.55.8
	}
	
	vrrp_script chk_nginx {
	    # 运行脚本，脚本内容下面有，就是起到一个nginx宕机以后，自动开启服务
	    script "/opt/shell/nginx_check.sh"
	    # 检测时间间隔
	    interval 2
	    # 如果条件成立的话，则权重 -20
	    weight -20
	}
	
	# 定义虚拟路由，VI_1 为虚拟路由的标示符，自己定义名称
	vrrp_instance VI_1 {
	    # 来决定主从(从：BACKUP)
	    state MASTER
	    # 绑定虚拟 IP 的网络接口，根据自己的机器填写
	    interface eth0
	    # 虚拟路由的 ID 号， 两个节点设置必须一样
	    virtual_router_id 121
	    # 填写本机ip
	    mcast_src_ip 10.211.55.10
	    # 节点优先级,主节点比从节点优先级高
	    priority 100
	    # 优先级高的设置 nopreempt 解决异常恢复后再次抢占的问题
	    nopreempt
	    # 组播信息发送间隔，两个节点设置必须一样，默认 1s
	    advert_int 1
	
	    authentication {
	        auth_type PASS
	        auth_pass 1111
	    }
	
	    # 将 track_script 块加入 instance 配置块
	    track_script {
	        #执行 Nginx 监控的服务
	        chk_nginx
	    }
	
	    virtual_ipaddress {
	        # 虚拟ip,也就是解决写死程序的ip怎么能切换的ip,也可扩展，用途广泛。可配置多个。
	        10.211.55.100
	    }
	}
	
以上配置为主节点配置，从节点类似，有区别的已经标明。下面是监控服务脚本内容

	#!/bin/bash
	A=`ps -C nginx –no-header |wc -l`
	if [ $A -eq 0 ];then
	    /opt/nginx/sbin/nginx
	    sleep 2
	    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
	        killall keepalived
	    fi
	fi
	
定义变量，此变量为检查nginx进程脚本，如果进程为0，则启动nginx服务，再次检查nginx服务，如果仍没启动 ，杀掉所有keepalived的进程。

## 测试高可用
1. 打开浏览器，输入虚拟ip `10.211.55.100`
2. 显示的是`nginx1`的页面
3. 此时可以停掉一台nginx服务器

		/opt/nginx/sbin/nginx -s stop
4. 这时候，单独访问这台服务器，发现还是可以的，也就是说，`keepalived`监控到`nginx`服务down掉了，然后自动重启这台机器了。
5. 这时再停掉一台`keepalived`服务器

		/etc/init.d/keepalived stop
6. 刷新浏览器，显示的将是另外一台`nginx`
7. 这时就可以证明已经实现**`高可用`**