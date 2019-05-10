---
title: Java Time API
date: 2019-05-10 14:57:18
tags: "Java"
categories: "Java"
---

一、java.util
java.util 包提供了 Date 类来封装当前的日期和时间。 Date 类提供两个构造函数来实例化 Date 对象。

```java
// 当前日期，输出：Fri May 10 14:15:23 CST 2019
System.out.println(new Date()); 

// 格式化日期，输出：2019-05-10 14:18:03
System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));  

// 字符串日期解析，输出：Sun May 05 12:30:05 CST 2019
String dateStr = "2019-05-05 12:30:05";
System.out.println(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse(dateStr));
```

二、java.time
这个包是在Java8中引入的。在Java 8之前，所有关于时间和日期的API都存在各种使用方面的缺陷，主要有：

1. Java的`java.util.Date`和`java.util.Calendar`类易用性差，不支持时区，而且他们都不是线程安全的；
2. 用于格式化日期的类`DateFormat`被放在`java.text`包中，它是一个抽象类，所以我们需要实例化一个`SimpleDateFormat`对象来处理日期格式化，并且`DateFormat`也是非线程安全，这意味着如果你在多线程程序中调用同一个`DateFormat`对象，会得到意想不到的结果。
3. 对日期的计算方式繁琐，而且容易出错，因为月份是从0开始的，从`Calendar`中获取的月份需要加一才能表示当前月份。

增加的API：

1. LocalDate 表示一个具体的日期，不包含具体的时间和时区信息。

   ```java
   LocalDate now = LocalDate.now();                    // 当前日期
   LocalDate localDate = LocalDate.of(2017, 1, 4);     // 初始化一个日期：2017-01-04
   int year = localDate.getYear();                     // 年份：2017
   Month month = localDate.getMonth();                 // 月份：JANUARY
   int dayOfMonth = localDate.getDayOfMonth();         // 月份中的第几天：4
   DayOfWeek dayOfWeek = localDate.getDayOfWeek();     // 一周的第几天：WEDNESDAY
   int length = localDate.lengthOfMonth();             // 月份的天数：31
   boolean leapYear = localDate.isLeapYear();          // 是否为闰年：false
   ```

2. LocalTime 表示一句包含具体时间的日期

   ```java
   LocalTime localTime = LocalTime.of(17, 23, 52);     // 初始化一个时间：17:23:52
   int hour = localTime.getHour();                     // 时：17
   int minute = localTime.getMinute();                 // 分：23
   int second = localTime.getSecond();                 // 秒：52
   ```

3. LocalDateTime 是LocalDate和LocalTime的结合

   ```java
   LocalDateTime ldt1 = LocalDateTime.of(2017, Month.JANUARY, 4, 17, 23, 52);
   LocalDate localDate = LocalDate.of(2017, Month.JANUARY, 4);
   LocalTime localTime = LocalTime.of(17, 23, 52);
   LocalDateTime ldt2 = localDate.atTime(localTime);
   
   // LocalDate与LocalTime的转化
   LocalDate date = ldt1.toLocalDate();
   LocalTime time = ldt1.toLocalTime();
   ```

4. Instant 表示一个精确到纳秒的时间戳，`System.currentTimeMillis()`只精确到毫秒

   ```java
   // 当前时间戳
   Instant now = Instant.now()
   
   // ofEpochSecond()方法的第一个参数为秒，第二个参数为纳秒
   // 表示从1970-01-01 00:00:00开始后两分钟的10万纳秒的时刻
   // 输出1970-01-01T00:02:00.000100Z
   Instant instant = Instant.ofEpochSecond(120, 100000);
   ```

5. Duration 表示一个精确到纳秒的时间段

   ```java
   // 2017-01-05 10:07:00
   LocalDateTime from = LocalDateTime.of(2017, Month.JANUARY, 5, 10, 7, 0);    
   
   // 2017-02-05 10:07:00
   LocalDateTime to = LocalDateTime.of(2017, Month.FEBRUARY, 5, 10, 7, 0);
   
   // 表示从 2017-01-05 10:07:00 到 2017-02-05 10:07:00 这段时间
   Duration duration = Duration.between(from, to);     
   
   long days = duration.toDays();              // 这段时间的总天数
   long hours = duration.toHours();            // 这段时间的小时数
   long minutes = duration.toMinutes();        // 这段时间的分钟数
   long seconds = duration.getSeconds();       // 这段时间的秒数
   long milliSeconds = duration.toMillis();    // 这段时间的毫秒数
   long nanoSeconds = duration.toNanos();      // 这段时间的纳秒数
   
   // 通过of()方法创建，接受一个时间段长度，和一个时间单位作为参数
   Duration duration1 = Duration.of(5, ChronoUnit.DAYS);       // 5天
   Duration duration2 = Duration.of(1000, ChronoUnit.MILLIS);  // 1000毫秒
   ```

6. Period 表示以年月日来衡量时间段

   ```java
   // 长度为2年3个月6天的时间段
   Period period = Period.of(2, 3, 6);
   
   / /通过between()方法创建，只接收LocalDate类型的参数
   // 2017-01-05 到 2017-02-05 这段时间
   Period period = Period.between(
                   LocalDate.of(2017, 1, 5),
                   LocalDate.of(2017, 2, 5));
   ```

相关操作：

1. 与原有API的转化

   ```java
   // Date转LocalDate
   Date date = new Date();
   LocalDateTime localDateTime = date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
   ```

2. 增加和减少日期

   ```java
   LocalDate date = LocalDate.of(2017, 1, 5);          // 2017-01-05
   
   LocalDate date1 = date.withYear(2016);              // 修改为 2016-01-05
   LocalDate date2 = date.withMonth(2);                // 修改为 2017-02-05
   LocalDate date3 = date.withDayOfMonth(1);           // 修改为 2017-01-01
   
   LocalDate date4 = date.plusYears(1);                // 增加一年 2018-01-05
   LocalDate date5 = date.minusMonths(2);              // 减少两个月 2016-11-05
   LocalDate date6 = date.plus(5, ChronoUnit.DAYS);    // 增加5天 2017-01-10
   
   LocalDate date7 = date.with(nextOrSame(DayOfWeek.SUNDAY));      // 返回下一个距离当前时间最近的星期日
   LocalDate date9 = date.with(lastInMonth(DayOfWeek.SATURDAY));   // 返回本月最后一个星期六
   ```

3. 格式化日期

   ```java
   LocalDateTime dateTime = LocalDateTime.now();
   String strDate1 = dateTime.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20170105
   String strDate2 = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2017-01-05
   String strDate3 = dateTime.format(DateTimeFormatter.ISO_LOCAL_TIME);    // 14:20:16.998
   String strDate4 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));   // 2017-01-05
   
   // 今天是：2017年 一月 05日 星期四
   String strDate5 = dateTime.format(DateTimeFormatter.ofPattern("今天是：YYYY年 MMMM DD日 E", Locale.CHINESE)); 
   
   String strDate6 = "2017-01-05";
   String strDate7 = "2017-01-05 12:30:05";
   
   LocalDate date = LocalDate.parse(strDate6, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
   LocalDateTime dateTime1 = LocalDateTime.parse(strDate7, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
   ```

4. 时区与历法，见参考，这里不贴了

参考
[1] [Java 8新特性（四）：新的时间和日期API](https://lw900925.github.io/java/java8-newtime-api.html)
[2] [Java8 日期/时间（Date Time）API指南]([http://www.importnew.com/14140.html](http://www.importnew.com/14140.html))
[3] [Java 日期时间](https://www.runoob.com/java/java-date-time.html)