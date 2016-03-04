>#Masonry自动布局详解三：remake约束

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

>#支持原创，请[阅读原文](http://www.henishuo.com/masonry-remake-constraints/)

#效果图

---
本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/remake.gif)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码

---

看下代码：

```
@interface RemakeContraintsController ()

@property (nonatomic, strong) UIButton *growingButton;
@property (nonatomic, assign) BOOL isExpanded;

@end

@implementation RemakeContraintsController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.growingButton = [UIButton buttonWithType:UIButtonTypeSystem];
  [self.growingButton setTitle:@"点我展开" forState:UIControlStateNormal];
  self.growingButton.layer.borderColor = UIColor.greenColor.CGColor;
  self.growingButton.layer.borderWidth = 3;
  self.growingButton.backgroundColor = [UIColor redColor];
  [self.growingButton addTarget:self action:@selector(onGrowButtonTaped:) forControlEvents:UIControlEventTouchUpInside];
  [self.view addSubview:self.growingButton];
  self.isExpanded = NO;
}

- (void)updateViewConstraints {
  // 这里使用update也是一样的。
  // remake会将之前的全部移除，然后重新添加
  [self.growingButton mas_remakeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(0);
    make.left.right.mas_equalTo(0);
    if (self.isExpanded) {
      make.bottom.mas_equalTo(0);
    } else {
      make.bottom.mas_equalTo(-350);
    }
  }];
  
  [super updateViewConstraints];
}

- (void)onGrowButtonTaped:(UIButton *)sender {
  self.isExpanded = !self.isExpanded;
  if (!self.isExpanded) {
    [self.growingButton setTitle:@"点我展开" forState:UIControlStateNormal];
  } else {
    [self.growingButton setTitle:@"点我收起" forState:UIControlStateNormal];
  }
  
  // 告诉self.view约束需要更新
  [self.view setNeedsUpdateConstraints];
  // 调用此方法告诉self.view检测是否需要更新约束，若需要则更新，下面添加动画效果才起作用
  [self.view updateConstraintsIfNeeded];
  
  [UIView animateWithDuration:0.3 animations:^{
    [self.view layoutIfNeeded];
  }];
}

@end
```

#讲解

---
移除之前的所有约束，然后添加新约束的方法是：`mas_remakeConstraints`。

这里展开与收起的关键代码在这里：

```
if (self.isExpanded) {
  make.bottom.mas_equalTo(0);
} else {
  make.bottom.mas_equalTo(-350);
}
```

想要更新约束时添加动画，就需要调用关键的一行代码：`setNeedsUpdateConstraints`，这是选择对应的视图中的约束需要更新。

对于`updateConstraintsIfNeeded`这个方法并不是必须的，但是有时候不调用就无法起到我们的效果。但是，官方都是这么写的，从约束的更新原理上讲，这应该写上。我们要使约束立即生效，就必须调用`layoutIfNeeded`此方法。看下面的方法，就是动画更新约束的核心代码：

```
// 告诉self.view约束需要更新
[self.view setNeedsUpdateConstraints];
// 调用此方法告诉self.view检测是否需要更新约束，若需要则更新，下面添加动画效果才起作用
[self.view updateConstraintsIfNeeded];

[UIView animateWithDuration:0.3 animations:^{
  [self.view layoutIfNeeded];
}];
```

#源代码

---
大家可以到笔者的`github`下载源代码：[https://github.com/CoderJackyHuang/MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

> 温馨提示：本节所讲内容对应于`RemakeConstraintsController`中的内容

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**
