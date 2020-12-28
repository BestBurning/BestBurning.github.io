---
title: Flink数据从Source流入方式
comments: true
tags: 
- Flink
- BigData
categories: 
- technology
keywords: Flink,Source,Kafka
toc: true
date: 2020-6-28 15:50:04
---

## Source

### 集合

```scala
case class SensorReader(id: String, timestamp: Long, temperature: Double)

val dataList = List(
      SensorReader("1", now, 36.5),
      SensorReader("2", now - 30 * 1000, 39),
      SensorReader("3", now - 300 * 1000, 37.1),
      SensorReader("4", now, 36.2)
    )

val dataStream = env.fromCollection(dataList)
```

 

### 任意类型元素

```scala

val dataStream = env.fromElements(1,1.1,"hello",true)
//    val dataStream = env.readTextFile(inputPath)

//4 Kafka
//    val consumerProperties = new Properties()
//    consumerProperties.setProperty("bootstrap.servers", "kafka1:9092")
//    consumerProperties.setProperty("group.id", "flink-stream")
//    val topic = "flink-source";
//    val kafkaSource = new FlinkKafkaConsumer[String](topic,new SimpleStringSchema(),consumerProperties)
//
//    val dataStream = env.addSource(kafkaSource)
```

### 文件

```scala
//3 文件
val inputPath: String = "/path/to/file"
val dataStream = env.readTextFile(inputPath)
```

### **Kafka【重要】**

```scala
val consumerProperties = new Properties()
consumerProperties.setProperty("bootstrap.servers", "kafka1:9092")
consumerProperties.setProperty("group.id", "flink-stream")
val topic = "flink-source";
val kafkaSource = new FlinkKafkaConsumer[String](topic,new SimpleStringSchema(),consumerProperties)

val dataStream = env.addSource(kafkaSource)
```

### 自定义Source

场景一：测试

场景二：其他的数据源

```scala
val dataStream = env.addSource(new SensorReaderSource())

case class SensorReader(id: String, timestamp: Long, temperature: Double)

//继承SourceFunction，实现run()/cancel()
class SensorReaderSource() extends SourceFunction[SensorReader] {
  var running: Boolean = true
  val random = Random

  var currentTemp = 1.to(10).map(i => SensorReader("sensor_" + i, System.currentTimeMillis(), random.nextDouble() * 5 + 35)).toList

  def getDataList(): List[SensorReader] = {
    currentTemp.map(
      s => SensorReader(s.id,System.currentTimeMillis(),s.temperature+random.nextGaussian())
    )

  }

  override def run(ctx: SourceFunction.SourceContext[SensorReader]): Unit =
    while (running) {
      getDataList().foreach(
				// ctx.collect 用于发送数据
        data => ctx.collect(data)
      )
      Thread.sleep(1000)
    }

	//取消
  override def cancel(): Unit = running = false
}
```

## 参考

[Flink Data Sources](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/sources.html)

## 源码

源码放在了[Github](https://github.com/BestBurning)上，见[flink-scala](https://github.com/BestBurning/bigdata/blob/master/flink/flink-scala/src/main/scala/com/di1shuai/flink/scala/SourceDemo.scala)

