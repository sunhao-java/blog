title: Redis高可用集群搭建
tags: [Redis, database]
date: 2017-05-24
categories: 中间件
---
> 最近公司项目要求Redis集群且高可用，在查询了一系列文章，再结合项目实际情况，所以采用了这一套高可用集群方案

<!-- more -->

## 方案选型
Redis集群有三种方案实现：

1. Redis官方集群方案 Redis Cluster

	这种方案是redis官方提供的，采用slot的概念，一共分成16384个槽。对key的值进行散列，分配到这16384个slot中的某一个中。但是这个Redis集群，要保证16384个槽对应的node都正常工作，如果某个node发生故障，那它负责的slots也就失效，整个集群将不能工作。这不符合我们的要求：**高可用**。
	
2. 利用代理中间件实现大规模Redis集群

	过于复杂，并且我们redis的使用量不是很大，只需要保证稳定即可（高可用）。所以这个方案也没有采纳。
	
	
3. Redis Sentinel主从高可用方案

	部署redis主从服务（1个master，多个salve），然后通过redis官方的监控工具Sentinel（哨兵），对每个节点进行监控，实现自动故障迁移，即master死掉，将salve升级为master。基本原理是：心跳机制+投票裁决。

## 环境准备
1. 下载redis源码包。[地址](http://download.redis.io/releases/redis-3.2.9.tar.gz)
2. 准备1台服务器（按照部署情况，共有5个端口）
3. 5个端口中，1个master(6379)，2个slave(6380,6381)，2个sentinel（哨兵）(63791, 63792)

## sentinel配置(参数配置查看附录1)
1. 配置文件_63791(`sentinel_63791.conf`)

		# 端口
		port 63791
		daemonize yes
		# 很重要，见附录2
		protected-mode no
		# 日志文件
		logfile "/opt/redis-cluster/logs/sentinel_63791.log"
		#master-1
		# 监控master
		# sentinel monitor master_name ip port quorum
		# quorum是一个数字，指明当有多少个sentinel认为一个master失效时，master才算真正失效。
		sentinel monitor master-1 10.211.55.10 6381 1
		# 这个配置项指定了需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的。 单位是毫秒，默认为30秒
		sentinel down-after-milliseconds master-1 5000
		sentinel failover-timeout master-1 18000
		# 设置连接master和slave时的密码,master和slave用户名密码要一致
		sentinel auth-pass master-1 test123
		sentinel parallel-syncs master-1 1
2. 配置文件_63792(`sentinel_63792.conf`)

		port 63792
		daemonize yes
		protected-mode no
		logfile "/opt/redis-cluster/logs/sentinel_63792.log"
		#master-1
		sentinel monitor master-1 10.211.55.10 6381 1
		sentinel down-after-milliseconds master-1 5000
		sentinel failover-timeout master-1 18000
		sentinel auth-pass master-1 test123
		sentinel parallel-syncs master-1 1
		
## Master节点配置
`redis_master_6379.conf` 配置：

	port 6379
	daemonize yes
	requirepass test123
	masterauth test123
	
## 两个slave节点配置
1. `redis_slave_6380.conf`

		port 6380
		daemonize yes
		requirepass "test123"
		slaveof 10.211.55.10 6379
		masterauth "sunhao123"
		bind 0.0.0.0
2. `redis_slave_6381.conf` 

		port 6381
		daemonize yes
		requirepass "test123"
		slaveof 10.211.55.10 6379
		masterauth "sunhao123"
		bind 0.0.0.0
		
## 启动服务
按如下顺序依次启动服务：

	redis-server /opt/redis-cluster/redis_master_6379.conf 
	redis-server /opt/redis-cluster/redis_slave_6380.conf 
	redis-server /opt/redis-cluster/redis_slave_6381.conf 
	redis-sentinel /opt/redis-cluster/sentinel_63791.conf 
	redis-sentinel /opt/redis-cluster/sentinel_63792.conf 
	
## 查看各个节点的状态
1. 查看全部节点状态

        [root@CentOS-3 redis-cluster]# ps aux | grep redis
        root      1782  0.0  0.0 103252   824 pts/4    S+   21:36   0:00 grep redis
        root     25331  0.1  0.3 133528  7648 ?        Ssl  18:06   0:20 redis-server 0.0.0.0:6380                            
        root     25336  0.1  0.5 135576  9720 ?        Ssl  18:06   0:21 redis-server 0.0.0.0:6381                            
        root     29303  0.2  0.4 133524  7700 ?        Ssl  18:27   0:32 redis-sentinel *:63791 [sentinel]                    
        root     29308  0.2  0.4 133524  7704 ?        Ssl  18:27   0:33 redis-sentinel *:63792 [sentinel]                    
        root     29486  0.1  0.3 133528  7648 ?        Ssl  18:28   0:18 redis-server 0.0.0.0:6379
        
2. 查看master状态

		[root@CentOS-3 redis-cluster]# redis-cli -h 10.211.55.10 -p 6379
		10.211.55.10:6379> auth test123
		OK
		10.211.55.10:6379> info replication
		# Replication
		role:master
		connected_slaves:2
		slave0:ip=10.211.55.10,port=6380,state=online,offset=1568095,lag=1
		slave1:ip=10.211.55.10,port=6381,state=online,offset=1568095,lag=1
		master_repl_offset:1568095
		repl_backlog_active:1
		repl_backlog_size:1048576
		repl_backlog_first_byte_offset:519520
		repl_backlog_histlen:1048576
		10.211.55.10:6379> 
		
3. 查看slave的状态：

		[root@CentOS-3 redis-cluster]# redis-cli -h 10.211.55.10 -p 6380
		10.211.55.10:6380> auth test123
		OK
		10.211.55.10:6380> info replication
		# Replication
		role:slave
		master_host:10.211.55.10
		master_port:6379
		master_link_status:up
		master_last_io_seconds_ago:1
		master_sync_in_progress:0
		slave_repl_offset:1578354
		slave_priority:100
		slave_read_only:1
		connected_slaves:0
		master_repl_offset:0
		repl_backlog_active:0
		repl_backlog_size:1048576
		repl_backlog_first_byte_offset:0
		repl_backlog_histlen:0
		10.211.55.10:6380> 

4. 查看sentinel的状态：

		[root@CentOS-3 redis-cluster]# redis-cli -h 10.211.55.10 -p 63791
		10.211.55.10:63791> info sentinel
		# Sentinel
		sentinel_masters:1
		sentinel_tilt:0
		sentinel_running_scripts:0
		sentinel_scripts_queue_length:0
		sentinel_simulate_failure_flags:0
		master0:name=master-1,status=ok,address=10.211.55.10:6379,slaves=2,sentinels=2
		10.211.55.10:63791> 
		
## 验证redis sentinel的主从切换：
1. 首先关闭master节点，即kill掉master进程
2. 查看sentinel服务，发现端口6381升级为master节点，这时sentinel完成故障自动切换。

		[root@CentOS-3 redis-cluster]# redis-cli -h 10.211.55.10 -p 63791
		10.211.55.10:63791> info sentinel
		# Sentinel
		sentinel_masters:1
		sentinel_tilt:0
		sentinel_running_scripts:0
		sentinel_scripts_queue_length:0
		sentinel_simulate_failure_flags:0
		master0:name=master-1,status=ok,address=10.211.55.10:6381,slaves=2,sentinels=2
		10.211.55.10:63791> 
3. 启动刚才被shutdown的6379服务并查看，发现它变成了slave服务。

		[root@CentOS-3 redis-cluster]# redis-cli -h 10.211.55.10 -p 6379
		10.211.55.10:6379> auth test123
		OK
		10.211.55.10:6379> info replication
		# Replication
		role:slave
		master_host:10.211.55.10
		master_port:6381
		master_link_status:up
		master_last_io_seconds_ago:2
		master_sync_in_progress:0
		slave_repl_offset:15074
		slave_priority:100
		slave_read_only:1
		connected_slaves:0
		master_repl_offset:0
		repl_backlog_active:0
		repl_backlog_size:1048576
		repl_backlog_first_byte_offset:0
		repl_backlog_histlen:0
		10.211.55.10:6379> 
		
## 附录
1. Redis搭建sentinel,无法主从自动切换,一直卡在-sdown sentinel

	redis-sentinel有保护模式，所以要将这个模块关闭！
	
		protected-mode no

2. redis-sentinel配置项详细说明

	参考[链接](http://blog.csdn.net/a1282379904/article/details/52335051)