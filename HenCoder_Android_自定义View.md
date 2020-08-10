---
title: HenCoder Android：自定义View——绘制基础
---

**总结** 

1. 自定义绘制的方式是重写绘制方法，其中最常用的是```onDraw()```
2. 绘制的关键是Canvas的使用
   1. Canvas的绘制方法：drawXXX()（关键参数Paint）
   2. Canvas的辅助类方法：范围裁剪（clipXXX）和几何变换(Matrix)
3. 可以使用不同的绘制方法来控制遮盖关系

**1. onDraw()**

```java
Paint paint = new Paint();
@Override
protected void onDraw(Canvas canvas){
    super.onDraw(canvas);
    //绘制一个圆
    canvas.drawCircle(300, 300, 200, paint);
}
```

**2. Canvas.drawXXX()和Paint基础**

1. Canvas 类下的所有 draw- 打头的方法，例如 drawCircle() drawBitmap()

2. Paint类的几个常用方法，如下：

   ```java
   Paint.setStyle(Style style) //设置绘制模式
   Paint.setColor(int color) //设置颜色
   Paint.setStrokeWidth(float width) //设置线条宽度
   Paint.setTextSize(float textSize) //设置文字大小
   Paint.setAntiAlias(boolean aa) //设置抗锯齿开关
   ```

**3. View的坐标系**

在Android中，每个View都有一个自己的坐标系，彼此之间是互不影响的。这个坐标系的原点是View左上角的那个点；水平方向是X轴，右正左负；竖直方向是y轴，下正上负

![View的坐标系](http://ws3.sinaimg.cn/large/006tNc79ly1fig7syr2ghj30h40etdfz.jpg)

**4. drawPath(Path path, Paint paint)**

```Path```有两类方法，一类是直接描述路径，一类是辅助的设置或计算

 1. addXxx() ——添加子图形

    ```java
    addCircle(float x, float y, float radius, Direction dir) 添加圆
    ```

    ```Direction```是画圆的路径的方向：顺时针(CW clockwise)和逆时针(CCW counter-clockwise)，对于普通情况，这个参数填CW还是CCW都没有影响，它只是在需要填充图形(```Paint.Style```为FILL或FILL_AND_STROKE)，并且图形出现自相交时，用于判断填充范围

    其他的Path.addXxx()方法：

    ```java
    addOval(float left, float top, float right, float bottom, Direction dir) / addOval(RectF oval, Direction dir) 添加椭圆
    addRect(float left, float top, float right, float bottom, Direction dir) / addRect(RectF rect, Direction dir) 添加矩形
    addRoundRect(RectF rect, float rx, float ry, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float rx, float ry, Direction dir) / addRoundRect(RectF rect, float[] radii, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float[] radii, Direction dir) 添加圆角矩形
    addPath(Path path) 添加另一个 Path
    ```

	2. xxxTo() ——画线（直线或曲线）

    ```java
    lineTo(float x, float y)/rLineTo(float x, float y) 画直线
    区别：
    从当前位置向目标位置画一条直线， x 和 y 是目标位置的坐标。这两个方法的区别是，lineTo(x, y) 的参数是绝对坐标，而 rLineTo(x, y) 的参数是相对当前位置的相对坐标 （前缀 r 指的就是 relatively 「相对地」)。
    ```

    >当前位置：所谓当前位置，即最后一次调用画 Path 的方法的终点位置。初始值为原点 (0, 0)。

