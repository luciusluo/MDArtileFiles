>#Masonry自动布局详解九：scrollview实战场景

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

>#支持原创，请[阅读原文](http://www.henishuo.com/masonry-scrollview-use-scene/)

#提示

---
在`ios 6.0`上不能给tableview的`tableHeaderView`和`tableFooterView`添加约束，会`crash`。如果不用兼容到`iOS 6.0`，那就放心地使用了吧！

笔者所维护的项目都要求支持到`iOS 6.0`的，因此笔者对于使用`tableHeaderView`和`tableFooterView`的时候，就使用纯`frame`实现的。本教程本来是笔者希望寻找一种解决`iOS6.0`上使用`tableHeaderView`和`tableFooterView`上添加约束会崩溃的方案。找到了一种解决防止其崩溃的方案，但是会影响`tableview`的刷新代理方法，因此方案不成立。

#效果图

---
先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/B91DCC6D-8170-4FDE-8F5A-8EE3F25D0CD9.jpg)

#核心代码

---

看下代码：

```
@interface HeaderFooterViewController ()

@property (nonatomic, strong) UIScrollView *scrollView;

@end

@implementation HeaderFooterViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.scrollView = [[UIScrollView alloc] init];
  [self.view addSubview:self.scrollView];
  [self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
  }];
  
  UIView *headerView = [[UIView alloc] init];
  [self.scrollView addSubview:headerView];
  
  UIImageView *imgView = [[UIImageView alloc] init];
  [headerView addSubview:imgView];
  imgView.backgroundColor = [UIColor greenColor];
  imgView.layer.cornerRadius = 50;
  imgView.layer.masksToBounds = YES;
  imgView.layer.borderWidth = 0.5;
  imgView.layer.borderColor = [UIColor redColor].CGColor;
  
  CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
  
  UILabel *tipLabel = [[UILabel alloc] init];
  tipLabel.text = @"这里是提示信息，通常会比较长，可能会超过两行。为了适配6.0，我们需要指定preferredMaxLayoutWidth，但是要注意，此属性一旦设置，不是只在6.0上生效，任意版本的系统的都有作用，因此此值设置得一定要准备，否则计算结果会不正确。我们一定要注意，不能给tableview的tableHeaderView和tableFooterView添加约束，在6.0及其以下版本上会crash，其它版本没有";
  tipLabel.textAlignment = NSTextAlignmentCenter;
  tipLabel.textColor = [UIColor blackColor];
  tipLabel.backgroundColor = [UIColor clearColor];
  tipLabel.numberOfLines = 0;
  tipLabel.preferredMaxLayoutWidth = screenWidth - 30;
  [headerView addSubview:tipLabel];

  UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
  [button setTitle:@"Show detail" forState:UIControlStateNormal];
  [button setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
  [button setBackgroundColor:[UIColor blueColor]];
  button.layer.cornerRadius = 6;
  button.clipsToBounds = YES;
  button.layer.masksToBounds = YES;
  
  [headerView addSubview:button];

  [headerView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.top.mas_equalTo(0);
    make.width.mas_equalTo(self.view);
    make.bottom.mas_equalTo(button.mas_bottom).offset(60).priorityLow();
    make.height.greaterThanOrEqualTo(self.view);
  }];
  
  [imgView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(80);
    // 这里不能使用headerView，因为我们不给headerView添加约束
    //make.centerX.mas_equalTo(headerView);
    make.centerX.mas_equalTo(self.scrollView);
    make.width.height.mas_equalTo(100);
  }];
  
  [tipLabel mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.mas_equalTo(15);
    make.right.mas_equalTo(-15);
    make.top.mas_equalTo(imgView.mas_bottom).offset(40);
  }];
  [button mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_greaterThanOrEqualTo(tipLabel.mas_bottom).offset(80);
    make.left.mas_equalTo(15);
    make.right.mas_equalTo(-15);
    make.height.mas_equalTo(45);
  }];
 
  [self.scrollView mas_updateConstraints:^(MASConstraintMaker *make) {
    make.bottom.mas_equalTo(button.mas_bottom).offset(80).priorityLow();
    make.bottom.mas_greaterThanOrEqualTo(self.view);
  }];
}

@end
```

#讲解

---
这里只是说说如何设置约束以保证按钮的位置始终是有一定距离的：

```
make.bottom.mas_equalTo(button.mas_bottom).offset(80).priorityLow();
make.bottom.mas_greaterThanOrEqualTo(self.view);
```

我们设置了`scrollview`的`bottom`为`button`的底部`+80`个像素，但是其优先级为最低。然后又设置了`scrollview`的`bottom`要大于或者等于`self.view`，也就是说`contentSize.height`至少为屏高。

>温馨提示：对于`UILabel`，如果需要兼容到`iOS6.0`，一定要设置`preferredMaxLayoutWidth`属性值，并且其值一定要与约束所生成的宽一样，否则就会出现显示不完全的问题。因此这个，笔者也创造了好几个`bug`。

#源代码

---
大家可以到笔者的`github`下载源代码：[https://github.com/CoderJackyHuang/MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

> 温馨提示：本节所讲内容对应于`ScrollViewComplexController`中的内容

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**
