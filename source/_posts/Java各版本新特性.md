---
title: Java各版本新特性
tags: 
  - Java
comments: true
categories: 
  - technology
toc: true
keywords: 'Java,JDK,版本,新特性,8,9,10,11,12,13,14,15'
date: 2020-08-30 17:27:32
---


# JDK各版本特性

## 历史版本主要特性


**JDK 1.5** ：enum、泛型，自动装箱与拆箱、可变参数、增强循环

JDK 1.6 ：支持脚本语言、JDBC4.0API

JDK 1.7： 支持try-with-resources、switch语句块增加String支持、NIO2.0包

**JDK 1.8**： lambda表达式、Stream API、新的日期时间的API、方法引用、构造器引用

JDK 9 ： 模块化系统、jshell

JDK 10 ： 局部变量的类型推断

**JDK 11**： ZGC的引入、Epsilon GC

JDK 12： switch表达式、Shenandoah GC、增强GC

JDK 13： switch表达式引入yield、文本块

JDK14 ：instanceof 模式识别、Records、启用Parallel Scavenge+Serial GC组合、删除CMS GC

## 1.8

![](https://images.di1shuai.com/2021-01-23-1.8.png)

### Lambda表达式

```java
System.out.println("foreach-less1.8");
//foreach-less1.8
List<String> strings = Arrays.asList("a","b","c","c");
for (String str:strings) {
    System.out.println(str);
}
System.out.println("-------");
System.out.println("foreach 1.8");
//foreach 1.8
strings.forEach(str -> System.out.println(str));
```

### Stream API

```java
//stream
strings.stream()
        .distinct()
        .limit(2)
        .filter(str -> str == "a")
        .forEach(str -> System.out.println(str));

System.out.println("===========");
System.out.println("peek");
strings.parallelStream()
        .skip(2)
        .forEach(str -> System.out.println(str));
System.out.println("===========");
distinctPrimary("12","23","12","23");

```

```java
    public static void distinctPrimary(String...numbers) {
        List<String> strings = Arrays.asList(numbers);
        List<Integer> collect = strings.parallelStream()
                .map(e -> new Integer(e))
//              .filter(e -> Primes.isPrime(e))
                .distinct()
                .collect(Collectors.toList());
        System.out.println(collect);
    }
```

### Interface

- 默认方法
- 静态方法

```java
public interface Animal {
    void run();

    default void jump(){
        System.out.println(this.getClass().getSimpleName()+ " jump");
    }

    static void eat(){
        System.out.println(Animal.class.getSimpleName()+" eat");
    }
}
```

### Date Time API - LocalDate & LocalDateTime

```java
public class LocalDateAndTimeAndDateTime {

    public static void main(String[] args) {
//        localdate();
//        localtime();
//        localdatetime();
//        test1();
        System.out.println(System.currentTimeMillis());
    }

    public static void test1(){
        LocalDateTime localDateTime1 = LocalDateTime.parse("2016-09-06T06:00:00");
        LocalDateTime localDateTime2 = LocalDateTime.parse("2016-09-06T07:00:00");
//        System.out.println(localDateTime1.getLong(ChronoField.NANO_OF_DAY));
        long time1 = localDateTime1.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();
        long time2 = localDateTime2.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli();

    }

    public static void localdate(){
        LocalDate today = LocalDate.now();
        System.out.println(today);

        LocalDate someday = LocalDate.of(2016,8,9);

        LocalDate withstr = LocalDate.parse("2016-08-09");

        if (someday.equals(withstr))
            System.out.println(true);
        else
            System.out.println(false);

        // 取本月第1天：
        LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth()); // 2016-08-09
        // 取本月第2天：
        LocalDate secondDayOfThisMonth = today.withDayOfMonth(2); // 2016-08-02
        // 取本月最后一天，再也不用计算是28，29，30还是31：
        LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth()); // 2016-08-31
        // 取下一天：
        LocalDate firstDayOf2015 = lastDayOfThisMonth.plusDays(1); // 变成了2016-09-01
        // 取2016年9月第一个周一，这个计算用Calendar要死掉很多脑细胞：
        LocalDate firstMondayOf2015 = LocalDate.parse("2016-09-01").with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2016-09-05
    }

    public static void localtime() {
        //包含毫秒
        LocalTime now = LocalTime.now();
        System.out.println(now);

        //清除毫秒数
        LocalTime time = LocalTime.now().withNano(0);
        System.out.println(time);

        //构造时间
        LocalTime zero = LocalTime.of(0, 0, 0); // 00:00:00
        LocalTime mid = LocalTime.parse("12:00:00"); // 12:00:00
        System.out.println(zero);
        System.out.println(mid);

    }

    public static void localdatetime() {
        long start = System.currentTimeMillis();
//        LocalDateTime now = LocalDateTime.now();
//        System.out.println(now);
//        LocalDateTime hello = LocalDateTime.parse("2016-08-09T12:00:00:000",DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss:zzz"));
//        System.out.println(hello);
        int i = 5;
        System.out.println(System.currentTimeMillis()-start);
    }
}
```

### Optional

```java
package com.diyishuai.java8.optional;

import com.diyishuai.java8.Student;

import java.util.Optional;

/**
 * @author Shea
 * @date 2021-01-20
 * @description
 */
public class OptionalDemo {

    private static boolean isNull = false;

    public static void main(String[] args) {
        String id = "666";

        //1.7
        Student student = getById(id);
        if (student != null) {
            System.out.println("if : name -> " + student.getName());
        }

        //1.8
        Optional<Student> studentOpt = Optional.ofNullable(getById(id));
        studentOpt.ifPresent(s ->
                System.out.println("opt : name -> " + s.getName())
        );
    }

    private static Student getById(String id) {
        return isNull ? null : new Student().setId(id).setName(id + " student");
    }

}

```

### Base64 - `java.util.Base64`

将Base64引入工具包

```java
package com.diyishuai.java8.base64;

import java.nio.charset.StandardCharsets;
import java.util.Base64;

/**
 * @author Shea
 * @date 2021-01-20
 * @description
 */
public class Base64Demo {

    private static Base64.Encoder encoder = Base64.getEncoder();

    private static Base64.Decoder decoder = Base64.getDecoder();

    public static void main(String[] args) {
        String str = "Hello Java8, 你好 Java8";
        System.out.println("原字符串 -> " + str);

        String encode = encode(str);
        System.out.println("Base64 Encode -> " + encode);

        String decode = decode(encode);
        System.out.println("Base64 Decode -> " + decode);
    }

    public static String decode(String encode) {
        return new String(decoder.decode(encode),StandardCharsets.UTF_8);
    }

    public static String encode(String str) {
        return encoder.encodeToString(str.getBytes(StandardCharsets.UTF_8));
    }

}
```

### 方法引用

- 类静态方法引用

```java
private static void classStaticFunctionReference() {
    System.out.println("类静态方法引用");
    List list = new ArrayList();

    list.add(new Apple("1"));
    list.add(new Apple("2"));
    list.add(new Apple("3"));
    list.add(new Apple[]{new Apple("4"), new Apple("5")});

    list.stream().forEach(Show::show);

}

class Show {

    public static void show(Object obj) {
        if (obj instanceof List) {
            for (Object o : (List) obj) {
                System.out.print(o.toString() + "\t");
            }
            System.out.println();
        } else if (obj instanceof Object[]) {
            for (Object o : (Object[]) obj) {
                System.out.print(o.toString() + "\t");
            }
            System.out.println();
        } else {
            System.out.println(obj.toString());
        }
    }

}
```

- 实例方法引用

```java
private static void instanceFunctionReference() {
    System.out.println("实例方法引用");
    List<Apple> list = new ArrayList();

    list.add(new Apple("1"));
    list.add(new Apple("2"));
    list.add(new Apple("3"));

    list.stream().map(a -> a.getId()).forEach(System.out::println);
    System.out.println("----");
    list.stream().map(Apple::getId).forEach(System.out::println);
}
```

- 类构造方法引用

无参构造

```java
// 0 无参构造
Supplier<Apple> appleSupplier = Apple::new;
Apple a0 = appleSupplier.get();
```

单参构造

```java
// 1 单参构造
Function<String, Apple> appleFunction = Apple::new;
Apple a1 = appleFunction.apply("1");
```

双参构造

```java
BiFunction<String,Double,Apple> appleBiFunction = Apple::new;
Apple a2 = appleBiFunction.apply("2", 0.5);
```

### JVM的新特性 - `Metaspace` 替代 `PermGen space`

使用元空间Metaspace代替持久代（PermGen space），

JVM参数使用 `-XX:MetaSpaceSize` 和 `-XX:MaxMetaspaceSize` 设置大小。

## 9

![](https://images.di1shuai.com/2021-01-23-9.png)

### 模块化

### 不可变集合工厂方法

< 9 

```java
// before 9
List<String> stringList = new ArrayList<>();
stringList.add("Hello ");
stringList.add("Java 9");
List<String> unmodifiableList = Collections.unmodifiableList(stringList);
```

9

```java
 // after 9
List<String> unmodifiableList_9 = List.of("Hello ","Java 9");
```

### 接口支持私有方法

```java
public interface PrivateInterface {

    private void cantFindMe(){
        System.out.println("cant find me");
    }

}
```

### Optional 改进

- `stream()`
- `ifPresentOrElse()`
- `or()`

```java
Optional<Integer> optional = Optional.of(1);
optional.ifPresentOrElse(
        x -> System.out.println("Value: " + x),
        () -> System.out.println("Not Present.")
);

optional = Optional.empty();

optional.ifPresentOrElse(
        x -> System.out.println("Value: " + x),
        () -> System.out.println("Not Present.")
);
```

### `try-with-resources` 改进

```java
public static void main(String[] args) throws IOException {
    System.out.println(readDataBefore9("TryWithResources"));
    System.out.println(readDataAfter9("TryWithResources"));
}

static String readDataBefore9(String message) throws IOException {
    try (BufferedReader br = new BufferedReader(new StringReader(message));) {
        return br.readLine();
    }
}

static String readDataAfter9(String message) throws IOException {
    Reader inputString = new StringReader(message);
    BufferedReader br = new BufferedReader(inputString);
    try (br) {
        return br.readLine();
    }
}
```

### Stream API 更新

- `takeWhile()` 返回第一次遇到false之前的元素
- `dropWhile()` 返回第一次遇到false之后的元素
- `iterate`
- `ofNullable`

原数据

```java
Supplier<Stream<Integer>> streamSupplier = () -> Stream.of(1, 2, 3, 4, 5);
```

`takeWhile()`

```java
System.out.println("takeWhile");
streamSupplier.get()
        .takeWhile(x -> x < 3)
        .forEach(System.out::println);

```

```java
takeWhile
1
2
```

`dropWhile`

```java
System.out.println("dropWhile");
        streamSupplier.get()
                .dropWhile(x -> x < 3)
                .forEach(System.out::println);
```

```java
dropWhile
3
4
5
```

`iterate`

```java
System.out.println("iterate");
IntStream.iterate(2, x -> x < 10000, x -> x * x).forEach(System.out::println);
```

```java
iterate
2
4
16
256
```

`ofNullable`

```java
Stream.ofNullable(100).forEach(System.out::println);
Stream.ofNullable(null).forEach(System.out::println);
```

```java
ofNullable
100
```

### 多版本兼容Jar包

### **其他**

- HTTP 2客户端 (支持 WebSocket和 HTTP2 流以及服务器推送)
- 进程API（控制和管理操作系统进程）
- String底层存储结构更改(char[]替换为byte[])
- 标识符添加限制( String _ ="hello"不能用)
- 响应式流 API (支持Java 9中的响应式编程)

## 10


### 局部变量类型推断

```java
// Before 10
String str = "hello ";
Integer i = 1;
System.out.println(str + i);

// After 10
var obj1 = "hi ";
var obj2 = Integer.parseInt("1");
System.out.println(obj1 + obj2);
System.out.println("obj1 -> " + obj1.getClass().getSimpleName());
System.out.println("obj2 -> " + obj2);
```

### 不可变集合更新

List，Set，Map 提供了一个新的静态方法 `copyOf(Collection<? extends E> coll)`，它返回Collection集合一个不可修改的副本

```java
var oldList = new ArrayList<String>();
oldList.add("hello");
oldList.add("Java10");

var copyList = List.copyOf(oldList);
oldList.add("hi");
try {
    copyList.add("Java10");

}catch (UnsupportedOperationException e){
    System.out.println("操作不允许");
    e.printStackTrace();
}
```

### 并行全垃圾回收器G1

JDK 9引入 G1 作为默认垃圾收集器，执行GC 时采用的是基于单线程标记扫描压缩算法（mark-sweep-compact）。为了最大限度地减少 Full GC 造成的应用停顿的影响，Java 10 中将为 G1 引入多线程并行 GC，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

### 线程本地握手

Java 10 中线程管控引入JVM安全点的概念，将允许在不运行全局JVM安全点的情况下实现线程回调，由线程本身或者JVM线程来执行，同时保持线程处于阻塞状态，这将会很方便使得停止单个线程或不停止线程成为可能。

### Optional 更新 `orElseThrow()`

`orElseThrow()`在没有值时抛出异常

```java
Optional<String> op;
try {
    op = Optional.of(null);
    op.orElseThrow();
} catch (RuntimeException e) {
    System.out.println("捕获到");
}
op = Optional.of("hi");
op.orElseThrow();
System.out.println(op.get());
```

### 其他

- 基于 Java 的 实验性 JIT 编译器
- 类数据共享
- Unicode 语言标签扩展
- 根证书
- 基于时间（Time-Based）的版本控制模型

## 11

![](https://images.di1shuai.com/2021-01-23-11.png)

### 字符串 API更新

新增

- isBlank() 判空。
- strip() 去除首尾空格
- stripLeading() 去除字符串首部空格
- stripTrailing() 去除字符串尾部空格
- lines() 分割获取字符串流。
- repeat() 复制字符串

```java
// 判空
System.out.println(
        "isBlank() ->" +
                "".isBlank() + "\t" +    // true
                "   ".isBlank() + "\t" + // true
                "1".isBlank() + "\t"    // false
);
// 去除首尾空格
System.out.println("strip() ->" + "Java11 ".strip()); //"Java11"
// 去除首部空格
System.out.println("stripLeading() ->" + " Java11 ".stripLeading());   // "Java11 "
// 去除字符串尾部空格
System.out.println("stripTrailing() ->"+" Java11 ".stripTrailing());   // " Java11"
// 行数统计
System.out.println("lines() ->"+"a\nb\nc".lines().count());    // 3
// 复制字符串
System.out.println("repeat(3) ->"+"Java11".repeat(3));   // "Java11Java11Java11"
```

### Lambda 推断类型参数的局部变量语法

```java
var map = new HashMap<String, Object>();
map.put("Hello", "Java11");
map.put("hi", 11);
map.forEach((var k, var v) -> {
    System.out.println(k + ": " + v);
});
```

### HTTPClient 标准化

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://di1shuai.com"))
        .GET()
        .build();

// 同步
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// 异步
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
        .thenApply(HttpResponse::body)
        .thenAccept(System.out::println);
```

### 单文件java命令直接编译运行

Before 11

```java
// 编译
javac Hello.java
// 运行
java Hello
```

11

```java
java Hello.java
```

### ZGC-可伸缩低延迟垃圾收集器

ZGC ，即 Z Garbage Collector（垃圾收集器或垃圾回收器）。它是一个可伸缩的、低延迟的垃圾收集器。ZGC 主要为了满足如下目标进行设计：

- GC 停顿时间不超过 10ms
- 既能处理几百 MB 的小堆，也能处理几个 TB 的大堆
- 应用吞吐能力不会下降超过 15%（与 G1 回收算法相比）
- 方便在此基础上引入新的 GC 特性和利用 colord
- 针以及 Load barriers 优化奠定基础
- 只支持 Linux/x64 位平台

### **其他**

- 添加 Epsilon 垃圾收集器。
- 支持 TLS 1.3 协议
- 飞行记录器分析工具
- 动态类文件常量
- 低开销的 Heap Profiling

## 12

![](https://images.di1shuai.com/2021-01-23-12.png)

### switch 增强(预览)

支持case多个

before 12

```java
switch (day) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        today = TODAY.WORKING_DAY;
        break;
    case SATURDAY:
    case SUNDAY:
        today = TODAY.WEEKEND_DAY;
        break;
    default:
        throw new RuntimeException("N/A");
}
```

12

```java
switch (day) {
      case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY 
							-> today = TODAY.WORKING_DAY;
      case SATURDAY, SUNDAY 
							-> today = TODAY.WEEKEND_DAY;
      default -> throw new IllegalArgumentException("Invalid day");
}
```

### `NumberFormat` - 数字的格式化

```java
long number = 100000;
NumberFormat nfChina = NumberFormat.getCompactNumberInstance(Locale.CHINA, NumberFormat.Style.SHORT);
System.out.println(nfChina.format(number));
//10万
NumberFormat nfUK = NumberFormat.getCompactNumberInstance(Locale.UK, NumberFormat.Style.SHORT);
System.out.println(nfUK.format(number));
//100K
```

### 字符串支持transform、indent操作

- transform 转换,可以配合函数式接口Function一起使用

```java
//transform
List.of("    Hello    ", "   Hi   ").forEach(
        s -> {
            String newStr = s.transform(String::strip)
                    .transform(e -> e + " Java 12");
            System.out.println(newStr);
        });
```

```java
Hello Java 12
Hi Java 12
```

- indent 缩进，每行开头增加空格space和移除空格

```java
//indent
String result = "Hello\nJava\n12\n".indent(3);
System.out.println(result);
```

```java
   Hello
   Java
   12
```

### Files.mismatch(Path, Path)

文件比较 返回第一个不匹配的位置，如果没有不匹配，则返回 -1L

```java
Path path1 = Paths.get("/Users/shuai/Documents/GitRepo/mine/language/java-versions/java-12/src/main/java/com/di1shuai/java12/files/FilesDemo.java");
Path path2 = Paths.get("/Users/shuai/Documents/GitRepo/mine/language/java-versions/java-12/src/main/java/com/di1shuai/java12/files/FilesDemo.java");

try {
    long fileMismatch = Files.mismatch(path1, path2);
    System.out.println(fileMismatch);
} catch (IOException e) {
    e.printStackTrace();
}
```

### Teeing Collector

Teeing Collector 是 Streams API 中引入的新的收集器实用程序，它的作用是 merge 两个 collector 的结果

```java
List<Student> students = Arrays.asList(
        new Student("shea", 28),
        new Student("abel", 25),
        new Student("sean", 40)
);
String collect = students.stream().collect(Collectors.teeing(
        Collectors.averagingInt(Student::getAge),
        Collectors.summingInt(Student::getAge),
        (avgAge, sumAge) -> avgAge + " : " + sumAge
));
System.out.println(collect);
```

```java
31.0 : 93
```

### **其他**

- 支持unicode 11（684个新字符、11个新blocks、7个新脚本）
- JVM 常量 API （主要在新的java.lang.invoke.constant包中定义了一系列基于值的符号引用类型，能够描述每种可加载常量。）
- Shenandoah GC（低暂停时间垃圾收集器）
- G1 收集器提升 （可中止的混合收集集合、及时返回未使用的已分配内存）
- 默认CDS档案
- JMH 基准测试

## 13

![](https://images.di1shuai.com/2021-01-23-13.png)

### Switch 增强 - yield

yield可直接返回值

<13

```java
// Before 13
TODAY today;
switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY 
				-> today = TODAY.WORKING_DAY;
    case SATURDAY, SUNDAY 
				-> today = TODAY.WEEKEND_DAY;
    default 
				-> throw new IllegalArgumentException("Invalid day");
}
return today;
```

13

```java
return switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> {
        yield TODAY.WORKING_DAY;
    }
    case SATURDAY, SUNDAY -> {
        yield TODAY.WEEKEND_DAY;
    }
    default -> throw new IllegalArgumentException("Invalid day");
};
```

 

### String文本块

<13

```java
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, Java 13</p>\n" +
              "    </body>\n" +
              "</html>\n";
