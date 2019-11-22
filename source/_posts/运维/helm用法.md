title: Helm学习
tags: [Docker,DevOps,k8s]
date: 2019-11-22
categories: 运维
---

1. 安装helm

        # 官方一键安装脚本
        curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
        $ chmod 700 get_helm.sh
        $ ./get_helm.sh
    
2. 安装Tiller
    1. 先在 K8S 集群上每个节点安装 socat 软件(yum install -y socat)，否则会报错
        
            E0522 22:22:15.492436   24409 portforward.go:331] an error occurred forwarding 38398 -> 44134: error forwarding port 44134 to pod dc6da4ab99ad9c497c0cef1776b9dd18e0a612d507e2746ed63d36ef40f30174, uid : unable to do port forwarding: socat not found.
            Error: cannot connect to Tiller

    2. 国内安装使用阿里云的镜像，Google被墙
    
            helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.13.1

    3. 查看所有的仓库
    
            [root@k8s-master home]# helm repo list
            NAME     	URL                                             
            stable   	https://kubernetes-charts.storage.googleapis.com
            local    	http://127.0.0.1:8879/charts

    4. 添加harbor的charts仓库
    
            helm repo add --username helm --password Helm12345 home-helm http://home.lodsve.com:9013/chartrepo/helm

    5. 更新仓库
    
            helm repo update

    6. 给 Tiller 授权
        1. 创建service account
        
                kubectl create serviceaccount --namespace kube-system tiller
                kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
        2. 授权
        
                kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

        3. 验证是否成功
        
                [root@k8s-master home]# kubectl get deploy --namespace kube-system tiller-deploy --output yaml | grep serviceAccount
                      serviceAccount: tiller
                      serviceAccountName: tiller

    7. 卸载
    
            helm reset或者helm reset --force
        
3. 创建charts
    1. 创建名为csc-fmis-config-server的chart
    
            helm create csc-fmis-config-server
    2. 编辑各种文件：
    
            [root@k8s-master workspace]# tree csc-fmis-config-server/
            csc-fmis-config-server/
            ├── Chart.yaml                      # Chart本身的版本和配置信息
            ├── charts                          # 依赖的chart
            ├── README.md                       # readme
            ├── templates                       # 配置模板目录
            │   ├── NOTES.txt                   # helm提示信息
            │   ├── _helpers.tpl                # 用于修改kubernetes objcet配置的模板
            │   ├── deployment.yaml             # k8s的部署yaml
            │   └── service.yaml                # k8s的服务yaml
            └── values.yaml                     # 上面模板中使用到的值
    
            2 directory, 7 files
    3. 使用go的template语法
    4. 检查配置和模板是否有效
    
            helm install --dry-run --debug csc-fmis-config-server -f custom.yaml
    5. 可以使用外部yaml文件覆盖charts内部的values.yaml中的值，类似spring-boot中的application.yml一样

4. 推送到远端helm charts repo
    1. 安装helm插件，以支持helm push
    
            helm plugin install https://github.com/chartmuseum/helm-push
    2. helm push 
    
            helm push csc-fmis-config-server/ --version="0.1.2" home-helm
    3. push语法
    
            $ helm push mychart-0.1.0.tgz chartmuseum       # push .tgz from "helm package"
            $ helm push . chartmuseum                       # package and push chart directory
            $ helm push . --version="7c4d121" chartmuseum   # override version in Chart.yaml
            $ helm push . https://my.chart.repo.com         # push directly to chart repo URL
        
5. 安装到k8s中
    1. `helm install csc-fmis-config-server`
    2. `helm install csc-fmis-config-server -f custom.yaml`
    3. `helm install csc-fmis-config-server --set foo=bar --set foo=newbar`
    4. `helm install --help`查看其它用法
    3. `helm list`
