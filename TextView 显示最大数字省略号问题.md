[TOC]

# Android 中限制 TextView 中的字符输入长度时，会有一些问题

## TextView有几种控制文本长度的方法

* android:maxWidth    控制View的长度来控制文本长度
* android:maxLength  控制字符的个数来控制文本长度
* android:maxEms      控制字符的长度来控制文本长度
* 在代码中对 text 进行截取，超出长度的部分替换成省略号

## ellipsize

### 常见属性如下：

* start：省略号在开头
* middle：省略号在中间
* end：省略号在结尾
* marquee：跑马灯显示

当想让 `TextView` 文本后边显示不下，需要显示省略号的时候

```xml
<TextView
          android:width="wrap_content"
          android:height="wrap_content"
          android:text="abcdefghijklmnopqrstuvwxyz"
          android:singleLine="true"
          android:ellipsize="end"
          android:maxEms="10"
          />
```

有如下几个条件：

1. TextView 的宽度不能是 `MATCH_PARENT”

2. TextView 的宽度必须小于字符串长度
3. 必须同时设置 `android:singleLine="true"` 或 `android:maxLine="1"`中的一个属性
4. 同时还要设置 `android:ellipsize="end"` 和 `android:maxEms="5"`两个属性

## maxLength

> maxLength 官方释义如下所示
>
> Set an input filter to constrain the text length to the specified number. 
>
> Must be an integer value, such as "`100`".

可限制 text 的长度为指定的数字，也就是直接限制了可以输入多少个字符串。个人理解是先对文本进行截取，再填入控件，所以不支持与 `ellipsize` 配合显示省略号的做法。使用 maxLength，汉字，英文字母，标点以及空格都占一位。

> 当 `android:maxLength` 与 `android:ellipsize` 属性同时使用时，不会有省略号效果

## maxEms

### 官方释义

```java
/**
 * Sets the width of the TextView to be at most {@code maxEms} wide.
 * <p>
 * This value is used for width calculation if LayoutParams does not force TextView to have an
 * exact width. Setting this value overrides previous maximum width configurations such as
 * {@link #setMaxWidth(int)} or {@link #setWidth(int)}.
 **/
```

等效于设置 `TextView` 的最大宽度

maxEms 支持省略号的缩进。
Ems 比较像一种单位或者权重，比如 maxEms 设为 10 时，大概可以展示 20 个英文字符，10 个中文字符（中英文不同的标点符号权重不同，比如英文逗号可以展示将近 50 个，但中文逗号只能展示 10 个，而英文的 * 号可以展示 20 个）。但实际上也并非一一对应的，下文为 maxEms 的具体定义，可知其实并不好控制

> `maxEms="10"` 限制 `TextView` 的最大宽度为 10 个大写 M 的字符宽度，同时也与字号大小有关。em 是一个印刷排版的单位，表示字宽的单位。 em 字面意思为：equal M（和 M 字符一致的宽度为一个单位）简称 em。ems 是 em 的复数表达。比如：`android:maxEms="5"`，我理解就是将 `TextView` 的最大宽度设为 5 个当前字号的字母 `M` 的宽度总和。（也有解释 em 为 max expanded memory standard 的缩写，表示一行的最大宽度）

# 参考文档

[浅谈 Android maxEms 属性](https://cloud.tencent.com/developer/article/1485243)