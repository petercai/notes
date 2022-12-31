# 日期工具类-操作字符串和Date、LocalDate互转，两个日期的时间差等_InfoQ写作社区
避免重复造轮子，相关方法基于 hutool 日期时间工具封装并做部分增强。需要先引入如下坐标

```
<dependency>
              <groupId>cn.hutool</groupId>
              <artifactId>hutool-all</artifactId>
              <version>${hutool.version}</version>
          </dependency>
```

### 字符串转 Date

```
Date dateTime = DateUtil.parseDate("2022-04-06");
    System.out.println(dateTime);
```

### Date 转字符串

```
String dateStr = DateUtil.date2Str(new Date());
System.out.println(dateStr);

String dateStr2 = DateUtil.date2Str("yyyy/MM/dd",new Date());
System.out.println(dateStr2);
```

### 字符串转 LocalDate

```
LocalDate localDate = DateUtil.parseLocalDate("2022-04-06");
System.out.println(localDate);
```

### Date 转 LocalDate

```
LocalDate localDate = DateUtil.date2LocalDate(new Date());
System.out.println(localDate);
```

### LocalDate 转字符串

```
String localDateStr = DateUtil.localDate2Str(LocalDate.now());
System.out.println(localDateStr);
```

### 两个日期的时间差

```
String beginDateStr = "2022-02-01 22:33:23";
Date beginDate = DateUtil.parse(beginDateStr);

String endDateStr = "2022-03-10 23:33:23";
Date endDate = DateUtil.parse(endDateStr);

long betweenDay = DateUtil.between(beginDate, endDate, DateUnit.DAY);
System.out.println(betweenDay);

String formatBetween = DateUtil.formatBetween(beginDate, endDate, BetweenFormater.Level.HOUR);
System.out.println(formatBetween);
```

### 一天的开始和结束时间

```
String dateStr = "2022-04-07 10:33:23";
Date date = DateUtil.parse(dateStr);


Date beginOfDay = DateUtil.beginOfDay(date);
System.out.println(beginOfDay);


Date endOfDay = DateUtil.endOfDay(date);
System.out.println(endOfDay);
```

### 工具类如下：

