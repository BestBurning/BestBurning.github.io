---
title: Java8新特性
tags: 
  - Java
comments: true
categories: 
  - technology
toc: true
keywords: 'Java8,JDK1.8,版本,新特性'
date: 2016-12-20 17:27:32
---

### Lambda

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

### Stream

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

```
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

### LocalDate & LocalDateTime

```
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

### 源码
[源码](https://github.com/BestBurning/java-versions/tree/master/java-8)已发布到了Github上，欢迎Star