# Java中的BigDecimal比较大于小于等于，四舍五入保留几位（setScale方法详解），加减乘除取余_Java_共饮一杯无_InfoQ写作社区
本文主要讲解 BigDecimal 的比较运算，保留精度和取整和基础运算,BigDecimal 与其他数据类型转换。

比较运算
----

比较 num1 是否大于 num2

```
public static boolean gt(@NotNull BigDecimal num1, BigDecimal num2) {
        return num1.compareTo(num2) > 0;
    }
```

比较 num1 是否小于 num2

```
public static boolean lt(@NotNull BigDecimal num1, BigDecimal num2) {
        return num1.compareTo(num2) < 0;
    }
```

比较 num1 是否大于等于 num2

```
public static boolean ge(@NotNull BigDecimal num1, BigDecimal num2) {
        return num1.compareTo(num2) >= 0;
    }
```

比较 num1 是否小于等于 num2

```
public static boolean le(@NotNull BigDecimal num1, BigDecimal num2) {
        return num1.compareTo(num2) <= 0;
    }
```

比较 num1 是否等于 num2

```
public static boolean eq(@NotNull BigDecimal num1, BigDecimal num2) {
        return num1.compareTo(num2) == 0;
    }
```

保留精度及取整
-------

核心主要是 `setScale(int newScale, int roundingMode)` 方法。主要是两个参数：

*   `newScale`为小数位数；
    
*   `roundingMode`为取舍模式；
    

### 取整（保留 0 位小数）

```
* 取整返回int 类型
     * @param num1
     * @param roundingMode
     * @return
     */
    public static int intValue(@NotNull BigDecimal num1,int roundingMode) {
        return num1.setScale(SCALA_ZERO, roundingMode).intValue();
    }
```

取整时`setScale(int newScale, int roundingMode)`第一个参数为 0，第二个为取舍模式。各个 roundingMode 详解如下：

*   `ROUND_UP`：正数时，舍弃小数后（整数部分）加 1，比如 100.39 结果为 100。负数时，舍弃小数后（整数部分）减去 1，-100.39 结果为 -101。
    
*   `ROUND_DOWN`：直接舍弃小数。
    
*   `ROUND_CEILING`：如果 BigDecimal 是正的，则做 ROUND\_UP 操作；如果为负，则做 ROUND\_DOWN 操作 （取附近较大的整数）。
    
*   `ROUND_FLOOR`: 如果 BigDecimal 是正的，则做 ROUND\_DOWN 操作；如果为负，则做 ROUND\_UP 操作（取附近较小的整数）。
    
*   `ROUND_HALF_UP`：四舍五入（取更近的整数）。
    
*   `ROUND_HALF_DOWN`：同 ROUND\_HALF\_UP 差别仅在于 0.5 时会向下取整。
    
*   `ROUND_HALF_EVEN`：取最近的偶数。
    
*   `ROUND_UNNECESSARY`：不需要取整，如果存在小数位，就抛 ArithmeticException 异常。
    

### 保留精度

四舍五入保留几位小数

```
* 四舍五入保留几位小数
     * @param scala 保留几位
     * @param num1 对应数值
     * @return
     */
    public static float halfUpValue(@NotNull BigDecimal num1,int scala) {
        return num1.setScale(scala, BigDecimal.ROUND_HALF_UP).floatValue();
    }
```

指定取舍规则，保留几位小数

```
* 指定取舍规则，保留几位小数
     * @param scala 保留几位
     * @param num1 对应数值
     * @param roundingMode 取舍规则
     * @return
     */
    public static BigDecimal roundingModeValue(@NotNull BigDecimal num1,int scala,int roundingMode) {
        
         * setScale(1,BigDecimal.ROUND_DOWN)直接删除多余的小数位，如2.35会变成2.3
         * setScale(1,BigDecimal.ROUND_UP)进位处理，2.35变成2.4
         * setScale(1,BigDecimal.ROUND_HALF_UP)四舍五入，2.35变成2.4
         * setScale(1,BigDecimal.ROUND_HALF_DOWN)四舍五入，2.35变成2.3，如果是5则向下舍
         */
        return num1.setScale(scala, roundingMode);
    }
```

基础运算
----

主要是以下方法：加：`BigDecimal add(BigDecimal augend)`减：`BigDecimal subtract(BigDecimal subtrahend)`乘：`BigDecimal multiply(BigDecimal multiplicand)`除：`BigDecimal divide(BigDecimal divisor)`取余：`BigDecimal[] divideAndRemainder(BigDecimal divisor)`，返回一个 BigDecimal 数组，返回数组中包含两个元素，第一个元素为两数相除的商，第二个元素为余数。

BigDecimal 与其他数据类型转换
--------------------

四舍五入保留几位小数返回字符串

```
* 四舍五入保留几位小数返回字符串
     * @param tScala 保留几位
     * @param num1 对应数值
     * @param tRoundingMode 舍入类型
     * @return
     */
    public static String toPlainString(@NotNull BigDecimal num1, int tScala, int tRoundingMode) {
        return num1.setScale(tScala, tRoundingMode).toPlainString();
    }
```

四舍五入保留两位小数返回 double 类型

```
* 四舍五入保留两位小数返回double类型
     * @param num1
     * @return
     */
    public static double doubleValue(@NotNull BigDecimal num1) {
        return num1.setScale(SCALA_TWO, BigDecimal.ROUND_HALF_UP).doubleValue();
    }
```

其他转换类似：floatValue()、 longValue() 、intValue()...。

> 本文内容到此结束了，
> 
> 如有收获欢迎点赞👍收藏💖关注✔️，您的鼓励是我最大的动力。
> 
> 如有错误❌疑问💬欢迎各位大佬指出。
> 
> **主页**：[共饮一杯无的博客汇总👨‍💻](https://www.infoq.cn/u/zhanjq/publish)  
> 
> **保持热爱，奔赴下一场山海**。🏃🏃🏃