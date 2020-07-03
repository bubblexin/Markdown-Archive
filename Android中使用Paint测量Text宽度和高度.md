Android中，在自定义View中通过Canvas绘制文字时，经常需要测量文字的宽度和高度。这里记录下几种比较常用的方法，仅作备忘。

- Paint.measureText （测量文本的宽度）

```java
Paint paint = new Paint();
paint.setTextSize(size);
float strWidth = paint.measureText(str);123
```

- Paint.getTextBounds (获得文字所在矩形区域，可以得到宽高)

```java
Paint paint = new Paint();
Rect rect = new Rect();  
paint.getTextBounds(str, 0, str.length(), rect);  
int w = rect.width();  
int h = rect.height();
123456
```

- Paint.getTextWidths(获得每个字符的宽度)

```java
float width = 0;
int len = str.length();  
Paint paint = new Paint();
float[] widths = new float[len];  
paint.getTextWidths(str, widths);  
for (int i = 0; i < len; i++) {  
     width += widths[i];  
}12345678
```

- 通过Paint.FontMetrics or Paint.FontMetricsInt来获取高度
  这两者的含义相同，只不过精度不同，一个float、一个int。关于Paint.FontMetrics，可以参考[这里](http://blog.51cto.com/mikewang/871765)。

```java
Paint paint = new Paint();
paint.setTextSize(size);//设置字体大小
paint.setTypeface(Typeface.xx);//设置字体
FontMetrics fontMetrics = getFontMetrics();
float height1 = fontMetrics.descent - fontMetrics.ascent;
float height2 = fontMetrics.bottom - fontMetrics.top;123456
```

这里获取的两个高度略有不同，height2的高度会略大于height1，这样在文本的顶部和底部就会有一些留白。具体使用哪个高度，要看具体需求了。

此外，还可以通过Paint.getFontSpacing和Paint.getFontMetrics(null)来获得高度，其实前者也是调用后者来实现的。这里的值和通过fontMetrics.descent - fontMetrics.ascent获得的大小是一致的。

- Layout.getDesiredWidth (获得宽度)

```java
TextPaint textPaint = new TextPaint();
paint.setTextSize(size);//设置字体大小
paint.setTypeface(Typeface.xx);//设置字体
float width = Layout.getDesiredWidth(str,textPaint);
1234
```

之前碰到一个问题，TextView在布局上占用的高度和属性textSize的大小不一样，要比textSize的值大一些（比如textSize=”12dp”，实际的高度大概有14-16dp），仔细看的话会发现文字的上方和下发留有空白。查了下文档，发现可以通过设置android:includeFontPadding来控制是否包含上下空白。

- canvas drawText方法baseY居中的计算方法为

```java
int baseline = (targetRect.bottom + targetRect.top - fontMetrics.bottom - fontMetrics.top) / 2; 
或者
int baseline = targetRect.centerY - top/2 - bottom/2. 
```



## 参考文章

[Android Paint: .measureText() vs .getTextBounds()](https://stackoverflow.com/questions/7549182/android-paint-measuretext-vs-gettextbounds)

