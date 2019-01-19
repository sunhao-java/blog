title: 安装kubernetes
tags: [Docker,kubernetes]
date: 2019-01-10
categories: 运维
---
1. 安装好3台虚拟机
	1. 10.20.2.194[k8s-master]
	2. 10.20.2.44[k8s-node1]
	3. 10.20.2.37[k8s-node2]
<!-- more -->
2.  基本配置，master和各个node都需要配置
    - 修改hostname
		    vim /etc/hostname 
		    hostnamectl set-hostname k8s-master|k8s-node1|k8s-node2|...|k8s-noden
    - 关闭selinux
	        vi /etc/sysconfig/selinux
	        SELINUX=disabled
    - 直接关闭防火墙
	        systemctl stop firewalld.service #停止firewall
	        systemctl disable firewalld.service #禁止firewall开机启动
    - 永久关闭swap---避免重启服务器后kubectl服务无法启动的问题
	        在/etc/fstab 里面找的 带有swap的一行，把他注释或者删除都可以，注释安全些。
	        vim /etc/fstab
	        修改后保存退出
    - 关闭swap命令
        	swapoff -a
    - 安装docker-ce仓库
        	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    - 安装docker相关，并且版本号为17.12.0.ce，匹配使用的kubernetes 
	        yum install net-tools nfs-utils telnet vim  docker-ce-17.12.0.ce-1.el7.centos -y
	        systemctl start docker && systemctl enable docker
    - 验证
	        [root@k8s-master shell]# docker version
	        Client:
	         Version:	17.12.0-ce
	         API version:	1.35
	         Go version:	go1.9.2
	         Git commit:	c97c6d6
	         Built:	Wed Dec 27 20:10:14 2017
	         OS/Arch:	linux/amd64
	
	        Server:
	         Engine:
	          Version:	17.12.0-ce
	          API version:	1.35 (minimum version 1.12)
	          Go version:	go1.9.2
	          Git commit:	c97c6d6
	          Built:	Wed Dec 27 20:12:46 2017
	          OS/Arch:	linux/amd64
	          Experimental:	false
    - 上传所需要的kubernetes文件，到/opt下，并解压
3. master上执行（到上传的kubernetes文件解压后的文件夹第一层中）
    	cd shell && sh init.sh && sh master.sh
    - 一般来说会成功的
    - 继续执行以下命令
	        mkdir -p $HOME/.kube
	        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	        sudo chown $(id -u):$(id -g) $HOME/.kube/config
    - 记住安装完输出的一条命令：
        	kubeadm join 10.20.2.194:6443 --token xie4h6.ariunlewb0qypf5v --discovery-token-ca-cert-hash sha256:4e9c3197ccf30d47cdf70afb975b8b7d0d0a77630a26c1fcd673c49c8739e143
    - 至此，master节点安装完成！
4. 在每一个node节点上执行（到上传的kubernetes文件解压后的文件夹第一层中）
    	cd shell && sh init.sh
    - 一般来说会成功的
    - 继续执行之前在master节点得到的join命令
        	kubeadm join 10.20.2.194:6443 --token xie4h6.ariunlewb0qypf5v --discovery-token-ca-cert-hash sha256:4e9c3197ccf30d47cdf70afb975b8b7d0d0a77630a26c1fcd673c49c8739e143
    - 完事之后，到master节点上执行kubectl get nodes，即可看到已注册的node节点
	        [root@k8s-master shell]# kubectl get nodes
	        NAME         STATUS     ROLES     AGE       VERSION
	        k8s-master   Ready      master    7m        v1.11.3
	        k8s-node1    Ready      <none>    30s       v1.11.3
	        k8s-node2    Ready      <none>    4s        v1.11.3
5. 至此整个集群搭建完毕
6. 使用https访问master的32000端口，使用令牌的方式登录
    	https://10.20.2.194:32000
7. 为dashboard创建用户，并生成令牌 
    - 创建dashboard管理用户
        	kubectl create serviceaccount dashboard-admin -n kube-system
    - 绑定用户为集群管理用户
        	kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
    - 获取tocken
        	kubectl describe secret -n kube-system dashboard-admin
    - 效果如下：
	        [root@k8s-master dashboard]# kubectl create serviceaccount dashboard-admin -n kube-system
	        serviceaccount/dashboard-admin created
	        [root@k8s-master dashboard]# kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
	        clusterrolebinding.rbac.authorization.k8s.io/dashboard-cluster-admin created
	        [root@k8s-master dashboard]# kubectl describe secret -n kube-system dashboard-admin-token-l7kpn
	        Error from server (NotFound): secrets "dashboard-admin-token-l7kpn" not found
	        [root@k8s-master dashboard]# kubectl describe secret -n kube-system dashboard-admin
	        Name:         dashboard-admin-token-ltcr7
	        Namespace:    kube-system
	        Labels:       <none>
	        Annotations:  kubernetes.io/service-account.name=dashboard-admin
	                      kubernetes.io/service-account.uid=002635bb-1106-11e9-a723-000c29023ca2
	
	        Type:  kubernetes.io/service-account-token
	
	        Data
	        ====
	        ca.crt:     1025 bytes
	        namespace:  11 bytes
	        token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHRjcjciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMDAyNjM1YmItMTEwNi0xMWU5LWE3MjMtMDAwYzI5MDIzY2EyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.FzeJOWlH5ugUDqPUErLWfbW_vqq-3wuBeY77v74Bm6Wl4a1Fv_2pMJoB7mQomVfvXegmg6_ISvrrm0mJ4f2DZC8SSUTMDEDLhgnrLAdHeuLJ82ZX04Oh7Pu9uFEVhHJc2rHBWX6tqyQ-rVLyGJsxM7R7L0FntXZ25uUWujwktegRzRXcMG30tnopsEQTx1fzb_47IygXRbGoiy2p1mGpOHlElqoGRvy6le5eLl38CPpuxQJbRUWFbpLsQV1Umcy4WPy1aifD2_q-l81eSRi4D0oxM1yuPksEq3ZdaYN4SYao1zdx_8D_CQwzESZ9-w4njGVbT5lkQbL4xkN2X4pP7g
    - 使用上面的token登录即可 
8. 忘记node节点join到master节点的命令怎么办？
    - 基本命令
        	kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
        1. &lt;master-ip&gt;:&lt;master-port&gt; ，本文这里对应得是10.20.2.194:6443
        2. token，一般token两天就过期了，如果过期了你需要重新创建（查看token命令是kubeadm token list，创建token命令是kubeadm token create)
	            [root@k8s-master ~]# kubeadm token list
	            TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION   EXTRA GROUPS
	            xie4h6.ariunlewb0qypf5v   23h       2019-01-07T00:05:01+08:00   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
	            [root@k8s-master ~]# kubeadm token create
	            5w6qwh.8n0ektfrjdct3ib4
        3. --discovery-token-ca-cert-hash，通过如下命令就可以得到
	            [root@k8s-master ~]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
	            4e9c3197ccf30d47cdf70afb975b8b7d0d0a77630a26c1fcd673c49c8739e143
        4. 这样就可以组成join命令了 