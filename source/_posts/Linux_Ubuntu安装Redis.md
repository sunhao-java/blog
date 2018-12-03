title: Linux_Ubuntu安装Redis
tags: [Linux,Ubuntu,Redis]
date: 2014-10-12
---
1. 下载安装
	- 将安装包上传，放入`/opt/software/redis`
	- 执行命令

			tar -zxf redis-2.8.13.tar.gz
			cd redis-2.8.13
			make
			sudo make install
			make test

<!-- more -->

2. 这时Redis 的可执行文件被放到了`/usr/local/bin`
	- 下载配置文件和init启动脚本：

			wget https://github.com/ijonas/dotfiles/raw/master/etc/init.d/redis-server
			wget https://github.com/ijonas/dotfiles/raw/master/etc/redis.conf
			sudo mv redis-server /etc/init.d/redis-server
			sudo chmod +x /etc/init.d/redis-server
			sudo mv redis.conf /etc/redis.conf

			备注:
			redis-server		--> [Linux_Ubuntu安装Redis_redis-server]
			Redis_redis.conf	--> [Linux_Ubuntu安装Redis_redis.conf]

3. 初始化用户和日志路径[必须执行]
	- 第一次启动Redis前，建议为Redis单独建立一个用户，并新建data和日志文件夹

			sudo useradd redis
			sudo mkdir -p /var/lib/redis
			sudo mkdir -p /var/log/redis
			sudo chown redis.redis /var/lib/redis
			sudo chown redis.redis /var/log/redis

4. 设置开机自动启动，关机自动关闭

		sudo update-rc.d redis-server defaults

5. 启动Redis：

		sudo /etc/init.d/redis-server start

6. 启动client客户端连接:

		root@sunhao-linux:~# redis-cli
		127.0.0.1:6379> set foo bar
		OK
		127.0.0.1:6379> get foo
		"bar"

---

#####问题解决：
1. 如果遇到make test发生错误：

		现象：
			root@sunhao-linux:/opt/redis# make test
			cd src && make test
			make[1]: 正在进入目录 `/opt/redis/src'
			You need tcl 8.5 or newer in order to run the Redis test
			make[1]: *** [test] 错误 1
			make[1]:正在离开目录 `/opt/redis/src'
			make: *** [test] 错误 2
		解决方案：
			root@sunhao-linux:/opt/redis# apt-get install tcl

---

#####PS:安装之后的信息
	=============================================================================
	jemalloc version   : 3.6.0-0-g46c0af68bd248b04df75e4f92d5fb804c3d75340
	library revision   : 1
	CC                 : gcc
	CPPFLAGS           :  -D_GNU_SOURCE -D_REENTRANT
	CFLAGS             : -std=gnu99 -Wall -pipe -g3 -O3 -funroll-loops  -fvisibility=hidden
	LDFLAGS            :
	EXTRA_LDFLAGS      :
	LIBS               :  -lpthread
	RPATH_EXTRA        :
	XSLTPROC           : false
	XSLROOT            :
	PREFIX             : /usr/local
	BINDIR             : /usr/local/bin
	INCLUDEDIR         : /usr/local/include
	LIBDIR             : /usr/local/lib
	DATADIR            : /usr/local/share
	MANDIR             : /usr/local/share/man
	srcroot            :
	abs_srcroot        : /opt/software/redis/redis-2.8.13/deps/jemalloc/
	objroot            :
	abs_objroot        : /opt/software/redis/redis-2.8.13/deps/jemalloc/
	JEMALLOC_PREFIX    : je_
	JEMALLOC_PRIVATE_NAMESPACE
	                   : je_
	install_suffix     :
	autogen            : 0
	experimental       : 1
	cc-silence         : 1
	debug              : 0
	code-coverage      : 0
	stats              : 1
	prof               : 0
	prof-libunwind     : 0
	prof-libgcc        : 0
	prof-gcc           : 0
	tcache             : 1
	fill               : 1
	utrace             : 0
	valgrind           : 0
	xmalloc            : 0
	mremap             : 0
	munmap             : 0
	dss                : 0
	lazy_lock          : 0
	tls                : 1
	=============================================================================
