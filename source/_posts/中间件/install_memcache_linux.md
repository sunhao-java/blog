title: 在Linux上安装Memcached服务
tags: [Linux, Memcached]
date: 2016-03-14
categories: 中间件
---
# 安装Memcached服务
1. 分别把memcached和libevent下载回来，放到 /tmp 目录下：

		# cd /tmp
		# wget http://www.danga.com/memcached/dist/memcached-1.2.0.tar.gz
		# wget http://www.monkey.org/~provos/libevent-1.2.tar.gz

<!-- more -->

2. 先安装libevent：

		# tar zxvf libevent-1.2.tar.gz
		# cd libevent-1.2
		# ./configure –prefix=/usr
		# make
		# make install

3. 测试libevent是否安装成功：

		# ls -al /usr/lib | grep libevent
		lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent-1.2.so.1 -> libevent-1.2.so.1.0.3
		-rwxr-xr-x 1 root root 263546 11?? 12 17:38 libevent-1.2.so.1.0.3
		-rw-r–r– 1 root root 454156 11?? 12 17:38 libevent.a
		-rwxr-xr-x 1 root root 811 11?? 12 17:38 libevent.la
		lrwxrwxrwx 1 root root 21 11?? 12 17:38 libevent.so -> libevent-1.2.so.1.0.3
		还不错，都安装上了。

4. 安装memcached，同时需要安装中指定libevent的安装位置：

		# cd /tmp
		# tar zxvf memcached-1.2.0.tar.gz
		# cd memcached-1.2.0
		# ./configure –with-libevent=/usr
		# make
		# make install
如果中间出现报错，请仔细检查错误信息，按照错误信息来配置或者增加相应的库或者路径。
安装完成后会把memcached放到 /usr/local/bin/memcached ，

5. 测试是否成功安装memcached：
6. 
		# ls -al /usr/local/bin/mem*
		-rwxr-xr-x 1 root root 137986 11?? 12 17:39 /usr/local/bin/memcached
		-rwxr-xr-x 1 root root 140179 11?? 12 17:39 /usr/local/bin/memcached-debug

# 启动Memcached服务

1. 启动Memcache的服务器端：

		# /usr/local/bin/memcached -d -m 10 -u root -l 192.168.141.64 -p 12000 -c 256 -P /tmp/memcached.pid

		-d选项是启动一个守护进程，
		-m是分配给Memcache使用的内存数量，单位是MB，我这里是10MB，
		-u是运行Memcache的用户，我这里是root，
		-l是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.0.200，
		-p是设置Memcache监听的端口，我这里设置了12000，最好是1024以上的端口，
		-c选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定，
		-P是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid，

2. 如果要结束Memcache进程，执行：

		# kill `cat /tmp/memcached.pid`
		也可以启动多个守护进程，不过端口不能重复。

# 测试Memcached

	[root@localhost /]# telnet 192.168.141.64 12000
	Trying 192.168.141.64...
	Connected to 192.168.141.64 (192.168.141.64).
	Escape character is '^]'.
	set key1 0 60 4
	zhou
	STORED
	get key1
	VALUE key1 0 4
	zhou
	END


> 至此Memcached安装成功!

# 常见问题:

1. 如果启动Memcached服务的时候遇到了

		/usr/local/bin/memcached: error while loading shared libraries: libevent-1.2.so.1: cannot open shared object file: No such file or directory;

	> 解决方案:

		[root@localhost bin]# LD_DEBUG=libs memcached -v 
		[root@localhost bin]# ln -s /usr/lib/libevent-1.2.so.1 /usr/lib64/libevent-1.2.so.1
		[root@localhost bin]# /usr/local/bin/memcached -d -m 100 -u root -p 12000 -c 1000 -P /tmp/memcached.pid
		[root@localhost bin]# ps -aux
		可以看到启动的Memcached服务了.

 

2. 把Memcached服务加载到Linux的启动项中.万一机器断电系统重启.那么Memcached就会自动启动了.

		假如启动Memcache的服务器端的命令为：
		# /usr/local/bin/memcached -d -m 10 -u root -l 192.168.141.64 -p 12000 -c 256 -P /tmp/memcached.pid容来自17jquery

	> 想开机自动启动的话，只需在/etc/rc.d/rc.local中加入一行，下面命令
	/usr/local/memcached/bin/memcached -d -m 10 -p 12000 -u apache -c 256 
	上面有些东西可以参考一下：即，ip不指定时，默认是本机，用户:最好选择是：apache 或 deamon
	这样，也就是属于哪个用户的服务，由哪个用户启动。

# memcached之java客户端: spymemcached与spring整合

---------
> net.spy.memcached.spring.MemcachedClientFactoryBean在net.spy.memcached.MemcachedClient每次使用的时候创建MemcachedClient的新实例。

	<bean id="memcachedClient" class="net.spy.memcached.spring.MemcachedClientFactoryBean">  
	    <property name="servers" value="host1:11211,host2:11211,host3:11211"/>  
	    <property name="protocol" value="BINARY"/>  
	    <property name="transcoder">  
	      <bean class="net.spy.memcached.transcoders.SerializingTranscoder">  
	        <property name="compressionThreshold" value="1024"/>  
	      </bean>  
	    </property>  
	    <property name="opTimeout" value="1000"/>  
	    <property name="timeoutExceptionThreshold" value="1998"/>  
	    <property name="hashAlg" value="KETAMA_HASH"/>  
	    <property name="locatorType" value="CONSISTENT"/>   
	    <property name="failureMode" value="Redistribute"/>  
	    <property name="useNagleAlgorithm" value="false"/>  
	</bean>  

> 属性说明：

1. Servers
一个字符串，包括由空格或逗号分隔的主机或IP地址与端口号
2. Daemon
设置IO线程的守护进程(默认为true)状态
3. FailureMode
设置故障模式(取消，重新分配，重试)，默认是重新分配
4. HashAlg
设置哈希算法(见net.spy.memcached.HashAlgorithm的值)
5. InitialObservers
设置初始连接的观察者(观察初始连接)
6. LocatorType
设置定位器类型(ARRAY_MOD,CONSISTENT),默认是ARRAY_MOD
7. MaxReconnectDelay
设置最大的连接延迟
8. OpFact
设置操作工厂
9. OpQueueFactory
设置操作队列工厂
10. OpTimeout
以毫秒为单位设置默认的操作超时时间
11. Protocol
指定要使用的协议(BINARY,TEXT),默认是TEXT
12. ReadBufferSize
设置读取的缓冲区大小
13. ReadOpQueueFactory
设置读队列工厂
14. ShouldOptimize
如果默认操作优化是不可取的，设置为false(默认为true)
15. Transcoder
设置默认的转码器(默认以net.spy.memcached.transcoders.SerializingTranscoder)
16. UseNagleAlgorithm
如果你想使用Nagle算法，设置为true
17. WriteOpQueueFactory
设置写队列工厂
18. AuthDescriptor
设置authDescriptor,在新的连接上使用身份验证