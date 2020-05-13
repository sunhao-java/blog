title: Flume消费Kafka中的消息存储到Hbase中
tags: [Flume,kafka,hbase]
date: 2020-05-13
categories: 大数据
---

## 准备
1. 先安装《OpenResty记录日志输出到kafka.md》和《安装HBase.md》实际操作安装好以下中间件
    - openResty[1.15.8.3]
    - kafka[2.5.0]
    - zookeeper[3.6.0]
    - hbase[0.94.27]
2. 特别注意Flume和Hbase的版本号可能会有兼容问题    

<!-- more -->
## 配置Hbase
1. 配置hbase的环境变量，可以添加到.bash_profile中
    ```shell
    export HBASE_HOME=/home/tools/hbase-0.94.27
    export PATH=$HBASE_HOME/bin:$PATH
    ```
2. 进入hbase shell环境
    ```
    [root@docker-server tools]# hbase shell
    HBase Shell; enter 'help<RETURN>' for list of supported commands.
    Type "exit<RETURN>" to leave the HBase Shell
    Version 0.94.27, rfb434617716493eac82b55180b0bbd653beb90bf, Thu Mar 19 06:17:55 UTC 2015

    hbase(main):001:0> create 'dcc', 'dcc-column'
    0 row(s) in 1.5810 seconds

    hbase(main):002:0> 
    ```
    创建`dcc`这张表，其中列族为`dcc-column`

## 安装、配置Flume
### 下载指定版本的Flume
```
# 这里使用1.7.0版本，比较稳定，也使用的比较多
wget http://archive.apache.org/dist/flume/1.7.0/apache-flume-1.7.0-bin.tar.gz
```
### 安装flume
```
tar -zxvf apache-flume-1.7.0-bin.tar.gz -C /home/tools/
mv /home/tools/apache-flume-1.7.0-bin /home/tools/flume-1.7.0
cd /home/tools/flume-1.7.0

# 配置环境变量
vim ~/.bash_profile
## add env
export FLUME_CLASSPATH=/home/tools/flume-1.7.0/lib
```
### 配置flume
```
# 创建配置文件
vim /home/tools/flume-1.7.0/conf/flume-kafka-hbase.conf
# 内容如下
```
```
# flume-kafka-hbase.conf
# 各组件命名  
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 指定数据源  
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
# kafka地址  
a1.sources.r1.kafka.bootstrap.servers = 192.168.137.100:9092
# 组ID  
a1.sources.r1.kafka.consumer.group.id = flume-dcc-group
# topic多个逗号隔开  
a1.sources.r1.kafka.topics = dcc-topic

# 指定hbase为数据存储  
a1.sinks.k1.type = asynchbase
# 表名  
a1.sinks.k1.table = dcc
# 列族名  
a1.sinks.k1.columnFamily = dcc-column
a1.sinks.k1.serializer = org.apache.flume.sink.hbase.SimpleAsyncHbaseEventSerializer

# channel类型  
a1.channels.c1.type = memory
# channel存储的事件容量  
a1.channels.c1.capacity = 1500000
# 事务容量  
a1.channels.c1.transactionCapacity = 10000

# 绑定source和sink到channel  
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
### 运行flume
```
/home/tools/flume-1.7.0/bin/flume-ng agent --conf /home/tools/flume-1.7.0/conf/ -f /home/tools/flume-1.7.0/conf/flume-kafka-hbase.conf -n a1 -Dflume.root.logger=INFO,console
```
### F&Q
如果出现ClassNotFoundException，请检查是否配置了Flume的环境变量，或者各组件的版本是否兼容

## 查看效果
1. 刷新在文章《OpenResty记录日志输出到kafka.md》中创建的html页面
2. 埋点的日志将会被Nginx中的lua推送到kafka中
3. 而Flume订阅了kafka中的`dcc-topic`，其将会把消费到的消息原封不动的推送到Hbase中存储
4. 进入Hbase的shell界面，可以看到数据确实存储下来了：
    ```
    [root@docker-server conf]# hbase shell
    HBase Shell; enter 'help<RETURN>' for list of supported commands.
    Type "exit<RETURN>" to leave the HBase Shell
    Version 0.94.27, rfb434617716493eac82b55180b0bbd653beb90bf, Thu Mar 19 06:17:55 UTC 2015

    hbase(main):001:0> scan 'dcc'
    ROW                                                                                COLUMN+CELL                                                                                                                                                                                                                                      
    defaultc37a20c9-5a08-41b7-9999-6b25eb7daa92                                       column=dcc-column:pCol, timestamp=1589355681397, value={"host":"dcc.yeahzee.com","time_local":"13\x5C/May\x5C/2020:15:41:19 +0800","http_referer":"http:\x5C/\x5C/www.yeahzee.com\x5C/","status":"304","remote_addr":"192.168.137.1","request_tim
                                                                                    e":"0.000","uri":"\x5C/dcc.gif","args":"domain=www.yeahzee.com&url=http%3A%2F%2Fwww.yeahzee.com%2F&title=Welcome%20to%20OpenResty!&referrer=&sh=1080&sw=1920&cd=24&lang=zh-CN&_setAccount=%E7%BC%83%E6%88%A0%E7%8F%AF%E9%8F%8D%E5%9B%AA%E7%98%91"
                                                                                    ,"http_user_agent":"Mozilla\x5C/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit\x5C/537.36 (KHTML, like Gecko) Chrome\x5C/81.0.4044.138 Safari\x5C/537.36","body_bytes_sent":"0"}                                                                  
    incRow                                                                            column=dcc-column:iCol, timestamp=1589355681398, value=\x00\x00\x00\x00\x00\x00\x00\x03                                                                                                                                                          
    2 row(s) in 0.4480 seconds

    hbase(main):002:0>     
    ```
    
## PS
1. 查看Hbase中的数据可以下载[工具](https://github.com/bit-ware/HBaseXplorer "工具")
2. 下载完成后按照操作系统选择对应的启动脚本双击，在打开的窗口填入Hbase连接的zookeeper地址，点击确定即可
3. 效果
![效果图](/img/big_data/1.png)