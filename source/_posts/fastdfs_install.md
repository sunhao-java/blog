title: 安装配置FastDFS
tags: [FastDFS]
date: 2017-05-26
---
> 在新的服务器上安装了FastDFS_v5.05，相比较FastDFSV3.02还是有很多变化，现将安装配置过程记录下，供大家参考，出于安全考虑，其中涉及到IP地址的地方，随意用了一个IP202.98.27.31，在访问量不大情况下，将tracker和storage都部署在同一台服务器上，后期根据业务需要进行扩展：

1. 软件下载：

		wget https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz
		wget http://jaist.dl.sourceforge.net/project/fastdfs/FastDFS%20Nginx%20Module%20Source%20Code/fastdfs-nginx-module_v1.16.tar.gz
		wget https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz
		wget http://nginx.org/download/nginx-1.8.0.tar.gz
		wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.gz
		wget http://zlib.net/zlib-1.2.8.tar.gz

2. libfastcommon安装：

		cp V1.0.7.tar.gz /usr/local/
		tar -zxvf V1.0.7.tar.gz
		cd libfastcommon-1.0.7
		./make.sh
		./make.sh install
		
	 libfastcommon.so默认安装到了/usr/lib64/libfastcommon.so，而FastDFS主程序设置的lib目录是/usr/local/lib，所以设置软连接
	 
	 	 ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
		 ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
		 ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
		 ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
		 
3. 安装FastDFS

	- 解压安装

			tar -zxvf V5.05.tar.gz -C /usr/local
			cd /usr/local/fastdfs-5.05/
			
			./make.sh
			./make.sh install
	- 配置文件修改

			cd /etc/fdfs
			cp tracker.conf.sample tracker.conf
			cp storage.conf.sample storage.conf
			cp client.conf.sample client.conf
			
		详细设置见附件
		
		- 附件1:
		
				tracker.conf配置中要修改的几个项：
				bind_addr=202.98.27.31
				port=22122
				http.server_port=8080
		- 附件2：
		
				storage.conf配置中要修改的几个项：
				group_name=group1
				bind_addr=202.98.27.31
				port=23000
				base_path=/usrdata/fastdfs
				store_path0=/usrdata/fastdfs
				tracker_server=202.98.27.31:22122
				http.server_port=8888
	- 启动

			启动tracker storage
			fdfs_trackerd /etc/fdfs/tracker.conf
			fdfs_storaged /etc/fdfs/storage.conf
4. 安装nginx插件
	- 安装

			tar -zxvf fastdfs-nginx-module_v1.16.tar.gz
			cd fastdfs-nginx-module/src/
	- `config`文件修改

			vi config
			修改如下配置，我这里原来是
			CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
			改成
			CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
	- 修改配置

			cp  mod_fastdfs.conf /etc/fdfs
			修改配置
			group_name=group1
			tracker_server=202.98.27.31:22122
			store_path0=/usrdata/fastdfs
			base_path=/usrdata/fastdfs
			url_have_group_name = true
	- 配置文件服务器的软连接

			ln -s /usrdata/fastdfs/data /usrdata/fastdfs/data/M00  (配置文件中stoage存放数据的路径)
	- 同时将以下两个文件复制到`/etc/fdfs/`

			cp /usr/local/fastdfs-5.05/http.conf /etc/fdfs/
			cp /usr/local/fastdfs-5.05/mime.types /etc/fdfs/
5. nginx安装：

	在每个Storage服务器上安装Nginx
	- pcre安装：
	
			tar -zxvf pcre-8.36.tar.gz
			cd pcre-8.36
			./configure
			make && make install
			cd ../
			
			ln -s /usr/local/lib/libpcre.so.1 /lib64/
	- zlib安装：

			tar -zxvf zlib-1.2.8.tar.gz
			cd zlib-1.2.8
			./configure
			make && make install
			cd ../
	- nginx安装：

			tar -zxvf nginx-1.8.0.tar.gz
			cd nginx-1.8.0
			
			./configure --prefix=/usr/local/nginx --add-module=/home/yq/fastdfs-nginx-module/src
			make
			make install
	- nginx配置

			cd /usr/local/nginx/conf
			vi nginx.conf
		
		在server中添加
		
			location /group1/M00{
			    root /usrdata/fastdfs/data;
			    ngx_fastdfs_module;
			}
	- 启动：
	
			/usr/local/nginx/sbin/nginx
6. 测试文件上传：

		/usr/bin/fdfs_test /etc/fdfs/client.conf upload benz.jpg
