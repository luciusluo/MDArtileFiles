>#Masonry自动布局详解四：复合约束

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

>#支持原创，请[阅读原文](http://www.henishuo.com/masonry-total-constraints-update/)

#效果图

---
本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/AB54C6FE-4C8B-49AF-B42F-1D6D26B1E34A.jpg)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码

---

看下代码：

```
@interface CompositeController ()

@property (nonatomic, strong) NSMutableArray *viewArray;
@property (nonatomic, assign) BOOL isNormal;

@end

@implementation CompositeController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  UIView *lastView = self.view;
  for (int i = 0; i < 6; i++) {
    UIView *view = UIView.new;
    view.backgroundColor = [self randomColor];
    view.layer.borderColor = UIColor.blackColor.CGColor;
    view.layer.borderWidth = 2;
    [self.view addSubview:view];
    
    [view mas_makeConstraints:^(MASConstraintMaker *make) {
      make.edges.equalTo(lastView).insets(UIEdgeInsetsMake(20, 20, 20, 20));
    }];
    
    lastView = view;
    
    [self.viewArray addObject:view];
    
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self
                                                                          action:@selector(onTap)];
    [view addGestureRecognizer:tap];
  }
  self.isNormal = YES;
}

- (void)onTap {
    UIView *lastView = self.view;
  
  if (self.isNormal) {
    for (NSInteger i = self.viewArray.count - 1; i >= 0; --i) {
      UIView *itemView = [self.viewArray objectAtIndex:i];
      [itemView mas_remakeConstraints:^(MASConstraintMaker *make) {
       make.edges.equalTo(lastView).insets(UIEdgeInsetsMake(20, 20, 20, 20));
      }];
      
      [self.view bringSubviewToFront:itemView];
      lastView = itemView;
    }
  } else {
    for (NSInteger i = 0; i < self.viewArray.count; ++i) {
      UIView *itemView = [self.viewArray objectAtIndex:i];
      [itemView mas_remakeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(lastView).insets(UIEdgeInsetsMake(20, 20, 20, 20));
      }];
      
      [self.view bringSubviewToFront:itemView];
      lastView = itemView;
    }
  }
  
  [self.view setNeedsUpdateConstraints];
  [self.view updateConstraintsIfNeeded];
  
  [UIView animateWithDuration:0.5 animations:^{
    [self.view layoutIfNeeded];
  } completion:^(BOOL finished) {
    self.isNormal = !self.isNormal;
  }];
}

- (NSMutableArray *)viewArray {
  if (_viewArray == nil) {
    _viewArray = [[NSMutableArray alloc] init];
  }
  
  return _viewArray;
}

- (UIColor *)randomColor {
  CGFloat hue = ( arc4random() % 256 / 256.0 );  //  0.0 to 1.0
  CGFloat saturation = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from white
  CGFloat brightness = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from black
  
  return [UIColor colorWithHue:hue saturation:saturation brightness:brightness alpha:1];
}

@end
```

#讲解

---
移除之前的所有约束，然后添加新约束的方法是：`mas_remakeConstraints`。

我们的目标是点击时，将里面的往外面，外面的往里面，并且显示动画效果。其中，最关键的代码是：

```
[self.view bringSubviewToFront:itemView];
lastView = itemView;
```

通过循环获取各个视图，外变成里，里变成外，我们一定要记录相对于下一个`view`以更新下一个视图的相对边缘。


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

> 温馨提示：本节所讲内容对应于`CompositeController`中的内容

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**
