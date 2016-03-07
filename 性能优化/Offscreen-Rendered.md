#序言

本篇是记录笔者探索离屏渲染的笔记，学习离屏渲染如何引起、如何解决。同时，好好理解一下什么离屏渲染（Offscreen-rendered），什么是当屏渲染（Onscreen-Rendered）。了解GPU与CPU的一些小知识。

本篇主要一起来学习学习：

* **Color Offscreen-Rendered Yellow.** Places a yellow overlay over content that is rendered offscreen.
* **Color Hits Green and Misses Red.** Marks views in green or red. A view that is able to use a cached rasterization is marked in green.

#基本概念

在OpenGL中，GPU屏幕渲染有以下两种方式：

* Onscreen-Rendered：当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行。
* Offscreen-Rendered：即离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。

#Offscreen-rendered缺点

与当前屏幕渲染相比，离屏渲染的代价是很高的，主要体现在两个方面：

* 创建新缓冲区：要想进行离屏渲染，首先要创建一个新的缓冲区。
* 上下文切换：离屏渲染的整个过程，需要多次切换上下文环境。先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕。但是，上下文环境的切换是要付出很大代价的。

#哪些可引起Offscreen-rendered

有以下方式可以引起离屏渲染：

* shouldRasterize设置为YES
* masks（遮罩）
* shadows（阴影）
* edge antialiasing（抗锯齿）
* group opacity（不透明）
* cornerRadius+clipToBounds/maskToBounds设置圆角
* 重写drawRect交由CPU渲染

#如何检测离屏渲染

在Instruments->Core Animation下，如下图，勾选上：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306-2@2x.png)

**注意：**要求在真机才可以的哦！

#光栅化（shouldRasterize）

shouldRasterize = YES时，在其他属性触发离屏渲染的同时，会将光栅化后的内容缓存起来，以便在下一帧的时候可以直接复用。这会隐式地创建一个位图，各种阴影遮罩等效果也会保存到位图中并缓存起来，从而减少渲染的次数，提高一定的性能。

当我们设置了shaouldRasterize = YES后，我们就可以开启**“Color Hits Green and Misses Red”**来检查该场景下光栅化操作是否是一个好的选择。命中缓存则显示绿色，不命中缓存，则会显示为红色。

因此，我们要通过检测是否值得使用光栅化。比如，如果在设置为YES后，在快速滚动的时候，绿色较少，而红色较多，说明并不可观，表示不建议使用光栅化。相反，如果在快速滚动的时候，基本都是绿色的，只有极少数偶尔出现红色，那么使用光栅化来优化是可以达到很好的效果的。

我们启动Instruments->Core Animation勾选如下选项：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306-5@2x.png)

下面是笔者测试的效果图：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/rasterize.gif)

从GIF图中可以看出来，命中率并不高，因此不适合使用光栅化来优化。

设置光栅化只需要一行代码：

```
// 光栅化
imgView.layer.shouldRasterize = YES;
```

设置上面的代码后，引起了离屏渲染：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306-3@2x-e1457266016462.png)

我们要根据实际检测的结果来决定是否应该使用光栅化。

#圆角引起离屏渲染

通过以下设置会引起离屏渲染：

```
imgView.layer.cornerRadius = 10;
imgView.layer.masksToBounds = YES;
```

或者这么设置：

```
//    self.headImageView.layer.cornerRadius = 30;
//    self.headImageView.clipsToBounds = YES;
```

直接设置cornerRadius属性是不会引起离屏渲染的，而且连圆角也不会有，因为只有后面的设置为YES，才会真正地渲染。

圆角引起的离屏渲染效果如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306-6@2x-e1457266342530.png)

#阴影引起离屏渲染

我们测试一下设置阴影（随意设置的）：

```
imgView.layer.shadowColor = [UIColor redColor].CGColor;
imgView.layer.shadowOffset = CGSizeMake(10, 10);
imgView.layer.shadowOpacity = 0.8;
```

检测到离屏渲染如下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306-4@2x-e1457266442220.png)

#edge antialiasing、group opacity

笔者测试了一下，直接设置这几个，并没有检测出离屏渲染，请会的人给补上小例子。笔者的测试小例子是这么写的：

```
if (type == 2) {
	// 测试发现未引起离屏渲染
	imgView.layer.edgeAntialiasingMask = kCALayerTopEdge;
	imgView.layer.allowsEdgeAntialiasing = NO;
} else if (type == 3) {
	// 透明，发现未引起离屏渲染
	imgView.opaque = NO;
	imgView.layer.allowsGroupOpacity = YES;
}
```

我想肯定是笔者理解错了，不了解这几个如何去使用~后续再补上吧！

#如何选择使用哪种渲染方式

摆在我们面前有三种选择：

* 当前屏幕渲染
* 离屏渲染
* CPU渲染（特殊的离屏渲染）

我们应该选择使用哪种方式呢？这需要根据具体的使用场景来决定。但是，我们**尽量使用当前屏幕渲染**，因为离屏渲染、CPU渲染可能会带来性能问题。所以，一般情况下我们要尽量使用当前屏幕渲染。

由于GPU的浮点运算能力比CPU强，CPU渲染的效率可能不如离屏渲染；但如果仅仅是实现一个简单的效果，直接使用CPU渲染的效率又可能比离屏渲染好，因为离屏渲染要涉及到缓冲区创建和上下文切换等耗时操作。

所以，具体的选择应该要我们通过性能测试所得的结果来决定到底采用哪一种方式优化的效果会更佳。

#最终优化后的效果

这里笔者去掉光栅化、去掉阴影、圆角采用生成圆角图片的方式异步加载，最终的效果如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160306_last-e1457267481792.png)

已经没有黄色部分了，说明我们优化得差不多了！

#源代码

本篇对应的源代码小Demo下载地址为：[CoderJackyHuang：PerformaceDemo](https://github.com/CoderJackyHuang/PerformanceDemo)

#推荐阅读

* [Color Blended Layers：图层混合优化](http://www.henishuo.com/color-blended-layers/)
* [Color Misaligned Images：图片像素对齐优化](http://www.henishuo.com/color-misaligned-images/)

#参考

* [Measure Graphics Performance](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/MeasuringGraphicsPerformance.html#//apple_ref/doc/uid/TP40004652-CH32-SW1)
* [关于性能的一些问题](http://www.reviewcode.cn/article.html?reviewId=7)