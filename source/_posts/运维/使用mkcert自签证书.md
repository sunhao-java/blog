title: 使用mkcert自签证书
tags: [DevOPS, 证书]
date: 2025-06-11

categories: 运维
---

> 假设需要为域名（www.baidu.com）或者 IP （192.168.1.100）生成自签证书。

<!-- more -->

# 准备工作（安装mkcert）

## 服务器可联网

1. Windows
    ```bash
    # Chocolatey
    choco install mkcert
    # Scoop
    scoop bucket add extras
    scoop install mkcert
    ```

2. Linux（CentOS/Ubuntu等）
    ```bash
    # yum
    sudo yum install nss-tools
    # apt
    sudo apt install libnss3-tools
    ```
    
3. MacOS
    ```bash
    # homebrew
    sudo brew install mkcert
    sudo brew install nss
    # macports
    sudo port selfupdate
    sudo port install mkcert
    sudo port install nss
    ```

## 服务器不可联网
> 如果服务器不可以联网，先在可联网电脑下载相关程序，并上传到服务器中。

1. Windows
    1. https://dl.filippo.io/mkcert/v1.4.4?for=windows/arm64
    2. https://dl.filippo.io/mkcert/v1.4.4?for=windows/amd64

2. Linux（CentOS/Ubuntu等）
    1. https://dl.filippo.io/mkcert/v1.4.4?for=linux/arm64
    2. https://dl.filippo.io/mkcert/v1.4.4?for=linux/amd64

3. MacOS
    1. https://dl.filippo.io/mkcert/v1.4.4?for=darwin/arm64
    2. https://dl.filippo.io/mkcert/v1.4.4?for=darwin/amd64

4. 安装
    1. Windows：将下载文件添加到环境变量PATH中即可
    2. MacOS/Linux：
        ```bash
        chmod u+x mkcert-v*-linux-amd64
        cp mkcert-v*-linux-amd64 /usr/local/bin/mkcert
        ```

# 生成证书

> 通过以上步骤可正常安装mkcert，接下来需要使用mkcert生成证书

1. 初始化配置
    ```bash
    mkcert -install
    ```
    
2. 为域名生成证书
    ```bash
    # 多个域名
    mkcert www.baidu.com www1.baidu.com 
    mkcert *.baidu.com
    # 多个IP
    mkcert 192.168.1.100 192.168.1.101
    ```

3. 执行第二步操作后，会在当前目录下生成以下文件，如下：
    ```bash
    -bash-4.2# ll
    -rw------- 1 root root 1704 Jun  9 13:45 www.baidu.com-key.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 www.baidu.com.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 www1.baidu.com-key.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 www1.baidu.com.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 *.baidu.com-key.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 *.baidu.com.pem
    -rw------- 1 root root 1704 Jun  9 13:45 192.168.1.100-key.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 192.168.1.100.pem
    -rw------- 1 root root 1704 Jun  9 13:45 192.168.1.101-key.pem
    -rw-r--r-- 1 root root 1493 Jun  9 13:45 192.168.1.101.pem
    ```

4. 以上文件将在nginx中配置，如下：
    ```nginx
    server {
        listen 443 ssl;
        server_name www.baidu.com;
        ssl_certificate /root/ssl/www.baidu.com.pem;
        ssl_certificate_key /root/ssl/www.baidu.com-key.pem;
        ...
    }
    ```

5. 查看自签证书的根证书，将在【验证】环节使用
    ```bash
    -bash-4.2# mkcert -CAROOT
    /root/.local/share/mkcert
    -bash-4.2# cd /root/.local/share/mkcert
    -bash-4.2# ll
    total 8
    -r-------- 1 root root 2484 Jun  9 13:37 rootCA-key.pem
    -rw-r--r-- 1 root root 1651 Jun  9 13:37 rootCA.pem
    -bash-4.2#
    ```
    其中rootCA.pem即为根证书，留作后用

# Nginx配置
1. 配置
    ```nginx
    server {
        listen 443 ssl;
        server_name www.teamhelper.cn;
    
        ssl_certificate /path/to/your/ssl/www.teamhelper.cn.pem;
        ssl_certificate_key /path/to/your/ssl/www.teamhelper.cn-key.pem;
    
        # 其他配置
    }
    ```
2. 验证、重启
    ```bash
    # 验证
    nginx -t
    # 重启
    nginx -s reload
    ```

# 验证
1. 浏览器直接访问该域名，会出现以下错误：

   ![image-20250905111652117](https://imgs.lodsve.com:9000/images/2025/09/05/48126498f3e5.png)

   并且在地址栏会提示域名证书错误：

   ![image-20250905111713759](https://imgs.lodsve.com:9000/images/2025/09/05/0d185534e6a1.png)

2. 点击地址栏红色的不安全，检查证书是否被替换成上面的自签证书

    ![image-20250905111739571](https://imgs.lodsve.com:9000/images/2025/09/05/74db92dc8615.png)

3. 两种处理证书错误的方案：
    - 安装根证书（但是需要在**每一台**访问的**客户端**都需要进行操作）
        - 将前面在服务器上执行`mkcert -CAROOT`得到的根证书`rootCA.pem`分发给每一个用户
        
        - Windows上，需要将该证书重命名为`rootCA.crt`，然后双击或者右键选择安装证书
        
          ![image-20250905111837315](https://imgs.lodsve.com:9000/images/2025/09/05/50594506ccac.png)
        
        - Mac上参考https://blog.csdn.net/qq_43004620/article/details/137969884进行配置
        
    - 仅忽略异常

        ![image-20250905111901107](https://imgs.lodsve.com:9000/images/2025/09/05/d7453e318f92.png)

4. 两种方案的弊端
    1. 证书有过期时间，默认是1年，也可设置成别的时间，过期后，需要重新安装，且每一个客户端都需要进行操作，但是浏览器地址栏不会再显示**不安全**的提示
    2. 只需要客户端在页面警告时，点击继续前往即可，不存在证书过期再重新安装的问题，但是浏览器地址栏会一直显示**不安全**的提示。    

5. 如果点击页面的高级后无继续前往...按钮，则需要进行以下步骤：

   ![image-20250905111901107](https://imgs.lodsve.com:9000/images/2025/09/05/d7453e318f92.png)

   浏览器地址栏输入以下地址，访问

   ![image-20250905112027474](https://imgs.lodsve.com:9000/images/2025/09/05/052864d8afbc.png)

   输入自签证书的域名，然后点击Delete按钮，即可看到**继续前往...**按钮

# 如何让Java程序信任该自签证书
解决自签证书，在Java程序中不被信任的临时解决方案：
1. 以root用户进入容器：
    ```bash
    docker exec -u root -it container_name bash
    ```
2. 导出服务端证书
    ```bash
    # xxx.xxx.xxx.xxx:8080：使用自签证书的域名及端口
    echo | openssl s_client -connect xxx.xxx.xxx.xxx:8080 2>/dev/null | openssl x509 > server.crt
    ```
3. 导入JVM信任库（默认密码：changeit）
    ```bash
    keytool -import -alias server-cert -file server.crt -keystore $JAVA_HOME/lib/security/cacerts
    ```
4. 重启容器（注意：这里不能使用docker-compose up -d命令，这会删除容器，重新创建容器）
    ```bash
    docker restart mengniu-teamhelper-operation-platform
    ```
5. 每次更新完成后（recreate容器后），都需要执行上述操作   