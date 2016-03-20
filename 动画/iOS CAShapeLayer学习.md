#前言

`CAShapeLayer`继承自`CALayer`，因此，可使用`CALayer`的所有属性。但是，`CAShapeLayer`需要和贝塞尔曲线配合使用才有意义。

关于`UIBezierPath`，请阅读文章：[iOS UIBezierPth精讲](http://www.henishuo.com/uibezierpath-draw/)

#基本知识

看看官方说明：

```
/* The shape layer draws a cubic Bezier spline in its coordinate space.
 *
 * The spline is described using a CGPath object and may have both fill
 * and stroke components (in which case the stroke is composited over
 * the fill). The shape as a whole is composited between the layer's
 * contents and its first sublayer.
 */
```
上面只是部分说明内容，由于较长，只放一部分出来。这里是说`CAShapeLayer`是在其坐标系统内绘制贝塞尔曲线的。因此，使用`CAShapeLayer`需要与`UIBezierPath`一起使用。

它有一个`path`属性，而`UIBezierPath`就是对`CGPathRef`类型的封装，因此这两者配合起来使用才可以的哦！

```
@property(nullable) CGPathRef path;
```

#CAShapeLayer和drawRect的比较

* `drawRect`：属于`CoreGraphics`框架，占用`CPU`，性能消耗大，不建议重写
* `CAShapeLayer`：属于`CoreAnimation`框架，通过`GPU`来渲染图形，节省性能。动画渲染直接提交给手机`GPU`，不消耗内存

这两者各有各的用途，而不是说有了`CAShapeLayer`就不需要`drawRect`。

**温馨提示**：`drawRect`只是一个方法而已，是`UIView`的方法，重写此方法可以完成我们的绘制图形功能。

#CAShapeLayer与UIBezierPath的关系

`CAShapeLayer`与`UIBezierPath`的关系：

1. `CAShapeLayer`中`shape`代表形状的意思，所以需要形状才能生效
2. 贝塞尔曲线可以创建基于矢量的路径，而`UIBezierPath`类是对`CGPathRef`的封装
3. 贝塞尔曲线给`CAShapeLayer`提供路径，`CAShapeLayer`在提供的路径中进行渲染。路径会闭环，所以绘制出了`Shape`
4. 用于`CAShapeLayer`的贝塞尔曲线作为`path`，其`path`是一个首尾相接的闭环的曲线，即使该贝塞尔曲线不是一个闭环的曲线

#CAShapeLayer与UIBezierPath画圆

效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/circleofcashapelayer-e1449477454675.png)

```
- (CAShapeLayer *)drawCircle {
  CAShapeLayer *circleLayer = [CAShapeLayer layer];
  // 指定frame，只是为了设置宽度和高度
  circleLayer.frame = CGRectMake(0, 0, 200, 200);
  // 设置居中显示
  circleLayer.position = self.view.center;
  // 设置填充颜色
  circleLayer.fillColor = [UIColor clearColor].CGColor;
  // 设置线宽
  circleLayer.lineWidth = 2.0;
  // 设置线的颜色
  circleLayer.strokeColor = [UIColor redColor].CGColor;
  
  // 使用UIBezierPath创建路径
  CGRect frame = CGRectMake(0, 0, 200, 200);
  UIBezierPath *circlePath = [UIBezierPath bezierPathWithOvalInRect:frame];
  
  // 设置CAShapeLayer与UIBezierPath关联
  circleLayer.path = circlePath.CGPath;
  
  // 将CAShaperLayer放到某个层上显示
  [self.view.layer addSublayer:circleLayer];
  
  return circleLayer;
}
```

注意，我们这里不是放在`-drawRect:`方法中调用的。我们直接将这个`CAShaperLayer`放到了`self.view.layer`上，直接呈现出来。

我们创建一个`CAShapeLayer`，然后配置相关属性，然后再通过`UIBezierPath`的类方法创建一个内切圆路径，然后将路径指定给`CAShapeLayer.path`，这就将两者关联起来了。最后，将这个层放到了`self.view.layer`上呈现出来。

#CAShapeLayer与UIBezierPath的简单Loading效果

效果图类似这样（懒自己做图，就百度了一个）：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/圆形进度条iOS.gif)

我们调用了上面这个画圆效果的代码：

```
- (void)drawHalfCircle {
  self.loadingLayer = [self drawCircle];
  
  // 这个是用于指定画笔的开始与结束点
  self.loadingLayer.strokeStart = 0.0;
  self.loadingLayer.strokeEnd = 0.75;
  
  self.timer = [NSTimer scheduledTimerWithTimeInterval:0.1
                                                target:self
                                              selector:@selector(updateCircle)
                                              userInfo:nil
                                               repeats:YES];
}

- (void)updateCircle {
  if (self.loadingLayer.strokeEnd > 1 && self.loadingLayer.strokeStart < 1) {
    self.loadingLayer.strokeStart += 0.1;
  } else if (self.loadingLayer.strokeStart == 0) {
    self.loadingLayer.strokeEnd += 0.1;
  }
  
  if (self.loadingLayer.strokeEnd == 0) {
    self.loadingLayer.strokeStart = 0;
  }
  
  if (self.loadingLayer.strokeStart >= 1 && self.loadingLayer.strokeEnd >= 1) {
    self.loadingLayer.strokeStart = 0;
    [self.timer invalidate];
    self.timer = nil;
  }
}
```

我们要实现这个效果，是通过`strokeStar`和`strokeEnd`这两个属性来完成的，看看官方说明：

```
/* These values define the subregion of the path used to draw the
 * stroked outline. The values must be in the range [0,1] with zero
 * representing the start of the path and one the end. Values in
 * between zero and one are interpolated linearly along the path
 * length. strokeStart defaults to zero and strokeEnd to one. Both are
 * animatable. */

@property CGFloat strokeStart;
@property CGFloat strokeEnd;
```
这里说明了这两个值的范围是[0,1]，当`strokeStart`的值为0慢慢变成1时，我们看到路径是慢慢消失的。这里实现的效果并不好，因为不能一起循环着。不过，在这里学习的目的已经达到了，后面学习动画效果时，才专门学习它。

#源代码下载

小伙伴们可以到github下载：[https://github.com/CoderJackyHuang/UIBezierPathLayerDemos](https://github.com/CoderJackyHuang/UIBezierPathLayerDemos)

要测试哪种效果，就打开对应的注释就可以了。

