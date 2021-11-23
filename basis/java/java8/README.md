https://blog.csdn.net/qq_28410283/article/details/80704487

## 1. Lambda表达式
## 2. 函数式接口
## 3. 流Stream
## 4. 组合式异步编程CompletableFuture
## 5. Optional静态类

几乎所有的Java程序员碰到NullPointerException时的第一冲动就是添加一个if语句，在调用方法使用该变量之前检查它的值是否为null，快速地搞定问题。

Java 8引入了一个名为java.util.Optional<T>的新的类。变量存在时，Optional类只是对类简单封装。变量不存在时，缺失的值会被建模成一个“空”的Optional对象，由方法Optional.empty()返回。Optional是一个容器对象，它可能包含空值，也可能包含非空值。当属性value被设置时，isPesent()方法将返回true，并且get()方法将返回这个值。

该类支持泛型，即其属性value可以是任何对象的实例。

### Optional类的方法

| **序号** | **方法**                                                     | **方法说明**                                                 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **1**    | `private Optional()`                                         | 无参构造，构造一个空Optional                                 |
| **2**    | `private Optional(T value)`                                  | 根据传入的非空value构建Optional                              |
| **3**    | `public static<T> Optional<T> empty()`                       | 返回一个空的Optional，该实例的value为空                      |
| **4**    | `public static <T> Optional<T> of(T value)`                  | 根据传入的非空value构建Optional，与Optional(T value)方法作用相同 |
| **5**    | `public static <T> Optional<T> ofNullable(T value)`          | 与of(T value)方法不同的是，ofNullable(T value)允许你传入一个空的value，当传入的是空值时其创建一个空Optional，当传入的value非空时，与of()作用相同 |
| **6**    | `public T get()`                                             | 返回Optional的值，如果容器为空，则抛出NoSuchElementException异常 |
| **7**    | `public boolean isPresent()`                                 | 判断当家Optional是否已设置了值                               |
| **8**    | `public void ifPresent(Consumer<? super T> consumer)`        | 判断当家Optional是否已设置了值，如果有值，则调用Consumer函数式接口进行处理 |
| **9**    | `public Optional<T> filter(Predicate<? super T> predicate)`  | 如果设置了值，且满足Predicate的判断条件，则返回该Optional，否则返回一个空的Optional |
| **10**   | `public<U> Optional<U> map(Function<? super T, ? extends U> mapper)` | 如果Optional设置了value，则调用Function对值进行处理，并返回包含处理后值的Optional，否则返回空Optional |
| **11**   | `public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)` | 与map()方法类型，不同的是它的mapper结果已经是一个Optional，不需要再对结果进行包装 |
| **12**   | `public T orElse(T other)`                                   | 如果Optional值不为空，则返回该值，否则返回other              |
| **13**   | `public T orElseGet(Supplier<? extends T> other)`            | 如果Optional值不为空，则返回该值，否则根据other另外生成一个  |
| **14**   | `public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)throws X` | 如果Optional值不为空，则返回该值，否则通过supplier抛出一个异常 |

### 使用Optional避免空指针

在我们日常开发过程中不可避免地会遇到空指针问题，在以前，出现空指针问题，我们通常需要进行调试等方式才能最终定位到具体位置，尤其是在分布式系统服务之间的调用，问题更难定位。在使用Optional后，我们可以将接受到的参数对象进行包装，比如，订单服务要调用商品服务的一个接口，并将商品信息通过参数传入，这时候，传入的商品参数可能直接传入的就是null，这时，商品方法可以使用Optional.of(T)对传入的对象进行包装，如果T为空，则会直接抛出空指针异常，我们看到异常信息就能立即知道发生空指针的原因是参数T为空；或者，当传入的参数为空时，我们可以使用Optional.orElse()或Optional.orElseGet()方法生成一个默认的实例，再进行后续的操作。

