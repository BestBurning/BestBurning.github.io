---
title: Flink-DataStream-Transformations
comments: true
tags: 
- Flink
- BigData
categories: 
- technology
keywords: Flink,Transformation
toc: true
date: 2020-12-26 15:50:04
---

## DataStream → DataStream

### Map

对每个成员进行操作

`dataStream.map { x => x * 2 }`

例如我们将测温枪数据流中的每一行转化为样例类

原始数据

```
id,时间戳,温度

sensor_4,1609203345495,39.126604003147314
sensor_2,1609203345495,39.42746976779783
sensor_8,1609203345495,38.573523839218375
sensor_6,1609203345495,37.65574402266021
sensor_1,1609203345495,35.97294873181826
sensor_10,1609203345495,35.11395246803552
sensor_5,1609203345495,37.17056374401738
...
```

样例类

```scala
case class SensorReader(id: String, timestamp: Long, temperature: Double)
```

map转化

```scala
    val dataStream = inputStream
      .map(
        data => {
          val array = data.split(",")
          SensorReader(array(0), array(1).toLong, array(2).toDouble)
        }
      )

```

### Filter

过滤

```scala
dataStream.filter { _ != 0 }
```

### FlatMap

压平

```scala
dataStream.flatMap { str => str.split(" ") }
```

## DataStream* → DataStream

### Union

合并多个流,要求数据格式一致

```
dataStream.union(otherStream1, otherStream2, ...)
```

```scala
    //  Union 合并 必须同类型，但可以多个
    val unionStream = highStream.union(normalStream)
```

## DataStream → KeyedStream

### KeyBy

按照key来分组

```scala
    // aggregation 分组聚合，计算每个传感器当前的最小温度
    val aggStream = dataStream
      .keyBy(_.id) // 根据ID分组
      //      .min("temperature")    // 温度最小的一组数据
      .minBy("temperature") //只保持温度最小,不影响时间的数据

```

## KeyedStream → DataStream

### Aggregations

```
keyedStream.sum(0)
keyedStream.sum("key")
keyedStream.min(0)
keyedStream.min("key")
keyedStream.max(0)
keyedStream.max("key")
keyedStream.minBy(0)
keyedStream.minBy("key")
keyedStream.maxBy(0)
keyedStream.maxBy("key")
```

## DataStream,DataStream → ConnectedStreams

### Connect

连接流，只能连接两个，但可以数据格式不同

```
someStream : DataStream[Int] = ...
otherStream : DataStream[String] = ...
val connectedStreams = someStream.connect(otherStream)
```

```scala
    //  Connect 合流 可以不同类型 但只能两个
    val warnStream = highStream.map(data => (data.id, data.temperature))
    val connectedStream = warnStream.connect(normalStream)
    val coMapResultStream = connectedStream
      .map(
        warnData => (warnData._1, warnData._2, "warning"),
        normalData => (normalData.id, "healthy")
      )
```

## Side Outputs

侧向输出用于分流

假定目前有一批测温枪的体温数据，按照温度分为37°以上的高温和37°以下的正常温度

```scala
        // 实现分流，按温度>37 和<=37 分为高温、常温流
    val highTag = OutputTag[SensorReader]("high-temperature")
    val normalTag = OutputTag[SensorReader]("normal-temperature")
    //  lambda
    val allStream = dataStream.process(
      (value, ctx: ProcessFunction[SensorReader, SensorReader]#Context, out: Collector[SensorReader]) => {
        out.collect(value) //正常输出
        if (value.temperature > 37) {
          ctx.output(highTag, value) //输出为高温
        } else {
          ctx.output(normalTag, value) // 输出为常温
        }
      }
    )
        val highStream = allStream.getSideOutput(highTag)
    val normalStream = allStream.getSideOutput(normalTag)

        highStream.print("high")
    normalStream.print("normal")
    allStream.print("all")
```

## 参考

[Flink Operators](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/)
[Flink Side Outputs](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/side_output.html)

## 源码

源码放在了[Github](https://github.com/BestBurning)上，见[flink-scala](https://github.com/BestBurning/bigdata/blob/master/flink/flink-scala/src/main/scala/com/di1shuai/flink/scala/Transformations.scala)