```
* <p>基于hutool的日期工具类增强</p>
 *
 * @author zjq
 * @date 2021/04/07
 */
public class DateUtil extends cn.hutool.core.date.DateUtil {

    private static final String[] PARSE_PATTERNS = {
            "yyyy-MM-dd", "yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd HH:mm", "yyyy-MM",
            "yyyy/MM/dd", "yyyy/MM/dd HH:mm:ss", "yyyy/MM/dd HH:mm", "yyyy/MM",
            "yyyy.MM.dd", "yyyy.MM.dd HH:mm:ss", "yyyy.MM.dd HH:mm", "yyyy.MM"};

    public static final DateTimeFormatter TIME_FORMATTER = DateTimeFormatter.ofPattern("HH:mm:ss");
    public static final DateTimeFormatter DATE_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    public static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    
     * 字符串转date类型
     *
     * @param dateStr
     * @return
     */
    public static Date parseDate(Object dateStr) {
        if (ObjectUtils.isNull(dateStr)) {
            return null;
        }
        if (dateStr instanceof Double) {
            return org.apache.poi.ss.usermodel.DateUtil.getJavaDate((Double) dateStr);
        }
        return parse(dateStr.toString(), PARSE_PATTERNS);
    }

    
     * Date类型转字符串
     *
     * @param date
     * @return yyyy-MM-dd HH:mm:ss
     */
    public static String date2Str(Date date) {
        return date2Str(null, date);
    }

    
     * Date类型转字符串
     *
     * @param format
     * @param date
     * @return
     */
    public static String date2Str(String format, Date date) {
        if (ObjectUtils.isNull(date)) {
            return null;
        }
        SimpleDateFormat dateFormat = StrUtil.isNotBlank(format) ?new SimpleDateFormat(format):
                new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        return dateFormat.format(date);
    }

    
     * 字符串转LocalTime类型
     *
     * @param dateStr
     * @return
     */
    public static LocalTime parseLocalTime(Object dateStr) {
        if (ObjectUtils.isNull(dateStr)) {
            return null;
        }
        if (dateStr instanceof Double) {
            return date2LocalTime(parseDate(dateStr));
        }
        return LocalTime.parse(dateStr.toString(), TIME_FORMATTER);
    }

    
     * Date转LocalTime
     *
     * @param date
     * @return
     */
    public static LocalTime date2LocalTime(Date date) {
        if (null == date) {
            return null;
        }
        return date.toInstant().atZone(ZoneId.systemDefault()).toLocalTime();
    }

    
     * LocalTime转Str
     *
     * @param localTime
     * @return HH:mm:ss
     */
    public static String localTime2Str(LocalTime localTime) {
        return localTime2Str(null, localTime);
    }

    
     * LocalTime转str
     *
     * @param format    格式
     * @param localTime
     * @return
     */
    public static String localTime2Str(String format, LocalTime localTime) {
        if (null == localTime) {
            return null;
        }
        DateTimeFormatter formatter = StrUtil.isBlank(format) ?
                TIME_FORMATTER : DateTimeFormatter.ofPattern(format);
        return localTime.format(formatter);
    }

    
     * 字符串转LocalDate类型
     *
     * @param dateStr
     * @return
     */
    public static LocalDate parseLocalDate(Object dateStr) {
        if (ObjectUtils.isNull(dateStr)) {
            return null;
        }
        if (dateStr instanceof Double) {
            return date2LocalDate(parseDate(dateStr));
        }
        return LocalDate.parse(dateStr.toString(), DATE_FORMATTER);
    }

    
     * Date转LocalDate
     *
     * @param date
     * @return
     */
    public static LocalDate date2LocalDate(Date date) {
        if (null == date) {
            return null;
        }
        return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
    }

    
     * LocalDate转Str
     *
     * @param localDate
     * @return
     */
    public static String localDate2Str(LocalDate localDate) {
        return localDate2Str(null, localDate);
    }

    
     * LocalDate转Str
     *
     * @param format    格式
     * @param localDate
     * @return
     */
    public static String localDate2Str(String format, LocalDate localDate) {
        if (null == localDate) {
            return null;
        }
        DateTimeFormatter formatter = StrUtil.isBlank(format) ?
                DATE_FORMATTER : DateTimeFormatter.ofPattern(format);
        return localDate.format(formatter);
    }

    
     * 字符串转LocalDateTime类型
     *
     * @param dateStr
     * @return
     */
    public static LocalDateTime parseLocalDateTime(Object dateStr) {
        if (ObjectUtils.isNull(dateStr)) {
            return null;
        }
        if (dateStr instanceof Double) {
            return date2LocalDateTime(parseDate(dateStr));
        }
        return LocalDateTime.parse(dateStr.toString(), DATETIME_FORMATTER);
    }

    
     * Date转LocalDateTime
     *
     * @param date
     * @return
     */
    public static LocalDateTime date2LocalDateTime(Date date) {
        if (null == date) {
            return null;
        }
        return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
    }

    
     * LocalDate转Str
     *
     * @param localDateTime
     * @return
     */
    public static String localDateTime2Str(LocalDateTime localDateTime) {
        return localDateTime2Str(null, localDateTime);
    }

    
     * LocalDate转Str
     *
     * @param format
     * @param localDateTime
     * @return
     */
    public static String localDateTime2Str(String format, LocalDateTime localDateTime) {
        if (null == localDateTime) {
            return null;
        }
        DateTimeFormatter formatter = StrUtil.isBlank(format) ?
                DATETIME_FORMATTER : DateTimeFormatter.ofPattern(format);
        return localDateTime.format(formatter);
    }
}
```

参考：[https://www.hutool.cn/docs/#/core/日期时间/日期时间工具-DateUtil](https://xie.infoq.cn/link?target=https%3A%2F%2Fwww.hutool.cn%2Fdocs%2F%23%2Fcore%2F%25E6%2597%25A5%25E6%259C%259F%25E6%2597%25B6%25E9%2597%25B4%2F%25E6%2597%25A5%25E6%259C%259F%25E6%2597%25B6%25E9%2597%25B4%25E5%25B7%25A5%25E5%2585%25B7-DateUtil)  

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