下面再看个具体例子：在User类中有个Address类，在Address类中有个Street类,Street类中有streetName属性，现在的需求是：根据传入的User实例，获取对应的streetName，如果User为null或Address为null或Street为null，返回“nothing found”，否则返回对应的streetName。

```java
@Data
public class User {
	private String name;
	private Integer age;
	private Address address;
}

@Data
public class Address {
	private Street street;
}

@Data
public class Street {
	private String streetName;
	private Integer streetNo;
}
```

```java
public String getUserSteetName(User user) {

		if (null != user) {
			Address address = user.getAddress();
			if (null != address) {
				Street street = address.getStreet();
				if (null != street) {
					return street.getStreetName();
				}
			}
		}

		return "nothing found";
	}
```

```java
public String getUserSteetNameBetter(User user) {

		Optional<User> userOptional = Optional.ofNullable(user);
		final String streetName = Optional.ofNullable(
				Optional.ofNullable(userOptional
				.orElse(new User()).getAddress())
				.orElse(new Address()).getStreet())
				.orElse(new Street()).getStreetName();
		return StringUtils.isEmpty(streetName) ? "nothing found" : streetName;
	}
```



## 6. 新的日期和时间API

java8引入了一套全新的时间日期API，新的时间及日期API位于java.time包中整合了很多Joda-Time的特性，java.time包中的类是不可变且线程安全的。

下面是一些关键类

- Instant——它代表的是时间戳
- LocalDate——不包含具体时间的日期，比如2014-01-14。它可以用来存储生日，周年纪念日，入职日期等。
- LocalTime——它代表的是不含日期的时间
- LocalDateTime——它包含了日期及时间，不过还是没有偏移信息或者说时区。
- ZonedDateTime——这是一个包含时区的完整的日期时间，偏移量是以UTC/格林威治时间为基准的。
- Duration —— Duration类表示秒或纳秒时间间隔，适合处理较短的时间，需要更高的精确性。我们能使用between()方法比较两个瞬间的差。
- Period —— 计算两个日期之间包含多少天、周、月、年。

