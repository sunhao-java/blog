title: Linux_Ubuntu_安装Nginx
tags: [Linux, Ubuntu, Nginx, frontend]
date: 2016-03-15
---

1. 下载

		sudo wget http://nginx.org/download/nginx-${version}.tar.gz

<!-- more -->

2. 解压

		tar -xzvf nginx-1.6.0.tar.gz

3. 进入相关nginx目录进行以下操作

		./configure --prefix=/opt/nginx --with-http_stub_status_module
		make
		make install
	其中：`--prefix`是制定nginx的安装目录

4. 检查是否安装成功

		cd /opt/nginx/sbin
		./nginx -t
	结果显示：

		nginx: the configuration file /opt/nginx/conf/nginx.conf syntax is ok
		nginx: configuration file /opt/nginx/conf/nginx.conf test is successful

5. 启动nginx

	`cd /opt/nginx/sbin` 目录下面 输入 `./nginx` 启动 `nginx`.<br/>
	如果不在当下目录可以通过一下命令启动：

		/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf
	`-c`制定配置文件的路径，不加nginx会自动加载默认路径的配置文件。

6. 启动、停止、重启nginx

		/opt/nginx/sbin/nginx
		/opt/nginx/sbin/nginx -s stop
		/opt/nginx/sbin/nginx -s reload

7. 常见问题解决办法
	- 缺少`pcre library`

	  错误信息：

			  ./configure: error: the HTTP rewrite module requires the PCRE library. You can either disable the module by using --without-http_rewrite_module option, or install the PCRE library into the system, or build the PCRE library statically from the source with nginx by using --with-pcre=<path> option.
	  解决方法：
		  - 下载安装pcre-8.31解决问题，解压后对pcre进行如下操作
		  `sudo wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.31.tar.gz`
		  - sudo  tar -xzvf pcre-8.31.tar.gz
		  - cd /usr/local/src/pcre-8.31
		  - ./configure
		  - make
		  - sudo make install

			运气好一次通过，运气不好，make pcre时会出错,缺少gcc-c++和libtool，也就是c++编译包
      		错误信息：

	      		libtool: compile: unrecognized option `-DHAVE_CONFIG_H'
				libtool: compile: Try `libtool --help' for more information.
				make[1]: *** [pcrecpp.lo] Error 1
				make[1]: Leaving directory `/usr/local/src//pcre-8.31'
				make: *** [all] Error 2root@wolfdog-virtual-machine:~/work/pcre-8.12$ libtool -help -DHAVE_CONFIG_H
				The program 'libtool' is currently not installed.  You can install it by typing:
				sudo apt-get install libtool
	  		解决方法：

	  			1. 需要先安装libtool和gcc-c++
				2. sudo apt-get install libtool
				3. sudo apt-get install g++ 或者 gcc-c++
	- 缺少`openssl `库

	  	错误信息：

  			./configure: error: the HTTP cache module requires md5 functions
			from OpenSSL library.  You can either disable the module by using
			--without-http-cache option, or install the OpenSSL library into the system,
			or build the OpenSSL library statically from the source with nginx by using
			--with-http_ssl_module --with-openssl=<path> options.
	  	缺少zlib库：

		  	./configure: error: the HTTP gzip module requires the zlib library.
			You can either disable the module by using --without-http_gzip_module
			option, or install the zlib library into the system, or build the zlib library
			statically from the source with nginx by using --with-zlib=<path> option.
	  	解决办法：

	  		sudo apt-get install openssl libssl-dev libperl-dev
	- 错误信息：

			./nginx -t
			./nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object file: No such file or directory
	  解决办法：`ln -s /usr/local/lib/libpcre.so.1 /lib`