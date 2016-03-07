>#学习UIBezierPath画图

---
笔者在写本篇文章之前，也没有系统学习过贝塞尔曲线，只是曾经某一次的需求需要使用到，才临时百度看了一看而且使用最基本的功能。现在总算有时间停下来好好研究研究这个神奇而伟大的贝塞尔先生！

笔者在学习时，首先看了两遍`UIBezierPath`类头文件定义，熟悉了一下相关的属性和方法。

>#支持原创，请[阅读原文](http://www.henishuo.com/uibezierpath-draw/)

#基础知识

---
使用`UIBezierPath`可以创建基于矢量的路径，此类是`Core Graphics`框架关于路径的封装。使用此类可以定义简单的形状，如椭圆、矩形或者有多个直线和曲线段组成的形状等。
    
`UIBezierPath`是`CGPathRef`数据类型的封装。如果是基于矢量形状的路径，都用直线和曲线去创建。我们使用直线段去创建矩形和多边形，使用曲线去创建圆弧（arc）、圆或者其他复杂的曲线形状。

使用`UIBezierPath`画图步骤：

 1. 创建一个`UIBezierPath`对象
 2. 调用`-moveToPoint:`设置初始线段的起点
 3. 添加线或者曲线去定义一个或者多个子路径
 4. 改变`UIBezierPath`对象跟绘图相关的属性。如，我们可以设置画笔的属性、填充样式等

#`UIBezierPath`创建方法介绍

---
我们先看看`UIBezierPath`类提供了哪些创建方式，这些都是工厂方法，直接使用即可。

```
+ (instancetype)bezierPath;
+ (instancetype)bezierPathWithRect:(CGRect)rect;
+ (instancetype)bezierPathWithOvalInRect:(CGRect)rect;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                            cornerRadius:(CGFloat)cornerRadius;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                        byRoundingCorners:(UIRectCorner)corners 
                              cornerRadii:(CGSize)cornerRadii;
+ (instancetype)bezierPathWithArcCenter:(CGPoint)center 
                                 radius:(CGFloat)radius 
                             startAngle:(CGFloat)startAngle 
                               endAngle:(CGFloat)endAngle 
                              clockwise:(BOOL)clockwise;
+ (instancetype)bezierPathWithCGPath:(CGPathRef)CGPath;
```

下面我们一个一个地介绍其用途。

```
+ (instancetype)bezierPath;
```
这个使用比较多，因为这个工厂方法创建的对象，我们可以根据我们的需要任意定制样式，可以画任何我们想画的图形。

```
+ (instancetype)bezierPathWithRect:(CGRect)rect;
```
这个工厂方法根据一个矩形画贝塞尔曲线。

```
+ (instancetype)bezierPathWithOvalInRect:(CGRect)rect;
```
这个工厂方法根据一个矩形画内切曲线。通常用它来画圆或者椭圆。

```
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                            cornerRadius:(CGFloat)cornerRadius;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                        byRoundingCorners:(UIRectCorner)corners 
                              cornerRadii:(CGSize)cornerRadii;
```
第一个工厂方法是画矩形，但是这个矩形是可以画圆角的。第一个参数是矩形，第二个参数是圆角大小。
第二个工厂方法功能是一样的，但是可以指定某一个角画成圆角。像这种我们就可以很容易地给`UIView`扩展添加圆角的方法了。	

```
+ (instancetype)bezierPathWithArcCenter:(CGPoint)center 
                                 radius:(CGFloat)radius 
                             startAngle:(CGFloat)startAngle 
                               endAngle:(CGFloat)endAngle 
                              clockwise:(BOOL)clockwise;
```
这个工厂方法用于画弧，参数说明如下：
`center`: 弧线中心点的坐标
`radius`: 弧线所在圆的半径
`startAngle`: 弧线开始的角度值
`endAngle`: 弧线结束的角度值
`clockwise`: 是否顺时针画弧线


>###温馨提示：我们下面的代码都是在自定义的BezierPathView类中的`- (void)drawRect:(CGRect)rect`方法中调用 

#画三角形