```

13

```java
String html13 = """
                <html>
                    <body>
                        <p>Hello,  Java 13</p>
                    </body>
                </html>
                """;
```

### SocketAPI 重构

传统的Java Socket API（java.net.ServerSocket 和 java.net.Socket）依赖于SocketImpl 的内部实现

在 Java 13之前，通过使用 PlainSocketImpl 作为 SocketImpl 的具体实现。

Java 13 中的新底层实现，引入 NioSocketImpl 的实现用以替换 SocketImpl 的 PlainSocketImpl 实现，此实现与 NIO（新 I/O）实现共享相同的内部基础结构，并且与现有的缓冲区高速缓存机制集成在一起。

### FileSystems.newFileSystem

FileSystems 类中添加了以下三种新方法，以便更容易地使用将文件内容视为文件系统的文件系统提供程序：

- newFileSystem(Path)
- newFileSystem(Path, Map<String, ?>)
- newFileSystem(Path, Map<String, ?>, ClassLoader)

### **ZGC 增强 - 释放未使用内存**

ZGC 是Java 11 中引入的最为瞩目的垃圾回收特性，是一种可伸缩、低延迟的垃圾收集器。但是实际使用中，它不能够主动将未使用的内存释放给操作系统。

Java 13 中对 ZGC 的改进，包括释放未使用内存给操作系统、支持最大堆大小为 16TB、JVM参数 `-XX:SoftMaxHeapSize` 来软限制堆大小

### **其他**

- 动态 CDS 存档， 扩展了 Java 10 中引入的类数据共享功能， 使用CDS 存档变得更容易。
- 文本块的字符串类新方法，如formatted(Object…args)，stripIndent()等。

## 14


### instanceof 模式匹配改进

<14

```java
Object obj = "  hello Java 14  ";
// < 14
if (obj instanceof String) {
    System.out.println(((String) obj).strip());
}
```

14

```java
Object obj = "  hello Java 14  ";
// 14
if (obj instanceof String str) {
    System.out.println(str.strip());
}
```

### switch 表达式 - 标准化

### Record 类型（预览功能）

recode

```java
/**
 * 构造方法
 * hashCode() 方法
 * euqals() 方法
 * toString() 方法
 * 类对象被final 关键字修饰，不能被继承。
 */
