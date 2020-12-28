---
title: Kafka常用操作(不定期更新)
comments: true
date: 2016-10-28
keywords: Kafka,操作
tags: 
- Kafka
- BigData
categories: 
- technology

---
#### 说明
[kafka官网](http://kafka.apache.org/)
kafka version 2.10
所有kafka的操作都是基于kafka2.10包下bin目录下的脚本文件

#### 操作
1. 启动zookeeper
```
./zookeeper-server-start.sh ../config/zookeeper.properties
```
2. 启动kafka
```
./kafka-server-start.sh ../config/server.properties
```
3. 停止所有kafka server
```
./kafka-server-stop.sh
```
4. 创建topic
```
./kafka-topics.sh --create --topic my-topic --partitions 1 --replication-factor 1 --zookeeper localhost:2181
```
5. 查看所有topic
```
./kafka-topics.sh --list --zookeeper localhost:2181
```
6. 查看指定topic信息
```
./kafka-topics.sh --describe --topic my-topic  --zookeeper localhost:2181
```
7. 更改topic的配置信息
```
./kafka-topics.sh --alter --topic my-topic --partitions 3 --zookeeper localhost:2181
```
	说明：本操作将my-topic的partitions更改为3个，要求至少启动三个kafka broker（也就是kafka server）
8. 删除topic
```
./kafka-topics.sh --delete --topic my-topic --zookeeper localhost:2181
```
9. producer向某个topic丢数据,然后在命令行内敲数据消费端就能够收到
```
./kafka-console-producer.sh --broker-list=localhost:9092,localhost:9093,localhost:9094 --topic my-topic
```
10. consumer消费某个topic
```
./kafka-console-consumer.sh --topic my-topic --zookeeper localhost:2181
```
11. 
