title: OpenResty记录日志输出到kafka
tags: [OpenResty,埋点采集,kafka]
date: 2020-05-11
categories: 大数据
---

## 安装zookeeper（基于docker安装）
1. docker-compose.yml

```
version: '3.1'
services:
  zookeeper:
    image: zookeeper
    container_name: zookeeper
    restart: always
    network_mode: "bridge"
    ports:
      - "2181:2181"
    volumes:
      - /home/docker_volume/zookeeper/zoo.cfg:/conf/zoo.cfg
      - /home/docker_volume/zookeeper/data/:/data/
      - /home/docker_volume/zookeeper/logs/:/datalog/
```
2. `docker-compose up -d`
<!-- more -->

## 安装kafka(基于docker安装)
1. docker-compose.yml

```
version: '3.1'
services:
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    restart: always
    network_mode: "bridge"
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 0
      # 宿主机ip
      KAFKA_ZOOKEEPER_CONNECT: 192.168.137.100:2181
      # 宿主机ip
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.137.100:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
```
2. `docker-compose up -d`

## 安装OpenResty
1. 见`安装OpenResty.md`

## 安装`lua-resty-kafka`
1. 下载`lua-resty-kafka`

```
wget https://github.com/doujiang24/lua-resty-kafka/archive/master.zip
unzip lua-resty-kafka-master.zip -d /home/software/
```
2. 拷贝`lua-resty-kafka`到`openresty`

```
mkdir /home/tools/openresty/lualib/kafka
cp -rf /home/software/lua-resty-kafka-master/lib/resty /home/tools/openresty/lualib/kafka/
```

