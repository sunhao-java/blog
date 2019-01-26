title: yapi 安装过程
tags: [Docker,yapi]
date: 2019-01-23
categories: 工具
---

`YApi`是`去哪儿网大前端技术中心（YMFE）`开发并开源的一款`高效、易用、功能强大的API管理平台`。支持`项目管理`、`接口管理`、`MockServer`、`文档管理`等等实用功能。

# 关于`YApi`
1. 源码：[https://github.com/YMFE/yapi](https://github.com/YMFE/yapi)
2. 使用教程：[https://yapi.ymfe.org/documents/index.html](https://yapi.ymfe.org/documents/index.html)
3. 在线Demo：[http://yapi.demo.qunar.com/](http://yapi.demo.qunar.com/)
4. 最新版本：[https://github.com/YMFE/yapi/releases](https://github.com/YMFE/yapi/releases)
5. 版本记录：[https://yapi.ymfe.org/documents/CHANGELOG.html](https://yapi.ymfe.org/documents/CHANGELOG.html)

# 创建镜像
1. 编写`Dockerfile`文件，文件太长，直接查看[Github](https://github.com/sunhao-java/docker-templates/blob/master/yapi/Dockerfile)上的文件
2. 下载好`yapi-1.4.4.tar.gz`
3. 可以直接先构建镜像，或者使用`docker-compose`来构建，推荐使用后者

# 创建`docker-compose.yml`文件
1. 文件太长了，请直接查看[Github](https://github.com/sunhao-java/docker-templates/blob/master/yapi/docker-compose.yml)上的文件
2. 可以自定义几处用户名密码，下面的`YApi`配置文件会使用到

# 创建`YApi`的配置文件
1. `mkdir config`
2. `vim config.json`

        {
          "port": "3000",
          "adminAccount": "sunhao@lodsve.com",
          "db": {
            "servername": "yapi-mongo",
            "port": 27017,
            "DATABASE": "yapi",
            "user": "admin",
            "pass": "admin123",
            "authSource": "admin"
          },
          "mail": {
            "enable": true,
            "host": "smtp.qiye.aliyun.com",
            "port": 25,
            "from": "sunhao@lodsve.com",
            "auth": {
              "user": "sunhao@lodsve.com",
              "pass": "xxxxxxxxx"
            }
          }
        }

# 启动`YApi`
1. `docker-compose up -d`

# 访问地址：
1. YApi

        url: http://your_yapi_server_ip:3000
        # 用户名在yapi的配置文件config.json中设置的，密码是默认初始密码
        username: sunhao@lodsve.com
        password: ymfe.org
2. mongo-express        
        
        url: http://your_yapi_server_ip:8081
        # 用户名密码在docker-compose.yml中设置的
        username: root
        password: root123
        
        
        
        
        
        