---
先看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/24456C0A-4A3F-4070-8CE6-CFFD95CEABCD.jpg)

```
// 画三角形
- (void)drawTrianglePath {
  
  UIBezierPath *path = [UIBezierPath bezierPath];
  [path moveToPoint:CGPointMake(20, 20)];
  [path addLineToPoint:CGPointMake(self.frame.size.width - 40, 20)];
  [path addLineToPoint:CGPointMake(self.frame.size.width / 2, self.frame.size.height - 20)];
  
  // 最后的闭合线是可以通过调用closePath方法来自动生成的，也可以调用-addLineToPoint:方法来添加
  //  [path addLineToPoint:CGPointMake(20, 20)];
  
  [path closePath];
  
  // 设置线宽
  path.lineWidth = 1.5;

  // 设置填充颜色
  UIColor *fillColor = [UIColor greenColor];
  [fillColor set];
  [path fill];
  
  // 设置画笔颜色
  UIColor *strokeColor = [UIColor blueColor];
  [strokeColor set];
  
  // 根据我们设置的各个点连线
  [path stroke];
}
```

我们设置画笔颜色通过`set`方法：

```
UIColor *strokeColor = [UIColor blueColor];
[strokeColor set];
```

如果我们需要设置填充颜色，比如这里设置为绿色，那么我们需要在设置画笔颜色之前先设置填充颜色，否则画笔颜色就被填充颜色替代了。也就是说，如果要让填充颜色与画笔颜色不一样，那么我们的顺序必须是先设置填充颜色再设置画笔颜色。如下，这两者顺序不能改变。因为我们设置填充颜色也是跟设置画笔颜色一样调用`UIColor`的`-set`方法。

```
// 设置填充颜色
UIColor *fillColor = [UIColor greenColor];
[fillColor set];
[path fill];

// 设置画笔颜色
UIColor *strokeColor = [UIColor blueColor];
[strokeColor set];
```

#画矩形

---
先看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/rect.jpg)

```
// 画矩形
- (void)drawRectPath {
  UIBezierPath *path = [UIBezierPath bezierPathWithRect:CGRectMake(20, 20, self.frame.size.width - 40, self.frame.size.height - 40)];
  
  path.lineWidth = 1.5;
  path.lineCapStyle = kCGLineCapRound;
  path.lineJoinStyle = kCGLineJoinBevel;
  
  // 设置填充颜色
  UIColor *fillColor = [UIColor greenColor];
  [fillColor set];
  [path fill];
  
  // 设置画笔颜色
  UIColor *strokeColor = [UIColor blueColor];
  [strokeColor set];
  
  // 根据我们设置的各个点连线
  [path stroke];
}
```

`lineCapStyle`属性是用来设置线条拐角帽的样式的，其中有三个选择：

```
/* Line cap styles. */

typedef CF_ENUM(int32_t, CGLineCap) {
    kCGLineCapButt,
    kCGLineCapRound,
    kCGLineCapSquare
};
```
其中，第一个是默认的，第二个是轻微圆角，第三个正方形。


`lineJoinStyle`属性是用来设置两条线连结点的样式，其中也有三个选择：

```
/* Line join styles. */

typedef CF_ENUM(int32_t, CGLineJoin) {
    kCGLineJoinMiter,
    kCGLineJoinRound,
    kCGLineJoinBevel
};
```
其中，第一个是默认的表示斜接，第二个是圆滑衔接，第三个是斜角连接。

#画圆

---
我们可以使用`+ bezierPathWithOvalInRect:`方法来画圆，当我们传的`rect`参数是一下正方形时，画出来的就是圆。

先看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/FCFB3F34-8645-4270-87B1-BC56C9FE763A.jpg)

```
- (void)drawCiclePath {
  // 传的是正方形，因此就可以绘制出圆了
  UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(20, 20, self.frame.size.width - 40, self.frame.size.width - 40)];
  
  // 设置填充颜色
  UIColor *fillColor = [UIColor greenColor];
  [fillColor set];
  [path fill];
  
  // 设置画笔颜色
  UIColor *strokeColor = [UIColor blueColor];
  [strokeColor set];
  
  // 根据我们设置的各个点连线
  [path stroke];
}
```

