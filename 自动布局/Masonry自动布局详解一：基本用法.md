#前言


说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

#效果图


本节详解`Masonry`的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/876F3994-1B92-4EE1-A144-EF4D0A2B942C-e1458443262674.jpg)

我们这里要布局使这三个控件的高度一致，而绿色和红色的控件高度和宽度相待。

#核心代码



看下代码：

```
- (void)viewDidLoad {
  [super viewDidLoad];
  
  UIView *greenView = UIView.new;
  greenView.backgroundColor = UIColor.greenColor;
  greenView.layer.borderColor = UIColor.blackColor.CGColor;
  greenView.layer.borderWidth = 2;
  [self.view addSubview:greenView];
  
  UIView *redView = UIView.new;
  redView.backgroundColor = UIColor.redColor;
  redView.layer.borderColor = UIColor.blackColor.CGColor;
  redView.layer.borderWidth = 2;
  [self.view addSubview:redView];
  
  UIView *blueView = UIView.new;
  blueView.backgroundColor = UIColor.blueColor;
  blueView.layer.borderColor = UIColor.blackColor.CGColor;
  blueView.layer.borderWidth = 2;
  [self.view addSubview:blueView];
  
  // 使这三个控件等高
  CGFloat padding = 10;
  [greenView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(padding);
    make.left.mas_equalTo(padding);
    make.right.mas_equalTo(redView.mas_left).offset(-padding);
    make.bottom.mas_equalTo(blueView.mas_top).offset(-padding);
    // 三个控件等高
    make.height.mas_equalTo(@[redView, blueView]);
    // 红、绿这两个控件等高
    make.width.mas_equalTo(redView);
  }];
  
  [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.height.bottom.mas_equalTo(greenView);
    make.right.mas_equalTo(-padding);
    make.left.mas_equalTo(greenView.mas_right).offset(padding);
  }];
  
  [blueView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.height.mas_equalTo(greenView);
    make.bottom.mas_equalTo(-padding);
    make.left.mas_equalTo(padding);
    make.right.mas_equalTo(-padding);
  }];
}
```

#讲解


添加约束的方法是：`mas_makeConstraints`。我们可以看到，约束可以使用链式方式，使用方法很简单，看起来像一句话。

看这句话：`make.top.height.bottom.mas_equalTo(greenView)`，意思是：使我的顶部、高度和底部都与`greenView`的顶部、高度和底部相等。因此，只要`greenView`的约束添加好了，那么`redView`的顶部、高度和底部也就自动计算出来了。

大多时候，我们并不会将一句话完整地写出来，而是使用简写的方式。如：

```
make.right.mas_equalTo(-padding);
```

其完整写法为：

```
make.right.mas_equalTo(bludView.superView.mas_right).offset(-padding);
```

当我们是要与父控件相对约束时，可以省略掉父视图。注意，并不是什么时候都可以省略，只有约束是同样的才可以省略。比如，约束都是`right`才可以。如果是一个`left`一个是`right`，那么就不能省略了。


#源代码


大家可以到笔者的`github`下载源代码：[https://github.com/CoderJackyHuang/MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

温馨提示：本节所讲内容对应于`BasicController`中的内容

