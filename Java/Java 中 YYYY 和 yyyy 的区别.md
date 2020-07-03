# Java 中 `YYYY` 和 `yyyy` 的区别

关于 Java 中 YYYY 和 yyyy 的区别，其实之前还没有特殊留意过，直到看到了一篇文章介绍

```kotlin
class TestJava {
    companion object {
        private val time = 1577721600000L
        @JvmStatic
        fun main(args: Array<String>) {
            val spf = SimpleDateFormat("YYYY-MM-dd")
            println(spf.format(time))
            spf.applyPattern("yyyy-MM-dd")
            println(spf.format(time))
        }
    }
}
```

结果如下

> 2020-12-31
> 2019-12-31
>
> Process finished with exit code 0

# 结论

1. 在JDK1.7国内，**只要跨年周包含次年的1月1日，那么这一周7个日期的Week year就都是次年的年份。**
2. 我们可以通过修改minimalDaysInFirstWeek和firstDayOfWeek，修改YYYY格式化后的值（即week year）

下边的截图说的很详细

![image-20200310160049414](https://tva1.sinaimg.cn/large/00831rSTly1gcow3tcrkdj30u011h47v.jpg)

# 参考文档

[每日一问 据说很多 app 在 2019 年最后一周都出现了日期上的 bug ?](https://www.wanandroid.com/wenda/show/11387)

[SimpleDateFormat](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html)

[GregorianCalendar WeekYear](https://docs.oracle.com/javase/7/docs/api/java/util/GregorianCalendar.html#week_year)

[JAVA中的SimpleDateFormat yyyy和YYYY的区别](https://blog.csdn.net/bewilderment/article/details/48391717)

[Java笔记之YYYY格式化日期](https://xiaoyong.ml/blog/posts/87a89a43/)