```jav
// 获取当前时间戳
// Instant类由一个静态的工厂方法now()可以返回当前时间戳
Instant timestamp = Instant.now();
log.info("当前的时间戳是：" + timestamp);

// 获取当前时间毫秒数
Date date = new Date();
log.info("" + date.getTime());
log.info("" + Calendar.getInstance().getTimeInMillis());
Instant timestamp1 = Instant.now();
log.info("" + timestamp1.toEpochMilli());

// 获取当前日期
LocalDate today = LocalDate.now();
log.info("今天的日期：" + today);

// 获取当前的年月日
int year = today.getYear();
int month = today.getMonthValue();
int day = today.getDayOfMonth();
log.info("年：" + year +" 月："+ month +" 日："+ day);

// 获取某个特定的日期
// 可以创建出任意一个日期，它接受年月日的参数，然后返回一个等价的LocalDate实例。
LocalDate dateOfBirth = LocalDate.of(2020, 11, 23);
log.info("生日：" + dateOfBirth);

// 检查两个日期是否相等
// LocalDate重写了equals方法来进行日期的比较
log.info("是否相等：" + dateOfBirth.equals(today));

// 使用MonthDay类。这个类由月日组合，不包含年信息，可以用来代表每年重复出现的一些日期或其他组合
MonthDay a = MonthDay.of(dateOfBirth.getMonth(),dateOfBirth.getDayOfMonth());
MonthDay b = MonthDay.from(today);
log.info("是否相等：" + a.equals(b));

// 如何判断某个日期在另一个日期的前面还是后面或者相等，
// 在java8中，LocalDate类中使用isBefore()、isAfter()、equals()方法来比较两个日期。
// 如果调用方法的那个日期比给定的日期要早的话，isBefore()方法会返回true。
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
log.info("是否在之前：" + tomorrow.isBefore(today));
log.info("是否在之后：" + tomorrow.isAfter(today));


// 获取1周后的日期
LocalDate oneToday = today.plus(1, ChronoUnit.WEEKS);
log.info("一周后的日期：" + oneToday);

// 获取1年前后的日期
LocalDate preYear = today.minus(1, ChronoUnit.YEARS);
log.info("一年前的日期：" + preYear);
LocalDate nextYear = today.plus(1, ChronoUnit.YEARS);
log.info("一年后的日期：" + nextYear);

// 获取当前时间 这里用的是LocalTime类，默认的格式是hh:mm:ss:nnn
LocalTime now = LocalTime.now();
log.info("现在的时间：" + now);

// 增加时间里面的小时数
LocalTime two = now.plusHours(2);
log.info("两小时后的时间：" + two);

// java8自带了Clock类，可以用来获取某个时区下（所以对时区是敏感的）当前的瞬时时间、日期。
// 用来代替System.currentTimelnMillis()与TimeZone.getDefault()方法
Clock clock = Clock.systemUTC();
log.info("clock：" + clock);
Clock clock1 = Clock.systemDefaultZone();
log.info("clock：" + clock1);

// java8中不仅将日期和时间进行了分离，同时还有时区。
// 比如ZonId代表的是某个特定时区，ZonedDateTime代表带时区的时间，等同于以前的GregorianCalendar类。
// 使用该类，可以将本地时间转换成另一个时区中的对应时间。
LocalDateTime localDateTime = LocalDateTime.now();
ZoneId zone = ZoneId.of(ZoneId.SHORT_IDS.get("ACT"));
ZonedDateTime dateAndTimeInNewYork = ZonedDateTime.of(localDateTime, zone);
log.info("现在时区的时间和在特定时区的时间：" + dateAndTimeInNewYork);

// 检查闰年
// LocalDate类有一个isLeapYear()方法来返回当前LocalDate对应的那年是否是闰年
log.info("{} 是否是闰年：{}", today,today.isLeapYear());

// 两个日期之间包含多少天，多少月
LocalDate date1 = LocalDate.of(2021, 11, 12);
LocalDate date2 = LocalDate.of(2021, 11, 23);
Period dValue = Period.between(date1, date2);
log.info("日期{}和日期{}相差{}个月", date1, date2, dValue.getMonths());
log.info("日期{}和日期{}相差{}天", date1, date2, dValue.get(ChronoUnit.DAYS));

LocalDate startDate = LocalDate.of(2015, 2, 20);
LocalDate endDate = LocalDate.of(2017, 2, 15);
Period period = Period.between(startDate, endDate);
log.info("Years:" + period.getYears() + 
		  " months:" + period.getMonths() + 
		  " days:"+period.getDays());

// 取两个日期直接相差多少天
// 方法1
long daysBetween = ChronoUnit.DAYS.between(date1, date2);
log.info("日期{}和日期{}相差{}天", date1, date2, daysBetween);
// 方法2
long minusDay = date2.toEpochDay() - date1.toEpochDay();
log.info("日期{}和日期{}相差{}天", date1, date2, minusDay);
long minusDay2 = endDate.toEpochDay() - startDate.toEpochDay();
log.info("日期{}和日期{}相差{}天", startDate, endDate, minusDay2);

// 两个日期之间相差多少秒，多少纳秒
Instant start = Instant.parse("2017-10-03T09:15:30.00Z");
Instant end = Instant.parse("2017-10-03T10:16:30.00Z");
Duration duration = Duration.between(start, end);
log.info("日期{}和日期{}相差{}秒",start, end, duration.getSeconds());

// 如何在java8中使用预定义的格式器来对日期进行解析/格式化
// 在java8之前，时间日期的格式化非常麻烦，经常使用SimpleDateFormat来进行格式化，但是SimpleDateFormat并不是线程安全的。
// 在java8中，引入了一个全新的线程安全的日期与时间格式器。并且预定义好了格式。
String dateString = "20211123";
LocalDate formatted = LocalDate.parse(dateString, DateTimeFormatter.BASIC_ISO_DATE);
log.info("字符串{}格式化后的日期格式是{}", dateString, formatted);

// 使用自定义的格式器来解析日期
String holidayString = "2021-11-23 11:11:11";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime holiday = LocalDateTime.parse(holidayString, formatter);
log.info("字符串{}格式化后的日期格式是{}", holidayString, holiday);

// 对日期进行格式化，转换成字符串
LocalDateTime arriveDate = LocalDateTime.now();
DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String landing = arriveDate.format(formatter1);
log.info("时间{}格式化后的字符串是{}", arriveDate, landing);
```