>注意：要画圆，我们需要传的`rect`参数必须是正方形哦！

#画椭圆

---
先看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/oval.jpg)

前面我们已经画圆了，我们可以使用`+ bezierPathWithOvalInRect:`方法来画圆，当我们传的`rect`参数是一下正方形时，画出来的就是圆。那么我们要是不传正方形，那么绘制出来的就是椭圆了。

```
// 画椭圆
- (void)drawOvalPath {
  // 传的是不是正方形，因此就可以绘制出椭圆圆了
  UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(20, 20, self.frame.size.width - 80, self.frame.size.height - 40)];
  
  // 设置填充颜色
  UIColor *fillColor = [UIColor greenColor];
  [fillColor set];
  [path fill];
  
  // 设置画笔颜色
  UIColor *strokeColor = [UIColor blueColor];
  [strokeColor set];
  
  // 根据我们设置的各个点连线
  [path stroke];
}
```

#画带圆角的矩形

---
```
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                            cornerRadius:(CGFloat)cornerRadius;
+ (instancetype)bezierPathWithRoundedRect:(CGRect)rect
                        byRoundingCorners:(UIRectCorner)corners 
                              cornerRadii:(CGSize)cornerRadii;
```
第一个工厂方法是画矩形，但是这个矩形是可以画圆角的。第一个参数是矩形，第二个参数是圆角大小。
第二个工厂方法功能是一样的，但是可以指定某一个角画成圆角。像这种我们就可以很容易地给`UIView`扩展添加圆角的方法了。

四个都是圆角10：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/roundedRect.jpg)

```
- (void)drawRoundedRectPath {
  UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(20, 20, self.frame.size.width - 40, self.frame.size.height - 40) cornerRadius:10];
  
  // 设置填充颜色
  UIColor *fillColor = [UIColor greenColor];
  [fillColor set];
  [path fill];
  
  // 设置画笔颜色
  UIColor *strokeColor = [UIColor blueColor];
  [strokeColor set];
  
  // 根据我们设置的各个点连线
  [path stroke];
}
```

如果要画只有一个角是圆角，那么我们就修改创建方法：

```
UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:CGRectMake(20, 20, self.frame.size.width - 40, self.frame.size.height - 40) byRoundingCorners:UIRectCornerTopRight cornerRadii:CGSizeMake(20, 20)];
```
其中第一个参数一样是传了个矩形，第二个参数是指定在哪个方向画圆角，第三个参数是一个`CGSize`类型，用来指定水平和垂直方向的半径的大小。看下效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/aroundedrect.jpg)

#画弧

---
画弧前，我们需要了解其参考系，如下图（图片来自网络）：

