#前言

`CALayer`包含在`QuartzCore`框架中，这是一个跨平台的框架，既可以用在`iOS`中又可以用在`Mac OS X`中。后面要学`Core Animation`就应该先学好`Layer`（层）。

我们看一下`UIView`与`Layer`之间的关系图（图片来源于网络）：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/4196_141022100135_1.png)

我们知道，`UIView`有一个属性`layer`，这个是在视图创建时就会自动创建一个图层。想要呈现出来，就需要到`Layer`。层是可以放很多个子层的，也就可以实现多种多样的效果。

#CALayer关键属性说明

```
// 与UIView的bounds类似的，获取或设置图层的大小
@property CGRect bounds;

/* The position in the superlayer that the anchor point of the layer's
 * bounds rect is aligned to. Defaults to the zero point. Animatable. */
// 获取或设置在父图层中对齐位置，默认为(0,0)，也就是左上角。
// 这个属性与UIView的center属性类似，不过在图层中使用的是position
// 这里关系到锚点，关于锚点在游戏世界里是非常关键的概念
// 支持隐式动画
@property CGPoint position;

/* The Z component of the layer's position in its superlayer. Defaults
 * to zero. Animatable. */
// 层与层之间有上下层的关系，设置Z轴方向的值，可以指定哪个层在上，哪个层在下
// 支持隐式动画
@property CGFloat zPosition;

/* Defines the anchor point of the layer's bounds rect, as a point in
 * normalized layer coordinates - '(0, 0)' is the bottom left corner of
 * the bounds rect, '(1, 1)' is the top right corner. Defaults to
 * '(0.5, 0.5)', i.e. the center of the bounds rect. Animatable. */
// 这个就是游戏中必须要懂的锚点，默认为(0.5, 0.5)，也就是正中央。
// 关于锚点的知识，当年自学cocos2d-x的时候也困扰过我一段时间，这个有专门的文章讲解的
// 大家可以查一查相关专题讲解。因为这个确实不好理解，一时说不通。
// 支持隐式动画
@property CGPoint anchorPoint;

/* A transform applied to the layer relative to the anchor point of its
 * bounds rect. Defaults to the identity transform. Animatable. */
// 图层形变，做动画常用
// 支持隐式动画
@property CATransform3D transform;
```

>温馨提示：在`CALayer`中很少使用`frame`属性，因为`frame`本身不支持动画效果，通常使用`bounds`和`position`代替。`CALayer`中透明度使用`opacity`表示而不是`alpha`；中心点使用`position`表示而不是`center`。

#实战点击放大移动效果

先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/move2.gif)

实现代码逻辑：

```
#define kLayerWidth 50

@interface HYBMoveCircleLayerController ()

@property (nonatomic, strong) CALayer *movableCircleLayer;

@end

@implementation HYBMoveCircleLayerController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.movableCircleLayer = [CALayer layer];
  // 指定大小
  self.movableCircleLayer.bounds = CGRectMake(0, 0, kLayerWidth, kLayerWidth);
  // 指定中心点
  self.movableCircleLayer.position = self.view.center;
  // 变成圆形
  self.movableCircleLayer.cornerRadius = kLayerWidth / 2;
  // 指定背景色
  self.movableCircleLayer.backgroundColor = [UIColor blueColor].CGColor;
  // 设置阴影
  self.movableCircleLayer.shadowColor = [UIColor grayColor].CGColor;
  self.movableCircleLayer.shadowOffset = CGSizeMake(3, 3);
  self.movableCircleLayer.shadowOpacity = 0.8;
  
  [self.view.layer addSublayer:self.movableCircleLayer];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
  CGFloat width = kLayerWidth;
  if (self.movableCircleLayer.bounds.size.width <= kLayerWidth) {
    width = kLayerWidth * 2.5;
  }
  
  // 修改大小
  self.movableCircleLayer.bounds = CGRectMake(0, 0, width, width);
  // 将中心位置放到点击位置
  self.movableCircleLayer.position = [[touches anyObject] locationInView:self.view];
  // 再修改成圆形
  self.movableCircleLayer.cornerRadius = width / 2;
}

@end
```

这里需要注意的是每次更新位置时，这个图层的大小和`cornerRadius`都需要更新，否则就不成圆形了!

#通过层内容呈现图片

效果图片：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/Simulator-Screen-Shot-2015年12月8日-上午11.37.51.png)

代码实现：

```
- (void)drawImageWithContent {
  CALayer *layer = [CALayer layer];
  layer.bounds = CGRectMake(0, 0, kPhotoWidth, kPhotoWidth);
  layer.position = self.view.center;
  layer.cornerRadius = kPhotoWidth / 2;
  // 要设置此属性才能裁剪成圆形，但是添加此属性后，下面设置的阴影就没有了。
  layer.masksToBounds = YES;
  layer.borderColor = [UIColor whiteColor].CGColor;
  layer.borderWidth = 1;
  
  // 如果只是显示图片，不做其它处理，直接设置contents就可以了，也就不会出现
  // 绘图和图像倒立的问题了
  layer.contents = (__bridge id _Nullable)([UIImage imageNamed:@"bb"].CGImage);
  
  [self.view.layer addSublayer:layer];
}
```

如果我们只是需要显示图片到图层上，通过设置`contents`属性就可以了，也就不用绘图，也不会出现图像倒立的问题了。

