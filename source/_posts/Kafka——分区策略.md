---
title: Kafka——分区策略
comments: true
tags: 
- Kafka
- BigData
categories: 
- technology
keywords: Kafka,分区策略,Partitioner,Assignor
toc: true
date: 2020-07-09 15:50:04
---


# Producer
## Partitioner 分区器

### DefaultPartitioner 默认分区器
- Key == Null

Kafka `2.4`之前的无Key策略是循环使用主题的所有分区，将消息以轮询的方式发送到每一个分区上,`2.4`之后增加了默认的粘性策略即：

对于同一批的数据，会用一个随机值对可用partition数量进行取模，然后把这个partition缓存起来

- Key ≠ Null

Hash key后，对partition数量进行取模

```
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster,
                         int numPartitions) {
        if (keyBytes == null) {
            return stickyPartitionCache.partition(topic, cluster);
        }
        // hash the keyBytes to choose a partition
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
```

[【译】Kafka Producer Sticky Partitioner](https://www.cnblogs.com/huxi2b/p/12540092.html)

### RoundRobinPartitioner 轮询分区器

```
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        int nextValue = nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (!availablePartitions.isEmpty()) {
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return availablePartitions.get(part).partition();
        } else {
            // no partitions are available, give a non-available partition
            return Utils.toPositive(nextValue) % numPartitions;
        }
    }

    private int nextValue(String topic) {
        AtomicInteger counter = topicCounterMap.computeIfAbsent(topic, k -> {
            return new AtomicInteger(0);
        });
        return counter.getAndIncrement();
    }
```

### UniformStickyPartitioner 粘滞分区器
```
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        return stickyPartitionCache.partition(topic, cluster);
    }


```

# Consumer
## Consumer  Assignor

当ConsumerGroupLeader收到来自CoordinatorGroup的member信息之后，会进行分区，分区策略主要有:

### RangeAssignor 范围分区  默认

先用 `partition` / `consumer` = 每个消费者至少要消费的分区个数

再用 `partition`% `consumer` = 字典序前多少个消费者需要多消费一个 

```java
int numPartitionsPerConsumer = numPartitionsForTopic / consumersForTopic.size();
int consumersWithExtraPartition = numPartitionsForTopic % consumersForTopic.size();

List<TopicPartition> partitions = AbstractPartitionAssignor.partitions(topic, numPartitionsForTopic);
for (int i = 0, n = consumersForTopic.size(); i < n; i++) {
    int start = numPartitionsPerConsumer * i + Math.min(i, consumersWithExtraPartition);
    int length = numPartitionsPerConsumer + (i + 1 > consumersWithExtraPartition ? 0 : 1);
    assignment.get(consumersForTopic.get(i).memberId).addAll(partitions.subList(start, start + length));
}
```

但是当消费多个topic，并且每个topic的partition对cunsumer取余后都多一些，那么会导致靠前的消费者消费较多分区，靠后的消费者消费较少分区，出现分区不均匀

### RoundRobin 轮询分区

先将所有消费的的partition装在List里面，然后用一个装了consumer环形迭代器去碰撞

```java
CircularIterator<MemberInfo> assigner = new CircularIterator<>(Utils.sorted(memberInfoList));

for (TopicPartition partition : allPartitionsSorted(partitionsPerTopic, subscriptions)) {
    final String topic = partition.topic();
    while (!subscriptions.get(assigner.peek().memberId).topics().contains(topic))
        assigner.next();
    assignment.get(assigner.next().memberId).add(partition);
}
```

```java
private List<TopicPartition> allPartitionsSorted(Map<String, Integer> partitionsPerTopic,
                                                     Map<String, Subscription> subscriptions) {
    SortedSet<String> topics = new TreeSet<>();
    for (Subscription subscription : subscriptions.values())
        topics.addAll(subscription.topics());

    List<TopicPartition> allPartitions = new ArrayList<>();
    for (String topic : topics) {
        Integer numPartitionsForTopic = partitionsPerTopic.get(topic);
        if (numPartitionsForTopic != null)
            allPartitions.addAll(AbstractPartitionAssignor.partitions(topic, numPartitionsForTopic));
    }
    return allPartitions;
}
```

### StickyAssignor 粘性分配

从 `0.11` 版本开始 

目标：

主题分区仍然尽可能均匀地**分布** 

主题分区尽可能与其**先前**分配的使用者**在一起**

[深入分析Kafka架构（三）：消费者消费方式、三种分区分配策略、offset维护 - osc_8vayftu3的个人空间 - OSCHINA](https://my.oschina.net/u/4262150/blog/3274346)

### CooperativeStickyAssignor

从 `2.4` 版本开始