## 配置
1. nginx.conf

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    #gzip  on;
    # 配置lua依赖库地址
    lua_package_path "/home/tools/openresty/lualib/kafka/?.lua;;";

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    include conf.d/*.conf;
}
```
2. `dcc.conf`

```
server {
    listen       80;
    server_name  dcc.lodsve.com;

    location /dcc.gif {
        #伪装成gif文件
        default_type image/gif;
        #本身关闭access_log，通过subrequest记录log
        access_log off;

        log_by_lua '
            -- 引入lua所有api
            local cjson = require "cjson"
            local producer = require "resty.kafka.producer"
            -- 定义kafka broker地址，ip需要和kafka的host.name配置一致
            local broker_list = {
                { host = "192.168.137.100", port = 9092 },
            }
            -- 定义json便于日志数据整理收集
            local log_json = {}
            log_json["uri"]=ngx.var.uri
            log_json["args"]=ngx.var.args
            log_json["host"]=ngx.var.host
            log_json["request_body"]=ngx.var.request_body
            log_json["remote_addr"] = ngx.var.remote_addr
            log_json["remote_user"] = ngx.var.remote_user
            log_json["time_local"] = ngx.var.time_local
            log_json["status"] = ngx.var.status
            log_json["body_bytes_sent"] = ngx.var.body_bytes_sent
            log_json["http_referer"] = ngx.var.http_referer
            log_json["http_user_agent"] = ngx.var.http_user_agent
            log_json["http_x_forwarded_for"] = ngx.var.http_x_forwarded_for
            log_json["upstream_response_time"] = ngx.var.upstream_response_time
            log_json["request_time"] = ngx.var.request_time
            -- 转换json为字符串
            local message = cjson.encode(log_json);
            -- 定义kafka异步生产者
            local bp = producer:new(broker_list, { producer_type = "async" })
            -- 发送日志消息,send第二个参数key,用于kafka路由控制:
            -- key为nill(空)时，一段时间向同一partition写入数据
            -- 指定key，按照key的hash写入到对应的partition
            local ok, err = bp:send("dcc-topic", nil, message)

            if not ok then
                ngx.log(ngx.ERR, "kafka send err:", err)
                return
            end
        ';

        #此请求不缓存
        add_header Expires "Fri, 01 Jan 1980 00:00:00 GMT";
        add_header Pragma "no-cache";
        add_header Cache-Control "no-cache, max-age=0, must-revalidate";

        #返回一个1×1的空gif图片
        empty_gif;
    }
}

server {
    listen       80;
    server_name  www.lodsve.com;

    location / {
        root   /home/tools/openresty/nginx/html/www;
        index  index.html index.htm;
    }
}
```

## 使用
1. 使用`docker-compose`依次启动`zookeeper`、`kafka`
2. 创建`/home/tools/openresty/nginx/html/www/dcc.js`文件，模拟埋点采集中心提供出去的前端SDK包

    ```
    (function () {
        var params = {};
        //Document对象数据
        if(document) {
            params.domain = document.domain || '';
            params.url = document.URL || '';
            params.title = document.title || '';
            params.referrer = document.referrer || '';
        }
        //Window对象数据
        if(window && window.screen) {
            params.sh = window.screen.height || 0;
            params.sw = window.screen.width || 0;
            params.cd = window.screen.colorDepth || 0;
        }
        //navigator对象数据
        if(navigator) {
            params.lang = navigator.language || '';
        }

        //解析_maq配置
        if(_data) {
            for(var i in _data) {
                params[_data[i][0]] = _data[i][1];
            }
        }

        //拼接参数串
        var args = '';
        for(var i in params) {
            if(args != '') {
                args += '&';
            }
            args += i + '=' + encodeURIComponent(params[i]);
        }

        //通过Image对象请求后端脚本
        var img = new Image(1, 1);
        img.src = 'http://dcc.lodsve.com/dcc.gif?' + args;
    })();
    ```
3. 创建`/home/tools/openresty/nginx/html/www/index.html`文件，模拟业务系统

    ```
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to OpenResty!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to OpenResty!</h1>
    <p>If you see this page, the OpenResty web platform is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="https://openresty.org/">openresty.org</a>.<br/>
    Commercial support is available at
    <a href="https://openresty.com/">openresty.com</a>.</p>

    <p><em>Thank you for flying OpenResty.</em></p>
    <script type="text/javascript">
        var _data = _data || [];
        _data.push(['_setAccount', '网站标识']);

        (function() {
            var dcc = document.createElement('script'); 
            dcc.type = 'text/javascript';
            dcc.async = true;
            dcc.src = 'http://www.lodsve.com/dcc.js';
            var s = document.getElementsByTagName('script')[0]; 
            s.parentNode.insertBefore(dcc, s);
        })();
    </script>
    </body>
    </html>
    ```
4. 启动`nginx`: `/home/tools/openresty/nginx/sbin/nginx`    
5. 进入kafka容器，打开kafka，创建topi，并且监听这个topic

    ```
    1. 进入容器
        docker exec -it kafka bash
    2. 创建topic
        cd /opt/kafka/bin
        sh kafka-topics.sh --zookeeper localhost:2181 --create --topic dcc-topic --partitions 1 --replication-factor 1
    3. 新打开一个窗口，进入kafka容器同样的目录，监听这个topic
        sh kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic dcc-topic --from-beginning
    4. 在之前的窗口，kafka容器中测试发送监听消息
        sh kafka-console-producer.sh --broker-list localhost:9092 --topic dcc-topic
    5. 然后输入内容回车，即可在另一个监听窗口看到
    6. 不要关闭该窗口
    ```
6. 客户机上配置host

    ```
    192.168.137.100   dcc.lodsve.com
    192.168.137.100   www.lodsve.com
    ```
7. 访问`http://www.lodsve.com`，即可在kafka的topic监听窗口看到如下效果了：

    ```
    {"host":"dcc.lodsve.com","time_local":"11\/May\/2020:10:28:57 +0800","http_referer":"http:\/\/www.lodsve.com\/","status":"200","remote_addr":"192.168.137.1","request_time":"0.000","uri":"\/dcc.gif","args":"domain=www.lodsve.com&url=http%3A%2F%2Fwww.lodsve.com%2F&title=Welcome%20to%20OpenResty!&referrer=&sh=1080&sw=1920&cd=24&lang=zh-CN&_setAccount=%E7%BD%91%E7%AB%99%E6%A0%87%E8%AF%86","http_user_agent":"Mozilla\/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/81.0.4044.138 Safari\/537.36","body_bytes_sent":"43"}
    {"host":"dcc.lodsve.com","time_local":"11\/May\/2020:11:00:38 +0800","http_referer":"http:\/\/www.lodsve.com\/","status":"304","remote_addr":"192.168.137.1","request_time":"0.000","uri":"\/dcc.gif","args":"domain=www.lodsve.com&url=http%3A%2F%2Fwww.lodsve.com%2F&title=Welcome%20to%20OpenResty!&referrer=&sh=1080&sw=1920&cd=24&lang=zh-CN&_setAccount=%E7%BD%91%E7%AB%99%E6%A0%87%E8%AF%86","http_user_agent":"Mozilla\/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/81.0.4044.138 Safari\/537.36","body_bytes_sent":"0"}
    ```