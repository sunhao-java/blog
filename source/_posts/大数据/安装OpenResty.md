title: 安装OpenResty
tags: [OpenResty,埋点采集]
date: 2020-05-11
categories: 大数据
---

## 下载
1. 打开url：[https://openresty.org/cn/download.html](https://openresty.org/cn/download.html)
2. 找到最新版本或者合适的版本
```
curl https://openresty.org/download/openresty-${version}.tar.gz
tar -zxvf openresty-${version}.tar.gz
cd openresty-${version}
```
<!-- more -->

## 编译、安装
1. 安装前的准备：安装基础库
```
yum install -y pcre-devel openssl-devel gcc curl
```
2. `./configure`
```
# prefix: 指定安装路径
./configure --prefix=/opt/openresty \
    # lua环境
    --with-luajit \
    --with-http_iconv_module \
    --with-http_postgres_module
```
3. 安装
```
make && make install
```

## 使用
1. 启动nginx
```
cd openresty-${version}/nginx/sbin
./nginx
```
2. 重新加载nginx配置
```
./nginx -s reload
```
3. 停止nginx
```
./nginx -s stop
```
4. 重启nginx
```
./nginx -s restart
```

## Q&A
1. 执行`./configure`时，可能会出现以下错误
    - 错误: `./configure: error: ngx_postgres addon was unable to detect version of the libpq library.`
    - 解决: `yum install -y postgresql-devel`