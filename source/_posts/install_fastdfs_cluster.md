title: FastDFS之集群部署
tags: [FastDFS]
date: 2017-06-05 15:24
---

## 环境准备
1. tracker：192.168.20.116
2. storage：
	- group1：192.168.20.117
	- group2：192.168.20.118
3. 准备好安装文件
	- fastdfs-5.05.zip
	- fastdfs-nginx-module.zip
	- libfastcommon-1.0.7.zip
	- nginx-1.8.1.tar.gz
	- ngx_cache_purge-2.3.tar.gz
	- pcre-8.36.zip
	- zlib-1.2.8.tar.gz

<!-- more -->

## <span id="install_tracker">安装tracker</span>
1. 安装依赖libfastcommon

		unzip libfastcommon-1.0.7.zip
		cd libfastcommon-1.0.7
		./make.sh 
		./make.sh  install
2. 安装FastDFS

		unzip fastdfs-5.05.zip
		cd fastdfs-5.05
		./make.sh 
		./make.sh install
		
	默认安装目录：`/usr/bin`
	
	配置文件在：`/etc/fdfs`，需要重命名吧`.sample`去掉
3. 配置

	编辑配置文件目录下的`tracker.conf`，一般只需改动以下几个参数即可：
	
		disabled=false            #启用配置文件
		port=22122                #设置tracker的端口号
		base_path=/opt/fastdfs   #设置tracker的数据文件和日志目录（需预先创建）
		http.server_port=8080     #设置http端口号
4. 创建数据存储文件夹 

		mkdir /opt/fastdfs
5. 启动

		fdfs_trackerd /etc/fdfs/tracker.conf
