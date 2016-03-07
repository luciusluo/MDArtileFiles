#前言

---

本次分享将从以下方面进行展开：

1. 曾被面试官问倒过的问题：层与视图的关系
2. `CALayer`类介绍及层与视图的关系
3. `CAShapeLayer`类介绍
4. `UIBezierPath`贝塞尔曲线讲解
5. `CoreAnimation`之动画子类介绍
6. `CATransitionAnimation`类实现各种过滤动画

关于`Core Animation`在`iOS`系统中的关系图如下：

---

![image](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/ca_architecture_2x.png)

可以看出，`Core Animation`是相对上层的封装，介于`UIKit`与`Core Graphics`、`OpenGL/OpenGL ES`之间。最底下还有一个`Graphics Hardware`，就是硬件了！！！

#层与视图的关系

---
我们先看看`Window`与`Layer`之间的关系：

![image](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/basics_layer_rendering_2x.png)

这个图告诉我们，层是基于绘画模型实现的，层并不会在我们的`app`中做什么事，实际上是层只是捕获`app`所提供的内容，并缓存成`bitmap`，当任何与层关联的属性值发生变化时，`Core Animation`就会将新的`bitmap`传给绘图硬件，并根据新的位图更新显示。

`UIView`是`iOS`系统中界面元素的基础，所有的界面元素都是继承自`UIView`。它本身完全是由`CoreAnimation`来实现的。它真正的绘图部分，是由一个`CALayer`类来管理。`UIView`本身更像是一个`CALayer`的管理器，访问它的跟绘图和跟坐标有关的属性，例如`frame`、`bounds`等，实际上内部都是在访问它所包含的`CALayer`的相关属性。

>提示：`layer-based drawing`不同于`view-based drawing`，后者的性能消耗是很高的，它是在主线程上直接通过`CPU`完成的，而且通常是在`-drawRect:`中绘制动画。

##UIView与CALayer的联系

我们看看`UIView`与`layer`之间的关系图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/lv.png)

我们可以看到，一个`UIView`默认就包含一个`layer`属性，而`layer`是可以包含`sublayer`的，因此形成了图层树。从此图可以看出这两者的关系：视图包含一个`layer`属性且这个`layer`属性可以包含很多个`sublayer`。

有人说`UIView`就像一个画板，而`layer`就像画布，一个画板上可以有很多块画布，但是画布不能有画板。

##UIView与CALayer的主要区别

1. `UIView`是可以响应事件的，但是`CALayer`不能响应事件
2. `UIView`主要负责管理内容，而`CALayer`主要负责渲染和呈现。如果没有`CALayer`，我们是看不到内容的。
3. `CALayer`维护着三个`layer tree`,分别是`presentLayer Tree`、`modeLayer Tree`、`Render Tree`,在做动画的时候，我们修改动画的属性，其实是修改`presentLayer`的属性值,而最终展示在界面上的其实是提供`UIView`的`modelLayer`。

官方说明了`UIView`与`CALayer`的联系：

>Layers are not a replacement for your app’s views—that is, you cannot create a visual interface based solely on layer objects. Layers provide infrastructure for your views. Specifically, layers make it easier and more efficient to draw and animate the contents of views and maintain high frame rates while doing so. However, there are many things that layers do not do. **Layers do not handle events**, **draw content**, **participate in the responder chain**, **or do many other things**. For this reason, every app must still have one or more views to handle those kinds of interactions.

#说说CALayer

---

我们首先得明确`Layer`在`iOS`系统上的坐标系起点是在左上角的，而在`OS X`系统上是左下角的：

![image](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_bounds_2x.png)

笔者对`Layer`相关的属性和方法画了这么一张图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/CALayerFull.png)


看看官方关于`Layer Tree`的说明：

![image](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/sublayer_hierarchies_2x.png)

关于这一节，请阅读[iOS CALayer解读](http://www.henishuo.com/calayer-learning/)，文章末尾有Demo可以下载运行看效果。

#说说UIBezierPath

---
关于这一节，请阅读[iOS UIBezierPath解读](http://www.henishuo.com/uibezierpath-draw/)，文章末尾有Demo可以下载运行看效果。

#说说CAShapeLayer

---
关于这一节，请阅读[iOS CAShapeLayer解读](http://www.henishuo.com/ios-cashapelayer-learning/)，文章末尾有Demo可以下载运行看效果。

#Core Animation介绍

---
我们在开发中常见的动画：

![image](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/basics_animation_types_2x.png)

笔者将`Core Animation`的关系图及相关属性、方法说明都通过该图来表达：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/CAAnimationFull-1.png)

如果我们要改变动画的行为，我们可以实现`CAAction`协议的方法，像这样：

```
- (id<CAAction>)actionForLayer:(CALayer *)theLayer
                        forKey:(NSString *)theKey {
    CATransition *theAnimation=nil;
 
    if ([theKey isEqualToString:@"contents"]) {
        theAnimation = [[CATransition alloc] init];
        theAnimation.duration = 1.0;
        theAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
        theAnimation.type = kCATransitionPush;
        theAnimation.subtype = kCATransitionFromRight;
    }
    return theAnimation;
}
```

#参考文档 

---
参考官方文档：[iOS CoreAnimation](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW19)

#[阅读原文](http://www.henishuo.com/core-animation/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**

