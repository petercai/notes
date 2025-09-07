---
title: 生产环境BigDecimal用错了
source: https://juejin.cn/post/7445625290434412581
author: 
published: 2024-12-07
created: 2024-12-14
description: 
tags:
  - clippings
---
## 前言

在日常开发中，很多小伙伴喜欢用 BigDecimal 来处理精确计算，比如钱、分数、比例啥的。

理论上，它比 double 或 float 更精确，但如果你用得不对，精度丢失的问题会让你哭晕在厕所。

今天我们就来聊聊 ，错误使用BigDecimal的6种场景，为什么会发生问题，以及怎么避免问题，希望对你会有所帮助。


## 1 直接用浮点数初始化

不少小伙伴习惯这样写：

```ini
BigDecimal num = new BigDecimal(0.1);
System.out.println(num); 
```

打印结果：0.1000000000000000055511151231257827021181583404541015625

并非打印的：0.1

问题出在哪？

这不是 BigDecimal 的问题，而是浮点数本身的“锅”。

在Java中，double的精度有限的，0.1 转换成二进制是个无限循环小数，直接传进去会带上误差。

正确姿势是传字符串：

```ini
BigDecimal num = new BigDecimal("0.1");
System.out.println(num); 
```

打印结果：0.1，是正确的。

注意：<span style="background:#fff88f">永远不要用 BigDecimal(double) 构造函数，用字符串或整数更靠谱。也可以使用BigDecimal.valueOf()函数。</span>

## 2 加减乘除时不设精度

有些小伙伴做加减乘除的时候，直接写：

```ini
BigDecimal a = new BigDecimal("1.03");
BigDecimal b = new BigDecimal("0.42");
//减法
BigDecimal result = a.subtract(b);
System.out.println(result); 
```

打印结果：：0.61，没问题。

但问题在 除法 时：

```ini
BigDecimal c = new BigDecimal("10");
BigDecimal d = new BigDecimal("3");
BigDecimal result = c.divide(d); 
```

运行直接炸了：java.lang.ArithmeticException: Non-terminating decimal expansion

报错的根本原因：10/3 是无限小数，BigDecimal 默认不保留小数点后面，精度溢出。

那么，我们要如何优化呢？

答：加一个 MathContext 或指定精度。

例如：

```ini
BigDecimal result = c.divide(d, 2, RoundingMode.HALF_UP);
System.out.println(result); 
```

打印结果：3.33，可以正常运行。

因此，我们需要注意，<span style="background:#fff88f">在BigDecimal 做除法时 ，必须指定精度</span>。

## 3 用 equals 判断相等

BigDecimal 的 equals 会比较 值和精度，这坑了不少人：

```ini
BigDecimal x = new BigDecimal("1.0");
BigDecimal y = new BigDecimal("1.00");

System.out.println(x.equals(y)); 
```

打印结果：false。

尽管 1.0 和 1.00 的数值相等，但精度不一样，equals 判定为不同。

优化方法，用 compareTo 比较数值：

例如：

```csharp
System.out.println(x.compareTo(y) == 0); 
```

打印结果：true

需要特别注意的地方是：我们在判断两个BigDecimal对象是否相等时，应该用 compareTo方法，别用 equals方法。

## 4 使用 scale 时忽视实际含义

有些小伙伴搞不清 scale（小数位数）和 precision（总位数）的区别，直接写：

```ini
BigDecimal num = new BigDecimal("123.4500");
System.out.println(num.scale()); 
```

打印结果：4

但如果你写成下面这样的：

```ini
BigDecimal stripped = num.stripTrailingZeros();
System.out.println(stripped.scale()); 
```

打印结果却是：2

scale 会发生变化，搞不好会影响后续计算。

那么，我们要如何优化方法呢？

答：明确 scale 的含义。

如果要固定小数位，使用 setScale：

```ini
BigDecimal fixed = num.setScale(2, RoundingMode.HALF_UP);
System.out.println(fixed); 
```

打印结果：123.45。

我们不要混淆 scale 和 precision，必要时显式设置小数位数。

最近就业形势比较困难，为了感谢各位小伙伴对苏三一直以来的支持，我特地创建了一些工作内推群， 看看能不能帮助到大家。

你可以在群里发布招聘信息，也可以内推工作，也可以在群里投递简历找工作，也可以在群里交流面试或者工作的话题。

添加苏三的**私人微信**：su\_san\_java，备注：**掘金+所在城市**，即可加入。

## 5 忽略不可变性

BigDecimal 是不可变的，但有些小伙伴会这样写：

```ini
BigDecimal sum = new BigDecimal("0");
for (int i = 0; i < 5; i++) {
    sum.add(new BigDecimal("1"));
}
```

打印结果：0

问题原因是 add 方法不会改变原对象，而是返回一个新的 BigDecimal 实例。

那么，我们要如何优化呢？

答：用变量接住返回值。

```ini
BigDecimal sum = new BigDecimal("0");
for (int i = 0; i < 5; i++) {
    sum = sum.add(new BigDecimal("1"));
}
System.out.println(sum); 
```

打印结果是：5

BigDecimal 操作后需要接住新实例。

## 6 忽视性能问题

BigDecimal 是很精确，但也很慢。

如果大量计算时用 BigDecimal，会拖累性能，比如计算利息：

```ini
BigDecimal principal = new BigDecimal("10000");
BigDecimal rate = new BigDecimal("0.05");
BigDecimal interest = principal.multiply(rate);
```

一个循环里搞上百万次，性能直接拉垮。

那么，这种情况我们又该如何优化呢？

答：能用整数就用整数（比如分代替元）。

批量计算时，用 double 计算，结果最后转换成 BigDecimal。

```ini
double principal = 10000;
double rate = 0.05;
BigDecimal interest = BigDecimal.valueOf(principal * rate);
System.out.println(interest); 
```

打印结果：500.00

参与大批量计算时，两个BigDecimal对象直接计算会比较慢，尽量少用，能优化的地方别放过。


