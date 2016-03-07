>#CABasicAnimation精讲

---
本教程写了这个效果图的demo，同时总结`CABasicAnimation`的使用方法。

![image](http://www.henishuo.com/wp-content/uploads/2015/12/animation.gif)

看完gif动画完，看到了什么？平移、旋转、缩放、闪烁、路径动画。

#实现平移动画

---
实现平移动画，我们可以通过`transform.translation`或者水平`transform.translation.x`或者垂直平移`transform.translation.y`添加动画。

```
// 平移动画
- (void)baseTranslationAnimation {
  UIView *springView = [[UIView alloc] initWithFrame:CGRectMake(0, 380, 50, 50)];
  [self.view addSubview:springView];
  springView.layer.borderColor = [UIColor greenColor].CGColor;
  springView.layer.borderWidth = 2;
  springView.backgroundColor = [UIColor redColor];
  
  CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.translation"];
  animation.duration = 2;
  
  CGFloat width = self.view.frame.size.width;
  animation.toValue = [NSValue valueWithCGPoint:CGPointMake(width - 50, 0)];
  
  // 指定动画重复多少圈是累加的
  animation.cumulative = YES;
  // 动画完成是不自动很危险
  animation.removedOnCompletion = NO;
  // 设置移动的效果为快入快出
  animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
  // 设置无限循环动画
  animation.repeatCount = HUGE_VALF;
  // 设置动画完成时，自动以动画回到原点
  animation.autoreverses = YES;
  // 设置动画完成时，返回到原点
  animation.fillMode = kCAFillModeForwards;
  
  [springView.layer addAnimation:animation forKey:@"transform.translation"];
}
```

`translation`是平移的意思，大家需要记住它。这里只是水平移动，其实我们可以直接对`transform.translation.x`设置动画。不过直接使用`transform.translation`也是可以的，我们设置`y`值为0就可以了。

首先，我们通过属性路径的方法来创建动画对象：

```
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.translation"];
```

我们设置目的地为水平移动到屏宽再减去控件的宽50，由于我们只是水平移动，垂直方向没有移动，因此第二个参数设置为0即可。我们需要明确一点，`toValue`这里是指移动的距离而不是移到这个点：

```
animation.toValue = [NSValue valueWithCGPoint:CGPointMake(width - 50, 0)];
```

对于其它属性的设置，看注释里的说明就可以明白了。

#旋转动画

---
旋转动画需要借助`CATransform3D`这个表示三维空间的结构体，可以X轴旋转、Y轴旋转、Z轴旋转：

```
// 旋转动画
- (void)baseRotationAnimation {
  UIView *springView = [[UIView alloc] initWithFrame:CGRectMake(0, 240, 50, 50)];
  [self.view addSubview:springView];
  springView.layer.borderColor = [UIColor greenColor].CGColor;
  springView.layer.borderWidth = 2;
  springView.backgroundColor = [UIColor redColor];
  
  CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform"];
  animation.duration = 2;
  
  // Z轴旋转180度
  CATransform3D transform3d = CATransform3DMakeRotation(3.1415926, 0, 0, 180);
  animation.toValue = [NSValue valueWithCATransform3D:transform3d];
  
  animation.cumulative = YES;
  animation.removedOnCompletion = NO;
  animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
  animation.repeatCount = HUGE_VALF;
  animation.autoreverses = YES;
  animation.fillMode = kCAFillModeForwards;
  
  [springView.layer addAnimation:animation forKey:@"transform"];
}
```

我们通过属性路径创建动画：

```
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform"];
```

然后通过创建`CATransform3D`结构体，指定旋转的角度为180度，X、Y轴不旋转，Z轴旋转180度：

```
CATransform3D transform3d = CATransform3DMakeRotation(3.1415926, 0, 0, 180);
animation.toValue = [NSValue valueWithCATransform3D:transform3d];
```

其它属性设置与平移动画一样。

#缩放动画

---
`transform.scale`这个是图的属性路径，设置`scale`值就可以达到缩放的效果：

```
// 缩放动画
- (void)baseScaleAnimation {
  UIView *springView = [[UIView alloc] initWithFrame:CGRectMake(0, 120, 50, 50)];
  [self.view addSubview:springView];
  springView.layer.borderColor = [UIColor greenColor].CGColor;
  springView.layer.borderWidth = 2;
  springView.backgroundColor = [UIColor redColor];
  
  CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
  animation.duration = 2;
  animation.fromValue = @(1);
  animation.toValue = @(0);
  animation.removedOnCompletion = NO;
  animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
  animation.repeatCount = HUGE_VALF;
  animation.autoreverses = YES;
  animation.fillMode = kCAFillModeForwards;
  
  [springView.layer addAnimation:animation forKey:@"transform.scale"];
}
```

我们通过属性路径方法创建动画对象：

```
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
```

我们设置了初始变换和最终变换为1和0：

```
animation.fromValue = @(1);
animation.toValue = @(0);
```

其实由于图初始状态值为正常状态，没有任何缩放，因此其值本就是1，所以`fromValue`可以不设置的。

#闪烁动画

---
我们这里说的闪烁动画其实就是透明度的变化，当然我们不能通过`alpha`值的变化来实现闪烁动画，因此这个属性是不具备隐式动画效果的。不过系统提供了`opacity`，我们可以通过这个值的变化来实现闪烁效果。

```
// 闪烁动画
- (void)baseSpringAnimation {
  UIView *springView = [[UIView alloc] initWithFrame:CGRectMake(0, 50, 50, 50)];
  [self.view addSubview:springView];
  springView.layer.borderColor = [UIColor greenColor].CGColor;
  springView.layer.borderWidth = 2;
  springView.backgroundColor = [UIColor redColor];
  
  CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"opacity"];
  animation.duration = 2;
  animation.fromValue = @(1);
  animation.toValue = @(0);
  animation.removedOnCompletion = NO;
  animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
  animation.repeatCount = HUGE_VALF;
  animation.autoreverses = YES;
  animation.fillMode = kCAFillModeForwards;
  
  [springView.layer addAnimation:animation forKey:@"opacity"];
}
```

我们通过属性路径`opacity`来创建动画对象，注意不能使用`alpha`，否则不会有动画效果的：

```
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"opacity"];
```

我们设置透明度从1->0变换，其它属性设置与上面平移动画一样：

```
animation.fromValue = @(1);
animation.toValue = @(0);
```

#路径动画

---
路径动画这里添加了灰常详细的注释说明，几乎都包含了所有常用的属性设置了：

```
// 路径动画
- (void)baseAnimation {
  UIView *animationView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 100, 100)];
  animationView.layer.borderWidth = 2;
  animationView.layer.borderColor = [UIColor redColor].CGColor;
  animationView.backgroundColor = [UIColor greenColor];
  [self.view addSubview:animationView];
  
  // 添加动画
  CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position"];
  // 起点，这个值是指position，也就是layer的中心值
  animation.fromValue = [NSValue valueWithCGPoint:CGPointMake(50, 50)];
  // 终点，这个值是指position，也就是layer的中心值
  animation.toValue = [NSValue valueWithCGPoint:CGPointMake(self.view.bounds.size.width - 50,
                                                            self.view.bounds.size.height - 100)];
  // byValue与toValue的区别：byValue是指x方向再移动到指定的宽然后y方向移动指定的高
  // 而toValue是整体移动到指定的点
  //  animation.byValue = [NSValue valueWithCGPoint:CGPointMake(self.view.bounds.size.width - 50 - 50,
  //                                                            self.view.bounds.size.height - 50 - 50 - 50)];
  // 线性动画
  animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
  animation.removedOnCompletion = NO;
  
  // 设定开始值到结束值花费的时间，也就是动画时长，单位为秒
  animation.duration = 2;
  
  // 播放速率，默认为1，表示常速
  // 设置为2则以2倍的速度播放，同样设置为N则以N倍速度播放
  // 如果值小于1，自然就是慢放
  animation.speed = 0.5;
  
  // 开始播放动画的时间，默认为0.0，通常是在组合动画中使用
  animation.beginTime = 0.0;
  
  // 播放动画的次数，默认为0，表示只播放一次
  // 设置为3表示播放3次
  // 设置为HUGE_VALF表示无限动画次数
  animation.repeatCount = HUGE_VALF;
  
  // 默认为NO，设置为YES后，在动画达到toValue点时，就会以动画由toValue返回到fromValue点。
  // 如果不设置或设置为NO，在动画到达toValue时，就会突然马上返回到fromValue点
  animation.autoreverses = YES;
  
  // 当autoreverses设置为NO时，最终会留在toValue处
  animation.fillMode = kCAFillModeForwards;
  // 将动画添加到层中
  [animationView.layer addAnimation:animation forKey:@"position"];
}
```

在图中`position`是层相对于父层的中心，而UI控件的`center`中心一样。这里要整体曲线路径移动，我们通过`position`中心点的变换就可以曲线路径移动。

这里设置了`CAMediaTiming`协议中的所有属性，详细看代码中的注释吧，已经很详细了！

#源代码

---
小伙伴们可以到github下载源代码了：[https://github.com/CoderJackyHuang/CALayerDemo](https://github.com/CoderJackyHuang/CALayerDemo)

###随手点个star吧~

#[阅读原文](http://www.henishuo.com/cabasicanimation-introduce-in-detail/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**


