title: 搭建Harbor企业级docker仓库
tags: [Docker,Harbor]
date: 2019-01-18
---
# 下载harbor安装文件
1. [harbor-online-installer-v1.7.1.tgz](https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-online-installer-v1.7.1.tgz "harbor-online-installer-v1.7.1.tgz")
2. 放置在`/opt`目录下
3. 解压
4. `cd /opt/harbor`

# 修改配置文件
1. 修改配置文件`/opt/harbor/harbor.cfg`

		vim /opt/harbor/harbor.cfg

		# 主机名，后面用这个地址来访问
		hostname = harbor.lodsve.com
		# 邮箱配置
		email_server = smtp.qiye.aliyun.com
		email_server_port = 25
		email_username = harbor@lodsve.com
		email_password = XXXXXXXXXXX
		email_from = Harbor <harbor@lodsve.com>
		email_ssl = false
		email_insecure = false
		# 禁止用户注册
		self_registration = off
		# 设置只有管理员可以创建项目
		project_creation_restriction = adminonly

# 执行安装脚本
	
	/opt/harbor/install.sh

执行过程：

	[Step 0]: checking installation environment ...

	Note: docker version: 18.09.0
	
	Note: docker-compose version: 1.23.1
	
	
	[Step 1]: preparing environment ...
	Generated and saved secret to file: /data/secretkey
	Generated configuration file: ./common/config/nginx/nginx.conf
	Generated configuration file: ./common/config/adminserver/env
	Generated configuration file: ./common/config/core/env
	Generated configuration file: ./common/config/registry/config.yml
	Generated configuration file: ./common/config/db/env
	Generated configuration file: ./common/config/jobservice/env
	Generated configuration file: ./common/config/jobservice/config.yml
	Generated configuration file: ./common/config/log/logrotate.conf
	Generated configuration file: ./common/config/registryctl/env
	Generated configuration file: ./common/config/core/app.conf
	Generated certificate, key file: ./common/config/core/private_key.pem, cert file: ./common/config/registry/root.crt
	The configuration files are ready, please use docker-compose to start the service.
	
	
	[Step 2]: checking existing instance of Harbor ...
	
	
	[Step 3]: starting Harbor ...
	Creating network "harbor_harbor" with the default driver
	Pulling log (goharbor/harbor-log:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-log
	321a8da5ee1f: Pull complete
	e58cb02d4a79: Pull complete
	b1addcae27cf: Pull complete
	0add5fe71c61: Pull complete
	701d7cb4751e: Pull complete
	ae052802ba8f: Pull complete
	474572a6c946: Pull complete
	Pulling registry (goharbor/registry-photon:v2.6.2-v1.7.1)...
	v2.6.2-v1.7.1: Pulling from goharbor/registry-photon
	321a8da5ee1f: Already exists
	427e471dc5bb: Pull complete
	79d644c380a9: Pull complete
	d1ee69ba441f: Pull complete
	13ee399ae5e6: Pull complete
	52da6cf3d71f: Pull complete
	e6dfe8d3336d: Pull complete
	2261e5dd4591: Pull complete
	Pulling registryctl (goharbor/harbor-registryctl:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-registryctl
	321a8da5ee1f: Already exists
	60ab2a220157: Pull complete
	685cb36a4aa6: Pull complete
	6ab9cbb7c05b: Pull complete
	d66f51b51c32: Pull complete
	152d893b8817: Pull complete
	Pulling postgresql (goharbor/harbor-db:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-db
	321a8da5ee1f: Already exists
	3b62caa7690c: Pull complete
	0c0b8f8af809: Pull complete
	68db7c777555: Pull complete
	810390407c8c: Pull complete
	d99f5e0b551e: Pull complete
	0dedd5da1f5d: Pull complete
	5e156cfb841f: Pull complete
	0433d5b9e1ad: Pull complete
	Pulling adminserver (goharbor/harbor-adminserver:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-adminserver
	321a8da5ee1f: Already exists
	3235adc5dfba: Pull complete
	36df358268ae: Pull complete
	f07cf44733c3: Pull complete
	153223fc88f2: Pull complete
	Pulling core (goharbor/harbor-core:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-core
	321a8da5ee1f: Already exists
	95d433145bab: Pull complete
	49d3e2a9635a: Pull complete
	6a4cbc768efe: Pull complete
	7e7d30cebeb5: Pull complete
	Pulling portal (goharbor/harbor-portal:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-portal
	321a8da5ee1f: Already exists
	0c2edbea17ee: Pull complete
	35f0e6ee2803: Pull complete
	815b36cabaa4: Pull complete
	Pulling redis (goharbor/redis-photon:v1.7.1)...
	v1.7.1: Pulling from goharbor/redis-photon
	321a8da5ee1f: Already exists
	e37a237fdce1: Pull complete
	a533db83c439: Pull complete
	60f1956f70fa: Pull complete
	c7eecf8b746b: Pull complete
	Pulling jobservice (goharbor/harbor-jobservice:v1.7.1)...
	v1.7.1: Pulling from goharbor/harbor-jobservice
	321a8da5ee1f: Already exists
	4809bd624b7e: Pull complete
	889c696c8f56: Pull complete
	72d181b0302b: Pull complete
	Pulling proxy (goharbor/nginx-photon:v1.7.1)...
	v1.7.1: Pulling from goharbor/nginx-photon
	321a8da5ee1f: Already exists
	044755eb163c: Pull complete
	Digest: sha256:9ec5644c667e87bf051e581ce74b2933d3ed469b27862534ba60ccf17b4ff57a
	Status: Downloaded newer image for vmware/nginx-photon:1.11.13
	Creating harbor-log ... done
	Creating harbor-log ... done
	Creating harbor-db ... done
	Creating harbor-adminserver ... done
	Creating registry ... done
	Creating harbor-db ... done
	Creating registry ... done
	Creating registry ... done
	Creating harbor-ui ... done
	Creating harbor-ui ... done
	Creating nginx ... done
	Creating harbor-jobservice ... done
	Creating harbor-jobservice ... done
	Creating nginx ... done
	
	✔ ----Harbor has been installed and started successfully.----
	
	Now you should be able to visit the admin portal at http://harbor.lodsve.com. 
	For more details, please visit https://github.com/vmware/harbor .

# Harbor启动和停止
1. Harbor 的日常运维管理是通过docker-compose来完成的，Harbor本身有多个服务进程，都放在docker容器之中运行，我们可以通过docker ps命令查看。

		[root@localhost harbor]# docker ps
		CONTAINER ID        IMAGE                                        COMMAND                  CREATED             STATUS                 PORTS                                                                              NAMES
		3c69bd72f85d        goharbor/nginx-photon:v1.7.1                 "nginx -g 'daemon of…"   6 hours ago         Up 6 hours (healthy)   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp                   nginx
		a88dcf21cd32        goharbor/harbor-portal:v1.7.1                "nginx -g 'daemon of…"   6 hours ago         Up 6 hours (healthy)   80/tcp                                                                             harbor-portal
		92523370eeb3        goharbor/harbor-jobservice:v1.7.1            "/harbor/start.sh"       6 hours ago         Up 6 hours                                                                                                harbor-jobservice
		3963d6efe445        goharbor/harbor-core:v1.7.1                  "/harbor/start.sh"       6 hours ago         Up 6 hours (healthy)                                                                                      harbor-core
		cb96daa8cfd8        goharbor/harbor-registryctl:v1.7.1           "/harbor/start.sh"       6 hours ago         Up 6 hours (healthy)                                                                                      registryctl
		e0c704663b1b        goharbor/harbor-db:v1.7.1                    "/entrypoint.sh post…"   6 hours ago         Up 6 hours (healthy)   5432/tcp                                                                           harbor-db
		277d48e55a01        goharbor/redis-photon:v1.7.1                 "docker-entrypoint.s…"   6 hours ago         Up 6 hours             6379/tcp                                                                           redis
		a1881948c75d        goharbor/registry-photon:v2.6.2-v1.7.1       "/entrypoint.sh /etc…"   6 hours ago         Up 6 hours (healthy)   5000/tcp                                                                           registry
		1a412be89d98        goharbor/harbor-adminserver:v1.7.1           "/harbor/start.sh"       6 hours ago         Up 6 hours (healthy)                                                                                      harbor-adminserver
		5a162a3252a8        goharbor/harbor-log:v1.7.1                   "/bin/sh -c /usr/loc…"   6 hours ago         Up 6 hours (healthy)   127.0.0.1:1514->10514/tcp                                                          harbor-log
2. 或者使用docker-compose 来查看

		[root@localhost harbor]# cd /opt/harbor/
		[root@localhost harbor]# docker-compose ps
		       Name                     Command                  State                                    Ports                              
		-------------------------------------------------------------------------------------------------------------------------------------
		harbor-adminserver   /harbor/start.sh                 Up (healthy)                                                                   
		harbor-core          /harbor/start.sh                 Up (healthy)                                                                   
		harbor-db            /entrypoint.sh postgres          Up (healthy)   5432/tcp                                                        
		harbor-jobservice    /harbor/start.sh                 Up                                                                             
		harbor-log           /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp                                       
		harbor-portal        nginx -g daemon off;             Up (healthy)   80/tcp                                                          
		nginx                nginx -g daemon off;             Up (healthy)   0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp
		redis                docker-entrypoint.sh redis ...   Up             6379/tcp                                                        
		registry             /entrypoint.sh /etc/regist ...   Up (healthy)   5000/tcp                                                        
		registryctl          /harbor/start.sh                 Up (healthy)                
3. Harbor的启动和停止
		
		启动Harbor
		# docker-compose start
		停止Harbor
		# docker-comose stop
		重启Harbor
		# docker-compose restart             

# 访问测试
1. 配置hosts

		10.20.2.152     harbor.lodsve.com 
2. 访问`http://harbor.lodsve.com`                
3. 默认用户名密码：`admin`/`Harbor12345`

# 测试上传和下载镜像
1. 修改各docker client配置

		[root@localhost harbor]# vim /usr/lib/systemd/system/docker.service
		ExecStart=/usr/bin/dockerd --insecure-registry harbor.lodsve.com

		systemctl daemon-reload
		systemctl  restart docker
2. 或者创建`/etc/docker/daemon.json`文件，在文件中指定仓库地址

		[root@localhost harbor]# sudo tee /etc/docker/daemon.json <<-'EOF'
		{
		  "insecure-registries": ["harbor.lodsve.com"]
		}
		EOF
		sudo systemctl daemon-reload
		sudo systemctl restart docker
3. 创建Dockerfile

		[root@localhost harbor]# vim Dockerfile 
		FROM centos:centos7.1.1503
		ENV TZ "Asia/Shanghai"
4. 创建镜像

		[root@localhost harbor]# docker build -t harbor.lodsve.com/library/centos7.1:0.1 .
5. 把镜像push到Harbor
	
		[root@localhost harbor]# docker login harbor.lodsve.com
		[root@localhost harbor]# docker push harbor.lodsve.com/library/centos7.1:0.1

# [TODO]Harbor配置TLS证书