#通过层代理绘制图片

---
上面的内容呈现是很简单的，但是如果我们要增加其它效果呢？那就需要别的方式了。如下效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/Simulator-Screen-Shot-2015年12月8日-上午11.42.25.png)

代码实现：

```
- (void)drawImage {
  CALayer *layer = [CALayer layer];
  layer.bounds = CGRectMake(0, 0, kPhotoWidth, kPhotoWidth);
  layer.position = self.view.center;
  layer.cornerRadius = kPhotoWidth / 2;
  // 要设置此属性才能裁剪成圆形，但是添加此属性后，下面设置的阴影就没有了。
  layer.masksToBounds = YES;
  layer.borderColor = [UIColor whiteColor].CGColor;
  layer.borderWidth = 1;
  
//  // 阴影
//  layer.shadowColor = [UIColor blueColor].CGColor;
//  layer.shadowOffset = CGSizeMake(4, 4);
//  layer.shadowOpacity = 0.9;
  
  // 指定代理
  layer.delegate = self;
  
  // 添加到父图层上
  [self.view.layer addSublayer:layer];
  
  // 当设置masksToBounds为YES后，要想要阴影效果，就需要额外添加一个图层作为阴影图层了
  CALayer *shadowLayer = [CALayer layer];
  shadowLayer.position = layer.position;
  shadowLayer.bounds = layer.bounds;
  shadowLayer.cornerRadius = layer.cornerRadius;
  shadowLayer.shadowOpacity = 1.0;
  shadowLayer.shadowColor = [UIColor redColor].CGColor;
  shadowLayer.shadowOffset = CGSizeMake(2, 1);
  shadowLayer.borderWidth = layer.borderWidth;
  shadowLayer.borderColor = [UIColor whiteColor].CGColor;
  [self.view.layer insertSublayer:shadowLayer below:layer];
  
  // 调用此方法，否则代理不会调用
  [layer setNeedsDisplay];
}

#pragma mark - CALayerDelegate
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
  // 将当前上下文入栈
  CGContextSaveGState(ctx);
  
  // 注意：坐标系统与UIView的不同，这里使用的是笛卡尔积坐标系，也就是左下角为(0,0)
  // 所以，我们只要记住这点就可以很容易地变换了。
  
  // 处理图片倒立的问题
  // 默认呈现是倒立的，因此需要将形变矩阵的sy设置为-1就成了正立的了
  // 先缩放后平移也可以
// CGContextScaleCTM(ctx, 1, -1);
// CGContextTranslateCTM(ctx, 0, -kPhotoWidth);
  
  // 先向平移后旋转也可以解决倒立的问题
  CGContextTranslateCTM(ctx, kPhotoWidth, kPhotoWidth);
  CGContextRotateCTM(ctx, 3.1415926 / 180 * 180);
  
  UIImage *image = [UIImage imageNamed:@"bb"];
  CGContextDrawImage(ctx, CGRectMake(0, 0, kPhotoWidth, kPhotoWidth), image.CGImage);
  
  // 任务完成后，将当前上下文退栈
  CGContextRestoreGState(ctx);
}
```

我们需要注意，一旦设置了层的`layer.masksToBounds`为`YES`，那么阴影效果就没有了，也就是不能直接对该层设置`shadow`相关的属性了，设置了也没有作用。因此，这里使用另外一个图层来专门呈现阴影效果的。

我们需要将阴影层放在图片层的下面，别把图片层给挡住了。默认是不会调用代理方法的，必须要调用`setNeedsDisplay`方法才会回调。因此，要绘制哪个层就调用哪个层对象调用该方法。

在代理方法中，我们介绍一下相关代码。`CGContextSaveGState`方法是入栈操作，也就是将当前设备上下文入栈。既然有入栈，自然也得有出栈，最后一行`CGContextRestoreGState`就是将设备上下文出栈。我们必须明确一点，绘图是在设备上下文上操作的，因此，我们所进行的绘图操作都会是栈顶元素上的。

**矩阵操作：**通过`CGContextDrawImage`绘制的图片是倒立的，因此我们需要进行矩阵相关变换。关于矩阵变换的知识是数学知识，也是知识难点，关于此理论知识，大家可以阅读：[http://www.tuicool.com/articles/Er6VNf6](http://www.tuicool.com/articles/Er6VNf6)

我们需要明确坐标系为笛卡尔积坐标系，坐标原点在左下角。这里提供了两种解决图像倒立问题的方法。

* 第一种：先绽放后平移
* 第二种：先平移后旋转

对于第一种，我们先设置矩阵的sx,sy分别为1，-1，然后平移到(0, -kPhotoWidth)。对于这一种不好理解，因此，笔者提供了第二种方法，更容易理解一些。

第二种，先平移到右上角，然后再旋转180度，正好正立。

网络上的一张图片：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/jINbUf.png)

坐标原点在左下角，我们将倒立的图片先平移到右上角，再顺时针旋转180度正好形成正立。

#源代码

---
小伙伴们可以到github下载源代码哦：[https://github.com/CoderJackyHuang/CALayerDemo](https://github.com/CoderJackyHuang/CALayerDemo)


#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

关注标哥个人网站：[标哥的技术博客](http://www.henishuo.com)


#支持并捐助

如果您觉得文章对您很有帮助，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
