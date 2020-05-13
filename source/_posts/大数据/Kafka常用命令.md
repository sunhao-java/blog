title: Kafka常用命令
tags: [kafka]
date: 2020-05-13
categories: 大数据
---

1. 查看kafka topic列表，使用--list参数
    
    ```
    ./kafka-topics.sh --zookeeper 192.168.137.100:2181 --list
    ```

2. 查看kafka特定topic的详情，使用--topic与--describe参数

    ```
    ./kafka-topics.sh --zookeeper 192.168.137.100:2181 --topic dcc-topic --describe
    ```

3. 查看consumer group列表，使用--list参数

    ```
    ./kafka-consumer-groups.sh --bootstrap-server 192.168.137.100:9292 --list
    ```

4. 查看特定consumer group 详情，使用--group与--describe参数
    
    ```
    ./kafka-consumer-groups.sh --bootstrap-server 192.168.137.100:9092 --group flume_test --describe
    ```
