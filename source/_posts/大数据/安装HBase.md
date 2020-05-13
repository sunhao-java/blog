title: 安装HBase
tags: [HBase]
date: 2020-05-13
categories: 大数据
---

## 下载
1. 地址：[https://mirror-hk.koddos.net/apache/hbase/\${hbase_version}/hbase-\${hbase_version}-bin.tar.gz](https://mirror-hk.koddos.net/apache/hbase/\${hbase_version}/hbase-\${hbase_version}-bin.tar.gz)
2. ${hbase_version}替换成具体的版本
3. tar -zxvf hbase-${hbase_version}-bin.tar.gz
4. cd hbase-${hbase_version}-bin/bin
5. ./start-hbase.sh

<!-- more -->
## HBase和Java的版本兼容性
| Hbase版本  |  JDK6 | JDK7  | JDK8  |
| ------------ | ------------ | ------------ | ------------ |
|  1.2 | Not Supported  | yes  | yes  |
|  1.1 |  Not Supported | yes  | Not Supported  |
|  1 | Not Supported  | yes  | Not Supported  |
|  0.98 | yes  | yes  | Not Supported  |
|  0.94 | yes  | yes  | N/A  |


## 单机部署
### HBase配置
1. 修改`conf/hbase-env.sh`文件

    ```
    # 配置JDK路径
    export JAVA_HOME=/opt/jdk1.8.0_191
    # true: 使用自带的zk来维护hbase集群
    # false: 使用外部zk来维护hbase集群
    export HBASE_MANAGES_ZK=false
    ```
2. 配置`config/hbase-site.xml`文件

    ```
    <configuration>
      <!-- hbase存放数据目录 -->
      <property>
        <name>hbase.rootdir</name>
        <value>file:///home/tools/hbase-2.2.4/data</value>
      </property>

      <!-- ZooKeeper配置开始：以下二选一 -->
      <!-- 方式1. 内部zk的方式 -->
      <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <!-- zk存储路径 -->
        <value>/data/soft/hbase-2.2.1/zookeeper</value>
      </property>
      <!-- 方式2. 外部zk的方式 -->
      <property>
        <name>hbase.zookeeper.quorum</name>
        <!-- zk服务ip:port -->
        <value>192.168.137.100:2181</value>
      </property>
      <!-- ZooKeeper配置结束 -->

      <!-- 集群模式 -->
      <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
      </property>

      <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
        <description>
          Controls whether HBase will check for stream capabilities (hflush/hsync).

          Disable this if you intend to run on LocalFileSystem, denoted by a rootdir
          with the 'file://' scheme, but be mindful of the NOTE below.

          WARNING: Setting this to false blinds you to potential data loss and
          inconsistent system state in the event of process and/or node failures. If
          HBase is complaining of an inability to use hsync or hflush it's most
          likely not a false positive.
        </description>
      </property>
    </configuration>
    ```

## 集群部署【待处理】

## 使用
    cd $HBASE_HOME/bin

1. 启动
    
        ./start-hbase.sh
2. 检查服务是否启动

        [root@docker-server conf]# jps
        126934 HMaster
        127768 Jps
        127007 HRegionServer
2. 停止
        ./stop-hbase.sh