>#Masonry自动布局详解四：整体动画更新约束

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

#效果图

---
本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/4.gif)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码

---

看下代码：

```
@interface TotalUpdateController ()

@property (nonatomic, strong) UIView *purpleView;
@property (nonatomic, strong) UIView *orangeView;
@property (nonatomic, assign) BOOL isExpaned;

@end

@implementation TotalUpdateController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  UIView *purpleView = [[UIView alloc] init];
  purpleView.backgroundColor = UIColor.purpleColor;
  purpleView.layer.borderColor = UIColor.blackColor.CGColor;
  purpleView.layer.borderWidth = 2;
  [self.view addSubview:purpleView];
  UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(onTap)
                                 ];
  [purpleView addGestureRecognizer:tap];
  self.purpleView = purpleView;
  
  UIView *orangeView = UIView.new;
  orangeView.backgroundColor = UIColor.orangeColor;
  orangeView.layer.borderColor = UIColor.blackColor.CGColor;
  orangeView.layer.borderWidth = 2;
  [self.view addSubview:orangeView];
  self.orangeView = orangeView;
  
  // 这里，我们不使用updateViewConstraints方法，但是我们一样可以做到。
  // 不过苹果推荐在updateViewConstraints方法中更新或者添加约束的
  [self updateWithExpand:NO animated:NO];
  
  UILabel *label = [[UILabel alloc] init];
  label.numberOfLines = 0;
  label.textColor = [UIColor redColor];
  label.font = [UIFont systemFontOfSize:16];
  label.textAlignment = NSTextAlignmentCenter;
  label.text = @"点击purple部分放大，orange部分最大值250，最小值90";
  [self.purpleView addSubview:label];
[label mas_makeConstraints:^(MASConstraintMaker *make) {
  make.bottom.mas_equalTo(0);
  make.left.right.mas_equalTo(0);
}];
}

- (void)updateWithExpand:(BOOL)isExpanded animated:(BOOL)animated {
  self.isExpaned = isExpanded;
  
  [self.purpleView mas_updateConstraints:^(MASConstraintMaker *make) {
    make.left.top.mas_equalTo(20);
    make.right.mas_equalTo(-20);
    if (isExpanded) {
      make.bottom.mas_equalTo(-20);
    } else {
      make.bottom.mas_equalTo(-300);
    }
  }];
  
  [self.orangeView mas_updateConstraints:^(MASConstraintMaker *make) {
    make.center.mas_equalTo(self.purpleView);
    
     // 这里使用优先级处理
    // 设置其最大值为250，最小值为90
    if (!isExpanded) {
      make.width.height.mas_equalTo(100 * 0.5).priorityLow();
    } else {
      make.width.height.mas_equalTo(100 * 3).priorityLow();
    }
    
    // 最大值为250
    make.width.height.lessThanOrEqualTo(@250);
    
    // 最小值为90
    make.width.height.greaterThanOrEqualTo(@90);
  }];
  
  if (animated) {
    [self.view setNeedsUpdateConstraints];
    [self.view updateConstraintsIfNeeded];
    
    [UIView animateWithDuration:0.5 animations:^{
      [self.view layoutIfNeeded];
    }];
  }
}

- (void)onTap {
  [self updateWithExpand:!self.isExpaned animated:YES];
}

@end
```

#讲解

---
移除之前的所有约束，然后添加新约束的方法是：`mas_remakeConstraints`。

这里展开与收起的关键代码在这里.设置最大最小值，这样就不会超出我们预期的范围。

```
// 最大值为250
make.width.height.lessThanOrEqualTo(@250);
    
// 最小值为90
make.width.height.greaterThanOrEqualTo(@90);
```

我们设置其固定的宽高，并且设置其优先级为最低，以保证我们所设置的最大最小值始终生效。

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