![image](http://img.blog.csdn.net/20130904200813921?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3JheW9uZGVuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
#define   kDegreesToRadians(degrees)  ((pi * degrees)/ 180)
- (void)drawARCPath {
  const CGFloat pi = 3.14159265359;

  CGPoint center = CGPointMake(self.frame.size.width / 2, self.frame.size.height / 2);
  UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
                                                      radius:100
                                                  startAngle:0
                                                    endAngle:kDegreesToRadians(135)
                                                   clockwise:YES];
  
  path.lineCapStyle = kCGLineCapRound;
  path.lineJoinStyle = kCGLineJoinRound;
  path.lineWidth = 5.0;
  
  UIColor *strokeColor = [UIColor redColor];
  [strokeColor set];
  
  [path stroke];
}
```

效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/arc.jpg)

我们要明确一点，画弧参数`startAngle`和`endAngle`使用的是弧度，而不是角度，因此我们需要将常用的角度转换成弧度。对于效果图中，我们设置弧的中心为控件的中心，起点弧度为0，也就是正东方向，而终点是135度角的位置。如果设置的`clockwise:YES`是逆时针方向绘制，如果设置为`NO`，效果如下：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/clockarc.jpg)

这两者正好是相反的。

#画二次贝塞尔曲线

---
先来学习一下关于控制点，如下图（图片来自网络）：

![image](http://dl2.iteye.com/upload/attachment/0090/9876/6a920716-76c9-39c8-8476-8952b765b2e1.jpeg)

画二次贝塞尔曲线，是通过调用此方法来实现的：

```
- (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint  
```

参数说明：

`endPoint`：终端点<br/>
`controlPoint`：控制点，对于二次贝塞尔曲线，只有一个控制点

看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/second.jpg)

```
- (void)drawSecondBezierPath {
  UIBezierPath *path = [UIBezierPath bezierPath];
  
  // 首先设置一个起始点
  [path moveToPoint:CGPointMake(20, self.frame.size.height - 100)];

  // 添加二次曲线
  [path addQuadCurveToPoint:CGPointMake(self.frame.size.width - 20, self.frame.size.height - 100)
               controlPoint:CGPointMake(self.frame.size.width / 2, 0)];
  
  path.lineCapStyle = kCGLineCapRound;
  path.lineJoinStyle = kCGLineJoinRound;
  path.lineWidth = 5.0;
  
  UIColor *strokeColor = [UIColor redColor];
  [strokeColor set];
  
  [path stroke];
}
```

画二次贝塞尔曲线的步骤：

1. 先设置一个起始点，也就是通过`-moveToPoint:`设置
2. 调用`-addQuadCurveToPoint:controlPoint:`方法设置终端点和控制点，以画二次曲线

在效果图中，拱桥左边的起始点就是我们设置的起始点，最右边的终点，就是我们设置的终端点，而我们设置的控制点为（width / 2, 0）对应于红色矩形中水平方向在正中央，而垂直方向在最顶部。

>这个样式看起来很像`sin`或者`cos`函数吧？这两个只是特例而已，其实可以画任意图形，只是想不到，没有做不到的。

#画三次贝塞尔曲线

---
贝塞尔曲线必定通过首尾两个点，称为端点；中间两个点虽然未必要通过，但却起到牵制曲线形状路径的作用，称作控制点。关于三次贝塞尔曲线的控制器，看下图：

![image](http://dl2.iteye.com/upload/attachment/0090/9882/0f79aaf8-02a0-3833-9b22-736584af5ffa.jpeg)

>提示：其组成是起始端点+控制点1+控制点2+终止端点

如下方法就是画三次贝塞尔曲线的关键方法，以三个点画一段曲线，一般和`-moveToPoint:`配合使用。

```
- (void)addCurveToPoint:(CGPoint)endPoint 
          controlPoint1:(CGPoint)controlPoint1 
          controlPoint2:(CGPoint)controlPoint2
```

看下效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/thirdarc.jpg)

实现代码是这样的：

```
- (void)drawThirdBezierPath {
  UIBezierPath *path = [UIBezierPath bezierPath];
  
  // 设置起始端点
  [path moveToPoint:CGPointMake(20, 150)];
  
  [path addCurveToPoint:CGPointMake(300, 150)
          controlPoint1:CGPointMake(160, 0)
          controlPoint2:CGPointMake(160, 250)];
  
  path.lineCapStyle = kCGLineCapRound;
  path.lineJoinStyle = kCGLineJoinRound;
  path.lineWidth = 5.0;
  
  UIColor *strokeColor = [UIColor redColor];
  [strokeColor set];
  
  [path stroke];
}
```

我们需要注意，这里确定的起始端点为(20,150)，终止端点为(300, 150)，基水平方向是一致的。控制点1的坐标是（160，0），水平方向相当于在中间附近，这个参数可以调整。控制点2的坐标是（160，250），如果以两个端点的连线为水平线，那么就是250-150=100，也就是在水平线下100。这样看起来就像一个`sin`函数了。

#源代码下载

---
小伙伴们可以到github下载：[https://github.com/CoderJackyHuang/UIBezierPathLayerDemos](https://github.com/CoderJackyHuang/UIBezierPathLayerDemos)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/uibezierpath-draw/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/uibezierpath-draw/)**



