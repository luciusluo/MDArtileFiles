#前言


说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

#效果图

本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/basicanimated.gif)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码


看下代码：

```
@interface UpdateConstraintsController ()

@property (nonatomic, strong) UIButton *growingButton;
@property (nonatomic, assign) CGFloat scacle;

@end

@implementation UpdateConstraintsController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.growingButton = [UIButton buttonWithType:UIButtonTypeSystem];
  [self.growingButton setTitle:@"点我放大" forState:UIControlStateNormal];
  self.growingButton.layer.borderColor = UIColor.greenColor.CGColor;
  self.growingButton.layer.borderWidth = 3;
  
  [self.growingButton addTarget:self action:@selector(onGrowButtonTaped:) forControlEvents:UIControlEventTouchUpInside];
  [self.view addSubview:self.growingButton];
  self.scacle = 1.0;
  
  [self.growingButton mas_updateConstraints:^(MASConstraintMaker *make) {
    make.center.mas_equalTo(self.view);
    
    // 初始宽、高为100，优先级最低
    make.width.height.mas_equalTo(100 * self.scacle).priorityLow();
    // 最大放大到整个view
    make.width.height.lessThanOrEqualTo(self.view);
  }];
}

#pragma mark - updateViewConstraints
- (void)updateViewConstraints {
 [self.growingButton mas_updateConstraints:^(MASConstraintMaker *make) {
   make.center.mas_equalTo(self.view);
   
   // 初始宽、高为100，优先级最低
   make.width.height.mas_equalTo(100 * self.scacle).priorityLow();
   // 最大放大到整个view
   make.width.height.lessThanOrEqualTo(self.view);
 }];
  
  [super updateViewConstraints];
}

- (void)onGrowButtonTaped:(UIButton *)sender {
  self.scacle += 0.5;
  
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

更新约束的方法是：`mas_updateConstraints`。下面笔者一行行代码说明。

让控件始终居中显示：

```
make.center.mas_equalTo(self.view);
```

让控件的宽和高相等且设置其优先级最低。关于优先级，这里先不细讲，后续文章会专门讲解。

```
make.width.height.mas_equalTo(100 * self.scacle).priorityLow();
```

让控件的宽和高小于或者等于`self.view`的宽和高，因此这个控件的最多能放大到全屏幕。

```
make.width.height.lessThanOrEqualTo(self.view);
```

将这三行代码放在一起，形成的话就是：使控件与父视图始终保持居中，控件的宽和高最大不能超过屏幕，且控件的宽和高可以变化。由于我们设置了第二行的代码优先级为`priorityLow`，因此其优先级是最低的，所以就可以保证宽高不能超过屏幕。

其实，当我们将更新代码放到`updateViewConstraints`这个方法中时，我们在`viewDidLoad`方法中写的：

```
  [self.growingButton mas_updateConstraints:^(MASConstraintMaker *make) {
    make.center.mas_equalTo(self.view);
    
    // 初始宽、高为100，优先级最低
    make.width.height.mas_equalTo(100 * self.scacle).priorityLow();
    // 最大放大到整个view
    make.width.height.lessThanOrEqualTo(self.view);
  }];
```
就不需要了。这里写上去的目的只是想说明用与不用都没有关系。

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


大家可以到笔者的`github`下载源代码：[MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

温馨提示：本节所讲内容对应于`UpdateConstraintsController`中的内容。也随手给个star吧！

