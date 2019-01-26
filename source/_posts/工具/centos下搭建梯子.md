title: centos下搭建梯子
tags: [Centos, 梯子]
date: 2019-01-23
categories: 工具
---

由于众所周知的一些原因，在我天朝内(JU)部(YU)网(WANG)络并不能访问`谷哥`，而`度娘`的不务正业，让我们撸码的人搜索感觉到很蛋疼。所以为了能用上`谷哥`，还能看看墙外的世界，这时候就需要一把梯子了，这里只介绍如果去搭建梯子，至于如何去造梯子，以后再说吧。

<!-- more -->

# 准备工作
1. 操作系统环境

        [root@sunhao-centos ~]$ cat /etc/redhat-release
        CentOS Linux release 7.6.1810 (Core) 
        [root@sunhao-centos ~]$ uname -a
        Linux sunhao-centos 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
2. 梯子是用`shadowsocks`来制造的(shadowsocks并不分客户端与服务端，只是运行哪个文件[sslocal/ssserver])
3. 这里假设梯子已经造好了，相关信息如下：
    
        {
            "server": "x.x.x.x",
            "server_port": 12345,
            "local_port": 1080,
            "password": "password",
            "timeout": 600,
            "method": "rc4-md5"
        }

# 安装`shadowsocks`
1. 因为要用`pip`来安装`shadowsocks`，而`pip`又依赖于`python`的环境
2. 一般Linux已经安装过`python`了，如果没有安装，则自行安装配置
3. `pip`的源在默认的源中是不存在的，所以要安装`epel源`

        # 命令
        rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
        # 效果
        [root@sunhao-centos ~]# rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
        Retrieving http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
        warning: /var/tmp/rpm-tmp.M4UWJg: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
        Preparing...                          ################################# [100%]
        Updating / installing...
           1:epel-release-7-11                ################################# [100%]
4. 这时就可以正常安装`pip`了

        # 命令
        yum install python-pip
        # 效果
        [root@sunhao-centos ~]# yum install python-pip
        Loaded plugins: fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        epel/x86_64/metalink                                                                                                                                                                                                                    | 8.6 kB  00:00:00     
         * base: mirrors.aliyun.com
         * epel: mirrors.huaweicloud.com
         * extras: mirrors.163.com
         * updates: mirrors.aliyun.com
        base                                                                                                                                                                                                                                    | 3.6 kB  00:00:00     
        epel                                                                                                                                                                                                                                    | 4.7 kB  00:00:00     
        extras                                                                                                                                                                                                                                  | 3.4 kB  00:00:00     
        updates                                                                                                                                                                                                                                 | 3.4 kB  00:00:00     
        (1/3): epel/x86_64/group_gz                                                                                                                                                                                                             |  88 kB  00:00:00     
        (2/3): epel/x86_64/updateinfo                                                                                                                                                                                                           | 955 kB  00:00:01     
        (3/3): epel/x86_64/primary_db                                                                                                                                                                                                           | 6.6 MB  00:00:01     
        Resolving Dependencies
        --> Running transaction check
        ---> Package python2-pip.noarch 0:8.1.2-6.el7 will be installed
        --> Finished Dependency Resolution
        
        ...省略
        
        Installed:
          python2-pip.noarch 0:8.1.2-6.el7                                                                                                                                                                                                                             
    
        Complete!
5. 然后安装`shadowsocks`

        命令：
        pip install shadowsocks    
        效果：
        [root@sunhao-centos ~]# pip install shadowsocks
        Collecting shadowsocks
          Downloading https://files.pythonhosted.org/packages/02/1e/e3a5135255d06813aca6631da31768d44f63692480af3a1621818008eb4a/shadowsocks-2.8.2.tar.gz
        Installing collected packages: shadowsocks
          Running setup.py install for shadowsocks ... done
        Successfully installed shadowsocks-2.8.2

# 配置梯子
1. 新建配置文件`shadowsocks.json`

        [root@sunhao-centos ~]$ cat /etc/shadowsocks.json 
        {
            "server": "x.x.x.x",            # 梯子IP
            "server_port": 12345,           # 端口
            "local_address": "127.0.0.1",   # 本地ip
            "local_port": 1080,             # 本地端口
            "password": "password",         # 连接梯子密码
            "timeout": 300,                 # 等待超时
            "method": "rc4-md5",            # 加密方式
        }
2. 启动
        
        命令：
        nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
        效果：
        [root@sunhao-centos-vbox ~]# nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
        [2] 25356
        [root@sunhao-centos-vbox ~]# ps aux|grep sslocal
        root     25326  0.4  1.0 206984 10976 pts/1    S    00:41   0:00 /usr/bin/python2 /usr/bin/sslocal -c /etc/shadowsocks.json /dev/null
3. 设置成开机启动

        将启动命令写入到/etc/rc.d/rc.local即可
        [root@sunhao-centos ~]$ cat /etc/rc.d/rc.local
        #!/bin/bash
        # THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
        #
        # It is highly advisable to create own systemd services or udev rules
        # to run scripts during boot instead of using this file.
        #
        # In contrast to previous versions due to parallel execution during boot
        # this script will NOT be run after all other services.
        #
        # Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
        # that this script will be executed during boot.
        
        sslocal -c /etc/shadowsocks.json -d start
        
        touch /var/lock/subsys/local
4. 验证梯子是否正常运行                

        [root@sunhao-centos-vbox ~]# curl --socks5 127.0.0.1:1080 http://httpbin.org/ip
        {
          # 显示为前面配置的ip即表示成功！  
          "origin": "x.x.x.x"
        }
        
# 安装`Privoxy`
上述安装好了梯子，但它是`socks5`代理，我们在shell里执行的命令，发起的网络请求现在还不支持socks5代理，只支持http/https代理。为了我门需要安装`privoxy`代理，它能把电脑上所有http请求转发给梯子

1. 安装

        命令：
        yum install privoxy -y
2. 配置

        命令：
        vim /etc/privoxy/config
    - 先搜索关键字`listen-address`找到`listen-address 127.0.0.1:8118`这一句，保证这一句没有注释，`8118`就是将来http代理要输入的端口。
    - 然后搜索`forward-socks5t`, 将`#forward-socks5t / 127.0.0.1:9050 .`此句前面的注释去掉，然后修改为`forward-socks5t / 127.0.0.1:1080 .`，意思是转发流量到本地的`1080`端口, 而`1080`端口是你的梯子监听的端口。
3. 启动privoxy

        systemctl restart privoxy 
        systemctl enable privoxy

# 最后，配置系统代理
1. 写在全局配置`/etc/profile`中

        [root@sunhao-centos ~]$ vim /etc/profile
        # 代理http
        export http_proxy=http://127.0.0.1:8118
        # 代理https
        export https_proxy=http://127.0.0.1:8118

# 测试梯子是否可以使用
1. 命令行

        curl www.google.com.hk
2. 浏览器访问`https://www.google.com.hk`
3. 重启系统，再进行上面的操作，检查开机启动是否起作用