6. 安装tracker代理nginx

	在tracker上安装的nginx主要为了提供http访问的反向代理、负载均衡以及缓存服务。
	
	1. 安装nginx（具体参考[安装Nginx](http://www.lodsve.com/2016/03/15/linux_ubuntu_install_nginx/)）
		
			tar -zxvf nginx-1.8.1.tar.gz 
			tar -zxvf ngx_cache_purge-2.3.tar.gz 
			cd nginx-1.8.1
			./configure --prefix=/opt/nginx --add-module=/opt/software/ngx_cache_purge-2.3 --without-http_gzip_module
			make 
			make install
			
		如果提示错误，可能缺少依赖的软件包，需先安装依赖包，再次运行./configure。nginx以及nginx cache purge插件模块安装完成，安装目录/opt/nginx。
	2. 配置nginx

			#user  nobody;
			worker_processes  1;
			
			error_log  logs/error.log  info;
			
			events {
			    worker_connections  1024;
			}
			
			http {
			    include       mime.types;
			    default_type  application/octet-stream;
			
			    #设置缓存参数
			    server_names_hash_bucket_size 128;
			    client_header_buffer_size 32k;
			    large_client_header_buffers 4 32k;
			    client_max_body_size 300m;
			    sendfile        on;
			    tcp_nopush     on;
			    proxy_redirect off;
			    proxy_set_header Host $http_host;
			    proxy_set_header X-Real-IP $remote_addr;
			    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			    proxy_connect_timeout 90;
			    proxy_send_timeout 90;
			    proxy_read_timeout 90;
			    proxy_buffer_size 16k;
			    proxy_buffers 4 64k;
			    proxy_busy_buffers_size 128k;
			    proxy_temp_file_write_size 128k;
			    # 设置缓存存储路径、存储方式、分配内存大小、磁盘最大空间、缓存期限
			    proxy_cache_path /var/cache/nginx/proxy_cache levels=1:2 keys_zone=http-cache:500m max_size=10g inactive=30d;
			
			    proxy_temp_path /var/cache/nginx/proxy_cache/tmp;
			
			    keepalive_timeout  65;
			
			    #设置group服务器
			    upstream fdfs_group1 {
			        server 192.168.20.117:8090 weight=1 max_fails=2 fail_timeout=30s;
			    }
			
			    upstream fdfs_group2 {
			        server 192.168.20.118:8090 weight=1 max_fails=2 fail_timeout=30s;
			    }
			
			    server {
			        listen       80;
			        server_name  localhost;
			        charset utf-8;
			
			        location /group1/M00 {
			                proxy_next_upstream http_502 http_504 error timeout invalid_header;
			                proxy_cache http-cache;
			                proxy_cache_valid  200 304 12h;
			                proxy_cache_key $uri$is_args$args;
			                proxy_pass http://fdfs_group1;
			                expires 30d;
			        }
			        location /group2/M00 {
			                proxy_next_upstream http_502 http_504 error timeout invalid_header;
			                proxy_cache http-cache;
			                proxy_cache_valid  200 304 12h;
			                proxy_cache_key $uri$is_args$args;
			                proxy_pass http://fdfs_group2;
			                expires 30d;
			        }
			
			        #设置清除缓存的访问权限
			        location ~ /purge(/.*) {
			                allow 127.0.0.1;
			                allow 192.168.72.0/24;
			                deny all;
			                proxy_cache_purge http-cache  $1$is_args$args;
			        }
			    }
			
			}
		创建缓存目录：/var/cache/nginx/proxy_cache/tmp
	3. 启动nginx

			/opt/nginx/sbin/nginx
			
## 安装storage
1. 安装

	参考[安装tracker](#install_tracker)前2步骤。

2. 配置

	编辑配置文件目录下的storage.conf，只需改动以下几个参数即可：
	
		disabled=false     #启用配置文件
		group_name=group1 #组名，根据实际情况修改
		port=23000 #设置storage的端口号
		base_path=/opt/fastdfs #设置storage的日志目录（需预先创建）
		store_path_count=1 #存储路径个数，需要和store_path个数匹配
		store_path0=/opt/fastdfs#存储路径
		tracker_server=192.168.20.116:22122#tracker服务器的IP地址和端口号
		http.server_port=8080   #设置http端口号
3. 创建存储目录

		mkdir /opt/fastdfs
4. 运行

		fdfs_storaged /etc/fdfs/storage.conf
5. 按照同样的配置在`192.168.20.118`上创建`storage2`，做相应的调整（ip等）

	另外每个group中所有storage的端口号必须一致。
	
6. 在storage上安装nginx

	在storage上安装的nginx主要为了提供http的访问服务，同时解决group中storage服务器的同步延迟问题。
	
	1. 解压`fastdfs-nginx-module.zip`插件

			unzip  fastdfs-nginx-module.zip
	2. 安装nginx（具体参考[安装Nginx](http://blog.lodsve.com/2016/03/15/linux_ubuntu_install_nginx/)）

			tar -zxvf nginx-1.8.1.tar.gz 
			cd nginx-1.8.1
			./configure --prefix=/usr/local/nginx --add-module=../fastdfs-nginx-module/src --without-http_gzip_module
			make 
			make install
	3. 配置
		- 配置FastDFS的nginx插件
			
			将FastDFS的nginx插件模块的配置文件copy到FastDFS配置文件目录
			
				cp /opt/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
			
			编辑/etc/fdfs配置文件目录下的mod_fastdfs.conf，设置storage信息并保存。
一般只需改动以下几个参数即可：
				
				base_path=/opt/fastdfs #保存日志目录
				tracker_server=192.168.20.116:22122 #tracker服务器的IP地址以及端口号
				storage_server_port=23000	 #storage服务器的端口号
				group_name=group1  #当前服务器的group名
				url_have_group_name = true        #文件url中是否有group名
				store_path_count=1                #存储路径个数，需要和store_path个数匹配
				store_path0=/opt/fastdfs         #存储路径
				
			在末尾增加2个组的具体信息：
			
				[group1]
				group_name=group1
				storage_server_port=23000
				store_path_count=1
				store_path0=/opt/fastdfs
				
				[group2]
				group_name=group2
				storage_server_port=23000
				store_path_count=1
				store_path0=/opt/fastdfs
		- 建立M00至存储目录的符号连接：

				ln -s /opt/fastdfs/data /opt/fastdfs/data/M00
		- 配置nginx

				vi nginx.conf
				
				# 在server中添加如下location
				location ~/group[1-3]/M00 {
				    root /home/fastdfs/data;
				    ngx_fastdfs_module;
				}
		- 启动
				
				/usr/local/nginx/sbin/nginx
				
## 测试
配置/etc/fdfs/client.conf

	base_path=/opt/fastdfs #日志存放路径
	tracker_server=192.168.20.116:22122 #tracker服务器IP地址和端口号
	http.tracker_server_port=8080   		#tracker服务器的http端口号
	
通过fdfs_upload_file上传一个文件到FastDFS，程序会自动返回文件的URL

	[root@localhost /opt/]#fdfs_upload_file /etc/fdfs/client.conf test 
	group1/M00/00/00/wKgUdVk1drmARcxsAAAOGgU8nEQ0713851
	
然后使用浏览器访问，访问正常`http://192.168.20.116/group1/M00/00/00/wKgUdVk1drmARcxsAAAOGgU8nEQ0713851`

## 监控
可以使用fdfs_monitor查看tracker和所有group的运行情况

	[root@localhost /opt/nginx/conf]#fdfs_monitor /etc/fdfs/client.conf   
	[2017-06-05 23:21:50] DEBUG - base_path=/opt/fastdfs, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0
	
	server_count=1, server_index=0
	
	tracker server is 192.168.20.116:22122
	
	group count: 2
	
	Group 1:
	group name = group1
	disk total space = 35660 MB
	disk free space = 29390 MB
	trunk free space = 0 MB
	storage server count = 1
	active server count = 1
	storage server port = 23000
	storage HTTP port = 8080
	store path count = 1
	subdir count per path = 256
	current write server index = 0
	current trunk file id = 0
	
	        Storage 1:
	                id = 192.168.20.117
	                ip_addr = 192.168.20.117  ACTIVE
	                http domain = 
	                version = 5.05
	                join time = 2017-06-05 19:59:44
	                up time = 2017-06-05 19:59:44
	                total storage = 35660 MB
	                free storage = 29390 MB
	                upload priority = 10
	                store_path_count = 1
	                subdir_count_per_path = 256
	                storage_port = 23000
	                storage_http_port = 8080
	                current_write_path = 0
	                source storage id = 
	                if_trunk_server = 0
	                connection.alloc_count = 256
	                connection.current_count = 0
	                connection.max_count = 1
	                total_upload_count = 3
	                success_upload_count = 3
	                total_append_count = 0
	                success_append_count = 0
	                total_modify_count = 0
	                success_modify_count = 0
	                total_truncate_count = 0
	                success_truncate_count = 0
	                total_set_meta_count = 0
	                success_set_meta_count = 0
	                total_delete_count = 0
	                success_delete_count = 0
	                total_download_count = 0
	                success_download_count = 0
	                total_get_meta_count = 0
	                success_get_meta_count = 0
	                total_create_link_count = 0
	                success_create_link_count = 0
	                total_delete_link_count = 0
	                success_delete_link_count = 0
	                total_upload_bytes = 7429
	                success_upload_bytes = 7429
	                total_append_bytes = 0
	                success_append_bytes = 0
	                total_modify_bytes = 0
	                success_modify_bytes = 0
	                stotal_download_bytes = 0
	                success_download_bytes = 0
	                total_sync_in_bytes = 0
	                success_sync_in_bytes = 0
	                total_sync_out_bytes = 0
	                success_sync_out_bytes = 0
	                total_file_open_count = 3
	                success_file_open_count = 3
	                total_file_read_count = 0
	                success_file_read_count = 0
	                total_file_write_count = 3
	                success_file_write_count = 3
	                last_heart_beat_time = 2017-06-05 23:21:47
	                last_source_update = 2017-06-05 23:20:24
	                last_sync_update = 1970-01-01 08:00:00
	                last_synced_timestamp = 1970-01-01 08:00:00 
	
	Group 2:
	group name = group2
	disk total space = 35660 MB
	disk free space = 29390 MB
	trunk free space = 0 MB
	storage server count = 1
	active server count = 1
	storage server port = 23000
	storage HTTP port = 8080
	store path count = 1
	subdir count per path = 256
	current write server index = 0
	current trunk file id = 0
	
	        Storage 1:
	                id = 192.168.20.118
	                ip_addr = 192.168.20.118  ACTIVE
	                http domain = 
	                version = 5.05
	                join time = 2017-06-05 20:05:35
	                up time = 2017-06-05 20:05:35
	                total storage = 35660 MB
	                free storage = 29390 MB
	                upload priority = 10
	                store_path_count = 1
	                subdir_count_per_path = 256
	                storage_port = 23000
	                storage_http_port = 8080
	                current_write_path = 0
	                source storage id = 
	                if_trunk_server = 0
	                connection.alloc_count = 256
	                connection.current_count = 0
	                connection.max_count = 1
	                total_upload_count = 1
	                success_upload_count = 1
	                total_append_count = 0
	                success_append_count = 0
	                total_modify_count = 0
	                success_modify_count = 0
	                total_truncate_count = 0
	                success_truncate_count = 0
	                total_set_meta_count = 0
	                success_set_meta_count = 0
	                total_delete_count = 0
	                success_delete_count = 0
	                total_download_count = 0
	                success_download_count = 0
	                total_get_meta_count = 0
	                success_get_meta_count = 0
	                total_create_link_count = 0
	                success_create_link_count = 0
	                total_delete_link_count = 0
	                success_delete_link_count = 0
	                total_upload_bytes = 833473
	                success_upload_bytes = 833473
	                total_append_bytes = 0
	                success_append_bytes = 0
	                total_modify_bytes = 0
	                success_modify_bytes = 0
	                stotal_download_bytes = 0
	                success_download_bytes = 0
	                total_sync_in_bytes = 0
	                success_sync_in_bytes = 0
	                total_sync_out_bytes = 0
	                success_sync_out_bytes = 0
	                total_file_open_count = 1
	                success_file_open_count = 1
	                total_file_read_count = 0
	                success_file_read_count = 0
	                total_file_write_count = 4
	                success_file_write_count = 4
	                last_heart_beat_time = 2017-06-05 23:21:37
	                last_source_update = 2017-06-05 22:26:05
	                last_sync_update = 1970-01-01 08:00:00
	                last_synced_timestamp = 1970-01-01 08:00:00