### java8中日期与时间API的几个关键点

经过上面的例子，我们已经对java8的时间日期有了一定的了解，现在回顾一下

- 它提供了javax.time.ZoneId用来处理时区。
- 它提供了LocalDate与LocalTime类
- Java 8中新的时间与日期API中的所有类都是不可变且线程安全的，这与之前的Date与Calendar API中的恰好相反，那里面像java.util.Date以及SimpleDateFormat这些关键的类都不是线程安全的。
- 新的时间与日期API中很重要的一点是它定义清楚了基本的时间与日期的概念，比方说，瞬时时间，持续时间，日期，时间，时区以及时间段。它们都是基于ISO日历体系的。

### 每个Java开发人员都应该至少了解这套新的API中的这五个类：

- Instant 它代表的是时间戳，比如2016-04-14T14:20:13.592Z，这可以从java.time.Clock类中获取，像这样： Instant current = Clock.system(ZoneId.of("Asia/Tokyo")).instant();
- LocalDate 它表示的是不带时间的日期，比如2016-04-14。它可以用来存储生日，周年纪念日，入职日期等。
- LocalTime - 它表示的是不带日期的时间
- LocalDateTime - 它包含了时间与日期，不过没有带时区的偏移量
- ZonedDateTime - 这是一个带时区的完整时间，它根据UTC/格林威治时间来进行时区调整
- 这个库的主包是java.time，里面包含了代表日期，时间，瞬时以及持续时间的类。它有两个子package，一个是java.time.foramt，这个是什么用途就很明显了，还有一个是java.time.temporal，它能从更低层面对各个字段进行访问。
- 时区指的是地球上共享同一标准时间的地区。每个时区都有一个唯一标识符，同时还有一个地区/城市(Asia/Tokyo)的格式以及从格林威治时间开始的一个偏移时间。比如说，东京的偏移时间就是+09:00。
- OffsetDateTime类实际上包含了LocalDateTime与ZoneOffset。它用来表示一个包含格林威治时间偏移量（+/-小时：分，比如+06:00或者 -08：00）的完整的日期（年月日）及时间（时分秒，纳秒）。
- DateTimeFormatter类用于在Java中进行日期的格式化与解析。与SimpleDateFormat不同，它是不可变且线程安全的，如果需要的话，可以赋值给一个静态变量。DateTimeFormatter类提供了许多预定义的格式器，你也可以自定义自己想要的格式。当然了，根据约定，它还有一个parse()方法是用于将字符串转换成日期的，如果转换期间出现任何错误，它会抛出DateTimeParseException异常。类似的，DateFormatter类也有一个用于格式化日期的format()方法，它出错的话则会抛出DateTimeException异常。
- 再说一句，“MMM d yyyy”与“MMm dd yyyy”这两个日期格式也略有不同，前者能识别出"Jan 2 2014"与"Jan 14 2014"这两个串，而后者如果传进来的是"Jan 2 2014"则会报错，因为它期望月份处传进来的是两个字符。为了解决这个问题，在天为个位数的情况下，你得在前面补0，比如"Jan 2 2014"应该改为"Jan 02 2014"。