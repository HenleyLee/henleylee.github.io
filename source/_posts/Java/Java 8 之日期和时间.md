---
title: Java 8 之日期和时间
date: 2018-11-28 19:36:12
categories: Java
tags:
  - Java
---

## 前言 ##
随着[`lambda`](http://javarevisited.blogspot.sg/2014/02/10-example-of-lambda-expressions-in-java8.html)表达式、[`streams`](http://java67.blogspot.sg/2014/04/java-8-stream-api-examples-filter-map.html)以及一系列小优化，Java8 推出了全新的日期时间 API。

Java 处理日期、日历和时间的方式一直为社区所诟病，将 `java.util.Date` 设定为可变类型，以及[SimpleDateFormat 的非线程安全](http://javarevisited.blogspot.sg/2012/03/simpledateformat-in-java-is-not-thread.html)使其应用非常受限。Java 也意识到需要一个更好的 API 来满足社区中已经习惯了使用 `JodaTime API` 的人们。

全新 API 的众多好处之一就是，明确了日期时间概念，例如：`瞬时(instant)`、`期间(duration)`、`日期`、`时间`、`时区`和`周期`。同时继承了 `Joda` 库按人类语言和计算机各自解析的时间处理方式。不同于老版本，新 API 基于 ISO 标准日历系统，`java.time` 包下的所有类都是不可变类型而且线程安全。

## java.time 的关键类 ##
新版 API 中 `java.time` 包含以下关键类：
 - **`Instant：`**瞬时实例。
 - **`LocalDate：`**本地日期，不包含具体时间。例如：`2014-01-14` 可以用来记录生日、纪念日、加盟日等。
 - **`LocalTime：`**本地时间，不包含日期。
 - **`LocalDateTime：`**组合了日期和时间，但不包含时差和时区信息。
 - **`ZonedDateTime：`**最完整的日期时间，包含时区和相对 UTC 或格林威治的时差。

新 API 还引入了 `ZoneOffSet` 和 `ZoneId` 类，使得解决时区问题更为简便。[解析和格式化时间](http://javarevisited.blogspot.sg/2011/09/step-by-step-guide-to-convert-string-to.html)的 `DateTimeFormatter` 类也全部重新设计。

## 日期和时间 API 的使用 ##
常有人问我学习一个新库的最好方式是什么？我的答案是在实际项目中使用它。项目中有很多真正的需求驱使开发者去发掘并学习新库。简单得说就是任务驱动学习探索。这对Java8新日期时间`API`也不例外。我创建了20个基于任务的实例来学习Java8的新特性。从最简单创建当天的日期开始，然后创建时间及时区，接着模拟一个日期提醒应用中的任务——计算重要日期的到期天数，例如生日、纪念日、账单日、保费到期日、信用卡过期日等。

### 1. 获取当前日期 ###
Java8 中的 `LocalDate` 用于表示当天日期。和 `java.util.Date` 不同，它只有日期，不包含时间。当仅需要表示日期时就用这个类。
```java
public static void getCurrentDate() {
    LocalDate today = LocalDate.now();
    System.out.println("Today's Local date : " + today);

    // 这个是作为对比
    Date date = new Date();
    System.out.println("Today's Local date : " + date);
}
```

上面的代码创建了当天的日期，不含时间信息。打印出的日期格式非常友好，不像老的 `Date` 类打印出一堆没有格式化的信息。

### 2. 获取当前的年、月、日信息 ###
`LocalDate` 类提供了获取年、月、日的快捷方法，其实例还包含很多其它的日期属性。通过调用这些方法就可以很方便的得到需要的日期信息，不用像以前一样需要依赖 `java.util.Calendar` 类了。
```java
public static void getCurrentDetailDate() {
    LocalDate today = LocalDate.now();
    int year = today.getYear();
    int month = today.getMonthValue();
    int day = today.getDayOfMonth();

    System.out.printf("Year : %d  Month : %d  day : %d %n", year, month, day);
}
```

运行后可以看到，在 Java8 中得到年、月、日信息是这么简单直观，想用就用，没什么需要记的。对比看看以前 Java 是怎么处理年月日信息的吧。

### 3. 处理特定日期 ###
在第一个例子里，我们通过静态工厂方法 `now()` 非常容易地创建了当天日期，还可以调用另一个有用的工厂方法 `LocalDate.of()` 创建任意日期，该方法需要传入年、月、日做参数，返回对应的 `LocalDate` 实例。这个方法的好处是没再犯老 API 的设计错误，比如年度起始于1900，月份是从0开始等等。日期所见即所得，就像下面这个例子表示了1月14日，没有任何隐藏机关。
```java
public static void handleSpecilDate() {
    LocalDate dateOfBirth = LocalDate.of(2010, 1, 14);
    System.out.println("Your Date of birth is : " + dateOfBirth);
}
```

运行后可以看到创建的日期完全符合预期，与写入的2010年1月14日完全一致。

### 4. 判断两个日期是否相等 ###
现实生活中有一类时间处理就是判断两个日期是否相等。常常会检查今天是不是个特殊的日子，比如生日、纪念日或非交易日。这时就需要把指定的日期与某个特定日期做比较，例如判断这一天是否是假期。下面这个例子会用 Java8 的方式去解决，`LocalDate` 重载了 `equal` 方法，请看下面的例子：
```java
public static void compareDate() {
    LocalDate today = LocalDate.now();
    LocalDate date1 = LocalDate.of(2014, 1, 14);
    if (date1.equals(today)) {
        System.out.printf("Today %s and date1 %s are same date %n", today, date1);
    }
}
```

这个例子中我们比较的两个日期相同。注意，如果比较的日期是字符型的，需要先解析成日期对象再作判断。对比[Java 老的日期比较方式](http://javarevisited.blogspot.sg/2012/02/3-example-to-compare-two-dates-in-java.html)，你会感到清风拂面。

### 5. 检查像生日这种周期性事件 ###
Java 中另一个日期时间的处理就是检查类似每月账单、结婚纪念日、EMI 日或保险缴费日这些周期性事件，比如在类似生日、纪念日、法定假日(国庆以及春节)、或者每个月固定时间发送邮件给客户这些周期性事件。Java 中如何检查这些节日或其它周期性事件呢？答案就是 `MonthDay` 类。这个类组合了月份和日，去掉了年，这意味着你可以用它判断每年都会发生事件。和这个类相似的还有一个 `YearMonth` 类。这些类也都是不可变并且线程安全的值类型。下面我们通过 `MonthDay` 来检查周期性事件：
```java
public static void recurringDate() {
    LocalDate today = LocalDate.now();
    LocalDate dateOfBirth = LocalDate.of(2010, 1, 14);
    MonthDay birthday = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
    MonthDay currentMonthDay = MonthDay.from(today);

    if (currentMonthDay.equals(birthday)) {
        System.out.println("Many Many happy returns of the day");
    } else {
        System.out.println("Sorry, today is not your birthday");
    }
}
```

只要当天的日期和生日匹配，无论是哪一年都会打印出祝贺信息。可以把程序整合进系统时钟，看看生日时是否会受到提醒，或者写一个单元测试来检测代码是否运行正确。

### 6. 获取当前时间 ###
与 Java8 获取日期的例子很像，获取时间使用的是 `LocalTime` 类，一个只有时间没有日期的 `LocalDate` 的近亲。可以调用静态工厂方法 `now()` 来获取当前时间。默认的格式是 `hh:mm:ss:nnn`。对比一下[Java8 之前获取当前时间的方式](http://javarevisited.blogspot.sg/2012/01/get-current-date-timestamps-java.html)。
```java
public static void getCurrentTime() {
    LocalTime time = LocalTime.now();
    System.out.println("Local time now : " + time);
}
```

运行后可以看到当前时间就只包含时间信息，没有日期。

### 7. 在现有的时间上增加小时 ###
通过增加小时、分、秒来计算将来的时间很常见。Java8 除了不变类型和线程安全的好处之外，还提供了更好的 `plusHours()` 方法替换 `add()`，并且是兼容的。注意，这些方法返回一个全新的 `LocalTime` 实例，由于其不可变性，返回后一定要用变量赋值。
```java
public static void plusHours() {
    LocalTime time = LocalTime.now();
    LocalTime newTime = time.plusHours(2); // adding two hours
    System.out.println("Local time now : " + time);
    System.out.println("Time after 2 hours : " +  newTime);
}
```

运行后可以看到，新的时间在当前时间的基础上增加了2个小时。和[旧版 Java 的增减时间的处理方式](http://javarevisited.blogspot.sg/2012/12/how-to-add-subtract-days-months-years-to-date-time-java.html)对比一下，看看哪种更好。

### 8. 计算一周后的日期 ###
和上个例子计算两小时以后的时间类似，这个例子会计算一周后的日期。`LocalDate` 日期不包含时间信息，它的 `plus()` 方法用来增加天、周、月，`ChronoUnit` 类声明了这些时间单位。由于 `LocalDate` 也是不变类型，返回后一定要用变量赋值。
```java
public static void nextWeek() {
    LocalDate today = LocalDate.now();
    LocalDate nextWeek = today.plus(1, ChronoUnit.WEEKS);
    System.out.println("Today is : " + today);
    System.out.println("Date after 1 week : " + nextWeek);
}
```

运行后可以看到新日期离当天日期是7天，也就是一周。你可以用同样的方法增加1个月、1年、1小时、1分钟甚至一个世纪，更多选项可以查看 `Java 8 API` 中的 `ChronoUnit`类。

### 9. 计算一年前或一年后的日期 ###
继续上面的例子，上个例子中我们通过 `LocalDate` 的 `plus()` 方法增加天数、周数或月数，这个例子我们利用 `minus()` 方法计算一年前的日期。
```java
public static void minusDate() {
    LocalDate today = LocalDate.now();
    System.out.println("Today's Local date : " + today);

    LocalDate previousYear = today.minus(1, ChronoUnit.YEARS);
    System.out.println("Date before 1 year : " + previousYear);

    LocalDate nextYear = today.plus(1, ChronoUnit.YEARS);
    System.out.println("Date after 1 year : " + nextYear);
}
```

### 10. 使用 Clock 时钟类 ###
Java8 增加了一个 `Clock` 时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到 `System.currentTimeInMillis()` 和 `TimeZone.getDefault()` 的地方都可用 `Clock` 替换。
```java
public static void clock() {
    // Returns the current time based on your system clock and set to UTC.
    Clock clock = Clock.systemUTC();
    System.out.println("Clock : " + clock);

    // Returns time based on system clock zone
    Clock defaultClock = Clock.systemDefaultZone();
    System.out.println("Clock : " + clock);
}
```

还可以针对 `Clock` 时钟做比较，像下面这个例子：
```java
public class MyClass {
    private Clock clock;  // dependency inject
    ...
    public void process(LocalDate eventDate) {
        if (eventDate.isBefore(LocalDate.now(clock)) {
            ...
        }
    }
}
```

这种方式处理[在不同时区下的日期](http://java67.blogspot.sg/2012/12/how-to-display-date-in-multiple-timezone-java.html)时会非常管用。

### 11. 判断日期是早于还是晚于另一个日期 ###
另一个工作中常见的操作就是如何判断给定的一个日期是大于某天还是小于某天？在 Java8 中，`LocalDate` 类有两类方法 `isBefore()` 和 `isAfter()` 用于比较日期。调用 `isBefore()` 方法时，如果给定日期小于当前日期则返回`true`。
```java
public static void isBeforeOrIsAfter() {
    LocalDate today = LocalDate.now();

    LocalDate tomorrow = LocalDate.of(2014, 1, 15);

    if (tomorrow.isAfter(today)) {
        System.out.println("Tomorrow comes after today");
    }

    LocalDate yesterday = today.minus(1, ChronoUnit.DAYS);

    if (yesterday.isBefore(today)) {
        System.out.println("Yesterday is day before today");
    }
}
```

在 Java8 中比较日期非常方便，不需要使用额外的 `Calendar` 类来做这些基础工作了。

### 12. 处理时区 ###
Java8 不仅分离了日期和时间，也把时区分离出来了。现在有一系列单独的类如 `ZoneId` 来处理特定时区，`ZoneDateTime` 类来表示某时区下的时间。这在 Java8 以前都是 [GregorianCalendar 类](http://javarevisited.blogspot.sg/2013/02/convert-xmlgregoriancalendar-to-date-xmlgregoriancalendar-java-example-tutorial.html)来做的。下面这个例子展示了如何把本时区的时间转换成另一个时区的时间。
```java
public static void getZoneTime() {
    // Date and time with timezone in Java 8
    ZoneId america = ZoneId.of("America/New_York");
    LocalDateTime localtDateAndTime = LocalDateTime.now();
    ZonedDateTime dateAndTimeInNewYork = ZonedDateTime.of(localtDateAndTime, america);
    System.out.println("Current date and time in a particular timezone : " + dateAndTimeInNewYork);
}
```

和以前[使用 GMT 的方式转换本地时间](http://javarevisited.blogspot.sg/2012/04/how-to-convert-local-time-to-gmt-in.html)对比一下。注意，在 Java8 以前，一定要牢牢记住时区的名称，不然就会抛出下面的异常：
```java
Exception in thread "main" java.time.zone.ZoneRulesException: Unknown time-zone ID: ASIA/Tokyo
        at java.time.zone.ZoneRulesProvider.getProvider(ZoneRulesProvider.java:272)
        at java.time.zone.ZoneRulesProvider.getRules(ZoneRulesProvider.java:227)
        at java.time.ZoneRegion.ofId(ZoneRegion.java:120)
        at java.time.ZoneId.of(ZoneId.java:403)
        at java.time.ZoneId.of(ZoneId.java:351)
```

### 13. 表示固定日期 ###
与 `MonthDay` 检查重复事件的例子相似，`YearMonth` 是另一个组合类，用于表示信用卡到期日、FD 到期日、期货期权到期日等。还可以用这个类得到当月共有多少天，`YearMonth` 实例的 `lengthOfMonth()` 方法可以返回当月的天数，在判断2月有28天还是29天时非常有用。
```java
public static void checkCardExpiry() {
    YearMonth currentYearMonth = YearMonth.now();
    System.out.printf("Days in month year %s: %d%n", currentYearMonth, currentYearMonth.lengthOfMonth());
    YearMonth creditCardExpiry = YearMonth.of(2018, Month.FEBRUARY);
    System.out.printf("Your credit card expires on %s %n", creditCardExpiry);
}
```

根据上述数据，你可以提醒客户信用卡快要到期了，个人认为这个类非常有用。

### 14. 检查闰年 ###
`LocalDate` 类有一个很实用的方法 `isLeapYear()` 判断该实例是否是一个闰年，如果你还是想重新发明轮子，这有一个代码示例，[纯 Java 逻辑编写的判断闰年](http://java67.blogspot.sg/2012/12/how-to-check-leap-year-in-java-program.html)的程序。
```java
public static void isLeapYear() {
    LocalDate today = LocalDate.now();
    int year = today.getYear();
    if (today.isLeapYear()) {
        System.out.printf("Year %d is a Leap year %n", year);
    } else {
        System.out.printf("Year %d is not a Leap year %n", year);
    }
}
```

你可以多写几个日期来验证是否是闰年，最好是写 `JUnit` 单元测试做判断。

### 15. 计算两个日期之间的天数和月数 ###
有一个常见日期操作是计算两个日期之间的天数、周数或月数。在 Java8 中可以用 `java.time.Period` 类来做计算。下面这个例子中，我们计算了当天和将来某一天之间的月数。
```java
public static void calcDateBetweenDays() {
    LocalDate today = LocalDate.now();
    LocalDate java8Release = LocalDate.of(2014, Month.DECEMBER, 14);
    Period periodToNextJavaRelease = Period.between(java8Release, today);
    System.out.println("Months left between today and Java 8 release : " + periodToNextJavaRelease.getMonths());
}
```

从上面可以看到现在是一月，Java8 的中计算的当前日期是4月18日，中间相隔3天。

### 16. 包含时差信息的日期和时间 ###
在 Java8 中，`ZoneOffset` 类用来表示时区，举例来说印度与 GMT 或 UTC 标准时区相差`+05:30`，可以通过 `ZoneOffset.of()` 静态方法来获取对应的时区。一旦得到了时差就可以通过传入 `LocalDateTime` 和 `ZoneOffset` 来创建一个 `OffSetDateTime` 对象。
```java
public static void zoneOffset() {
    LocalDateTime datetime = LocalDateTime.of(2014, Month.JANUARY, 14, 19, 30);
    ZoneOffset offset = ZoneOffset.of("+05:30");
    OffsetDateTime date = OffsetDateTime.of(datetime, offset);
    System.out.println("Date and Time with timezone offset in Java : " + date);
}
```

现在的时间信息里已经包含了时区信息了。注意：`OffSetDateTime` 是对计算机友好的，`ZoneDateTime` 则对人更友好。

### 17. 获取当前时间戳 ###
如果你还记得 Java8 以前是如何获得当前时间戳，那么现在你终于解脱了。`Instant` 类有一个静态工厂方法 `now()` 会返回当前的时间戳，如下所示：
```java
public static void getTimestamp() {
    Instant timestamp = Instant.now();
    System.out.println("What is value of this instant " + timestamp);
}
```

时间戳信息里同时包含了日期和时间，这和 `java.util.Date` 很像。实际上 `Instant` 类确实等同于 Java8 之前的 `Date` 类，你可以使用 `Date` 类和 `Instant` 类各自的转换方法互相转换，例如：`Date.from(Instant)` 将 `Instant` 转换成 `java.util.Date`，`Date.toInstant()` 则是将 `Date` 类转换成 `Instant` 类。

### 18. 使用预定义的格式化工具去解析或格式化日期 ###
在 Java8 以前的世界里，日期和时间的格式化非常诡异，唯一的帮助类 `SimpleDateFormat` 也是非线程安全的，而且用作局部变量解析和格式化日期时显得很笨重。幸好线程局部变量能使它在多线程环境中变得可用，不过这都是过去时了。Java8 引入了全新的日期时间格式工具，线程安全而且使用方便。它自带了一些常用的内置格式化工具。下面这个例子使用了 `BASIC_ISO_DATE` 格式化工具将2016年4月18日格式化成20160418。
```java
public static void formateDateWithPredefined() {
    String dayAfterTommorrow = "20140116";
    LocalDate formatted = LocalDate.parse(dayAfterTommorrow, DateTimeFormatter.BASIC_ISO_DATE);
    System.out.printf("Date generated from String %s is %s %n", dayAfterTommorrow, formatted);
}
```

很明显的看出得到的日期和给出的日期是同一天，但是格式不同。

### 19. 使用自定义格式化工具解析日期 ###
上个例子使用了[Java 内置的格式化工具](http://java67.blogspot.sg/2014/12/string-to-date-example-in-java-multithreading.html)去解析日期字符串。尽管内置格式化工具很好用，有时还是需要定义特定的日期格式，下面这个例子展示了如何创建自定义日期格式化工具。例子中的日期格式是“MMM dd yyyy”。可以调用`DateTimeFormatter`的`ofPattern()`静态方法并传入任意格式返回其实例，格式中的字符和以前代表的一样，`M` 代表月，`m` 代表分。如果格式不规范会抛出 `DateTimeParseException` 异常，不过如果只是把M写成m这种逻辑错误是不会抛异常的。
```java
public static void formateDateWithCustom() {
    String goodFriday = "Apr 18 2014";
    try {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMM dd yyyy", Locale.US);
        LocalDate holiday = LocalDate.parse(goodFriday, formatter);
        System.out.printf("Successfully parsed String %s, date is %s%n", goodFriday, holiday);
    } catch (DateTimeParseException ex) {
        System.out.printf("%s is not parsable!%n", goodFriday);
        ex.printStackTrace();
    }
}
```

日期值与传入的字符串是匹配的，只是格式不同而已。

### 20. 把日期转换成字符串 ###
上两个例子都用到了 `DateTimeFormatter` 类，主要是从字符串解析日期。现在我们反过来，把 `LocalDateTime` 日期实例转换成特定格式的字符串。这是迄今为止[Java 日期转字符串最为简单的方式](http://java67.blogspot.sg/2013/01/how-to-format-date-in-java-simpledateformat-example.html)了。下面的例子将返回一个代表日期的格式化字符串。和前面类似，还是需要创建 `DateTimeFormatter` 实例并传入格式，但这回调用的是 `format()` 方法，而非 `parse()` 方法。这个方法会把传入的日期转化成指定格式的字符串。
```java
public static void convertDateToString() {
    LocalDateTime arrivalDate = LocalDateTime.now();
    try {
        DateTimeFormatter format = DateTimeFormatter.ofPattern("MMM dd yyyy  hh:mm a", Locale.US);
        String landing = arrivalDate.format(format);
        System.out.printf("Arriving at :  %s %n", landing);
    } catch (DateTimeException ex) {
        System.out.printf("%s can't be formatted!%n", arrivalDate);
        ex.printStackTrace();
    }
}
```

当前时间被指定的“MMM dd yyyy hh:mm a”格式格式化，格式包含3个代表月的字符串，时间后面带有AM和PM标记。

## Java8 日期时间 API 总结 ##
通过这些例子，你肯定已经掌握了 Java8 日期时间 API 的新知识点。现在我们来回顾一下这个优雅 API 的使用要点：
1. 提供了 `javax.time.ZoneId` 获取时区。
2. 提供了 `LocalDate` 和 `LocalTime` 类。
3. Java8 的所有日期和时间 `API` 都是不可变类并且线程安全，而现有的 `Date` 和 `Calendar` API中的 `java.util.Date` 和 `SimpleDateFormat` 是非线程安全的。
4. 主包是 `java.time`，包含了表示日期、时间、时间间隔的一些类。里面有两个子包`java.time.format`用于格式化， `java.time.temporal`用于更底层的操作。
5. 时区代表了地球上某个区域内普遍使用的标准时间。每个时区都有一个代号，格式通常由区域/城市构成(Asia/Tokyo)，在加上与格林威治或 `UTC` 的时差。例如：东京的时差是+09:00。
6. `OffsetDateTime` 类实际上组合了 `LocalDateTime` 类和 `ZoneOffset` 类。用来表示包含和格林威治或 UTC 时差的完整日期（年、月、日）和时间（时、分、秒、纳秒）信息。
7. `DateTimeFormatter` 类用来格式化和解析时间。与 `SimpleDateFormat` 不同，这个类不可变并且线程安全，需要时可以给静态常量赋值。`DateTimeFormatter` 类提供了大量的内置格式化工具，同时也允许你自定义。在转换方面也提供了 `parse()` 将字符串解析成日期，如果解析出错会抛出 `DateTimeParseException`。`DateTimeFormatter` 类同时还有 `format()` 用来格式化日期，如果出错会抛出 `DateTimeException` 异常。
8. 再补充一点，日期格式“MMM d yyyy”和“MMM dd yyyy”有一些微妙的不同，第一个格式可以解析“Jan 2 2014”和“Jan 14 2014”，而第二个在解析“Jan 2 2014”就会抛异常，因为第二个格式里要求日必须是两位的。如果想修正，你必须在日期只有个位数时在前面补零，就是说“Jan 2 2014”应该写成 “Jan 02 2014”。

如何使用 Java8 的全新日期时间 API 就介绍到这了。这些简单的例子对帮助理解新 API 非常有用。由于这些例子都基于真实任务，在做 Java 日期编程时就不用再东张西望了。我们学会了如何创建并操作日期实例，学习了纯日期、以及包含时间信息和时差信息的日期、学会了怎样计算两个日期的间隔，这些在计算当天与某个特定日期间隔的例子中都有所展示。 我们还学到了在Java8中如何线程安全地解析和格式化日期，不用再使用蹩脚的线程局部变量技巧，也不用依赖`Joda Time`第三方库。新`API`可以作为处理日期时间操作的标准。

## Java8 其它教程 ##
如果你喜欢这个教程并希望看到更多关于 Java 8 的教程，下面这些精彩的文章都值得一看：
 - 如何在 Java8 中用一行代码搞定文件读取？([示例](http://javarevisited.blogspot.sg/2015/02/how-to-read-file-in-one-line-java-8.html))
 - 学习 Java8 的十大教程（[教程](http://java67.blogspot.sg/2014/09/top-10-java-8-tutorials-best-of-lot.html)）
 - 免费的 Java8 教程和图书 （[资源](http://javarevisited.blogspot.sg/2013/11/java-8-tutorials-resources-and-examples-lambda-expression-stream-api-functional-interfaces.html)）
 - Java 8 `Comparator` 例子 （[示例](http://java67.blogspot.com/2014/11/java-8-comparator-example-using-lambda-expression.html)）
 - 如何使用 Java8 的 `Map` 函数（[示例](http://java67.blogspot.sg/2015/01/java-8-map-function-examples.html)）
 - 你准备好学习 Java8 的认证了吗 （[更多](http://javarevisited.blogspot.sg/2014/09/latest-OCPJP-exam-java-8-certification-oracle-java-se-8.html)）
 - 如何使用 Java8 的默认方法。（[看这里](http://javarevisited.blogspot.sg/2014/07/default-defender-or-extension-method-of-Java8-example-tutorial.html)）
 - 开始 Java8 之前需要温习的十个`Java 7` 特性（[更多](http://javarevisited.blogspot.sg/2014/04/10-jdk-7-features-to-revisit-before-you.html)）
 - Java8 学习 `Stream API` 十例（[示例](http://javarevisited.blogspot.sg/2014/03/2-examples-of-streams-with-Java8-collections.html)）
 - 如何在匿名类中使用 `Lambda` 表达式（[答案](http://javarevisited.blogspot.sg/2015/01/how-to-use-lambda-expression-in-place-anonymous-class-java8.html)）
 - 如何使用 Java8 的 `Predicates` 类过滤 `Collection`？（[答案](http://javarevisited.blogspot.sg/2015/02/how-to-filter-collections-in-java-8.html)）
 - Java 中如何随即访问文件？（[答案](http://javarevisited.blogspot.sg/2015/02/randomaccessfile-example-in-java-read-write-String.html)）

## 致谢 ##
[Java 8 - 20 Examples of Date and Time API](http://javarevisited.blogspot.sg/2015/03/20-examples-of-date-and-time-api-from-Java8.html)