record Student(String name,int age){

}
```

main

```java
public static void main(String[] args) {
        Student s = new Student("shea",18);
        System.out.println(
                """
                getName  -> %s        
                getAge   -> %s
                toString -> %s
                hashCode -> %s        
                """.formatted(
                        s.name(),
                        s.age(),
                        s.toString(),
                        s.hashCode()
                )
        );
    }
```

### NullPointerException 提示改进

14默认开启 `XX:+ShowCodeDetailsInExceptionMessages` 来显示详细信息

<14

```
java.lang.NullPointerException
  at com.di1shuai.java14.nullexception.NullPointerExceptionDemo.main(NullPointerExceptionDemo.java:21)
```

14

```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.toString()" because "str" is null
  at com.di1shuai.java14.nullexception.NullPointerExceptionDemo.main(NullPointerExceptionDemo.java:12)
```

### **其他**

- G1 的 NUMA 可识别内存分配
- 删除 CMS 垃圾回收器
- GC 支持 MacOS 和 Windows 系统

## 15

![](https://images.di1shuai.com/2021-01-23-15.png)

### EdDSA 数字签名算法

- 使用 Edwards-Curve 数字签名算法（EdDSA）实现加密签名。
- 与其它签名方案相比，EdDSA 具有更高的安全性和性能。
- 得到许多其它加密库（如 OpenSSL、BoringSSL）的支持。

### Sealed Classes（封闭类，预览）

封闭类 `sealed` class
`permits`可以允许三种类型的子类进行继承
1. final 类
2. non-sealed 不封闭类
3. sealed 类

封闭父类

```java
public sealed class SealedDemo
        permits FinalClass, NonSealedClass, SealedClass {

}
```

`final`

```
final class FinalClass extends SealedDemo {

}
```

`non-sealed`

```
non-sealed class NonSealedClass extends SealedDemo {

}
```

`sealed`

```
sealed class SealedClass extends SealedDemo permits SealedClass.SubSealed {

    non-sealed class SubSealed extends SealedClass {

    }
}
```

### Hidden Classes（隐藏类）

隐藏类天生为框架设计的。

隐藏类只能通过反射访问，不能直接被其他类的字节码

### 其他

- Disable and Deprecate Biased Locking（准备禁用偏向锁）
- instanceof 自动匹配模式（二次预览）
- ZGC，一个可伸缩、低延迟的垃圾回收器。（转正）
- Text Blocks，文本功能转正（JDK 13和14预览，14终于转正）
- Remove the Solaris and SPARC Ports（删除 Solaris 和 SPARC 端口）
- 外部存储器访问 API（允许Java 应用程序安全有效地访问 Java 堆之外的外部内存。）
- Record类型二次预览（在Java 14就预览过啦）

## 源码

[源码](https://github.com/BestBurning/java-versions)已发布在Github上，欢迎Star