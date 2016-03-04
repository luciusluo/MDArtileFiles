>#Masonry自动布局详解五：比例（multipliedBy）

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

>#支持原创，请[阅读原文](http://www.henishuo.com/masonry-multipliedby/)

#效果图

---
本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/806E55AD-C30B-4875-8B98-6901566E21B6.jpg)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码

---

看下代码：

```
@implementation AspectFitController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  // Create views
  UIView *topView = [[UIView alloc] init];
  topView.backgroundColor = [UIColor redColor];
  [self.view addSubview:topView];
  
  UIView *topInnerView = [[UIView alloc] init];
  topInnerView.backgroundColor = [UIColor greenColor];
  [topView addSubview:topInnerView];
  
  UIView *bottomView = [[UIView alloc] init];
  bottomView.backgroundColor = [UIColor blackColor];
  [self.view addSubview:bottomView];
  
  UIView *bottomInnerView = [[UIView alloc] init];
  bottomInnerView.backgroundColor = [UIColor blueColor];
  [bottomView addSubview:bottomInnerView];
  
  [topView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.right.top.mas_equalTo(0);
    make.height.mas_equalTo(bottomView);
  }];
  
  // width = height / 3
  [topInnerView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.right.mas_equalTo(topView);
    make.width.mas_equalTo(topInnerView.mas_height).multipliedBy(3);
    make.center.mas_equalTo(topView);
    
    // 设置优先级
    make.width.height.mas_equalTo(topView).priorityLow();
    make.width.height.lessThanOrEqualTo(topView);
  }];
  
  [bottomView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.right.bottom.mas_equalTo(0);
    make.height.mas_equalTo(topView);
    make.top.mas_equalTo(topView.mas_bottom);
  }];
  
  // width/height比为1/3.0，要求是同一个控件的属性比例
  [bottomInnerView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.bottom.mas_equalTo(bottomView);
    make.center.mas_equalTo(bottomView);
    // 注意，这个multipliedBy的使用只能是设置同一个控件的，比如这里的bottomInnerView，
    // 设置高/宽为3:1
    make.height.mas_equalTo(bottomInnerView.mas_width).multipliedBy(3);
    
    make.width.height.mas_equalTo(bottomView).priorityLow();
    make.width.height.lessThanOrEqualTo(bottomView);
  }];
}

@end
```

#讲解

---
移除之前的所有约束，然后添加新约束的方法是：`mas_remakeConstraints`。

我们的目标是点击时，将里面的往外面，外面的往里面，并且显示动画效果。其中，最关键的代码是：

```
make.height.mas_equalTo(bottomInnerView.mas_width).multipliedBy(3);
```

>提示：使用`multipliedBy`必须是对同一个控件本身，比如，上面的代码中，我们都是对`bottomInnerView.mas_width`本身的，如果修改成相对于其它控件，会出问题。

我们就说说`bottomInnerView`的约束如何添加。 我们希望`width/height`比为`1/3.0`，首先，我们设置了其`top`和`bottom`与父视图一致且始终在父视图中居中显示：

```
make.top.bottom.mas_equalTo(bottomView);
make.center.mas_equalTo(bottomView);
```

然后我们通过`make.width.height.lessThanOrEqualTo`设置了宽、高的最大值与父视图相同，然后设置了宽和高与父视图相等，但是优先级为最低，以保证子视图的宽高不超过父视图。

```
make.width.height.mas_equalTo(bottomView).priorityLow();
make.width.height.lessThanOrEqualTo(bottomView);
```

最后，我们设置了`bottomInnerView`的高为宽的3倍。

```
make.height.mas_equalTo(bottomInnerView.mas_width).multipliedBy(3);
```

#源代码

---
大家可以到笔者的`github`下载源代码：[https://github.com/CoderJackyHuang/MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

> 温馨提示：本节所讲内容对应于`AspectFitController`中的内容

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**
