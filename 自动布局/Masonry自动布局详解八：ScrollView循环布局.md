#前言

说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

#效果图


本节详解`Masonry`的循环创建视图的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/ADCBFCEE-48F9-46FD-8351-A3968A99DBE6-e1458444091566.jpg)

#核心代码



看下代码：

```
@interface ScrollViewController ()

@property (nonatomic, strong) UIScrollView *scrollView;

@end

@implementation ScrollViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.scrollView = [[UIScrollView alloc] init];
  self.scrollView.pagingEnabled = NO;
  [self.view addSubview:self.scrollView];
  self.scrollView.backgroundColor = [UIColor lightGrayColor];

  CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
  UILabel *lastLabel = nil;
  for (NSUInteger i = 0; i < 20; ++i) {
    UILabel *label = [[UILabel alloc] init];
    label.numberOfLines = 0;
    label.layer.borderColor = [UIColor greenColor].CGColor;
    label.layer.borderWidth = 2.0;
    label.text = [self randomText];
    
    // We must preferredMaxLayoutWidth property for adapting to iOS6.0
    label.preferredMaxLayoutWidth = screenWidth - 30;
    label.textAlignment = NSTextAlignmentLeft;
    label.textColor = [self randomColor];
    [self.scrollView addSubview:label];
    
    [label mas_makeConstraints:^(MASConstraintMaker *make) {
      make.left.mas_equalTo(15);
      make.right.mas_equalTo(self.view).offset(-15);
      
      if (lastLabel) {
        make.top.mas_equalTo(lastLabel.mas_bottom).offset(20);
      } else {
        make.top.mas_equalTo(self.scrollView).offset(20);
      }
    }];
    
    lastLabel = label;
  }
  
  [self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
    
    // 让scrollview的contentSize随着内容的增多而变化
    make.bottom.mas_equalTo(lastLabel.mas_bottom).offset(20);
  }];
}

- (UIColor *)randomColor {
  CGFloat hue = ( arc4random() % 256 / 256.0 );  //  0.0 to 1.0
  CGFloat saturation = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from white
  CGFloat brightness = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from black
  return [UIColor colorWithHue:hue saturation:saturation brightness:brightness alpha:1];
}

- (NSString *)randomText {
  CGFloat length = arc4random() % 50 + 5;
  
  NSMutableString *str = [[NSMutableString alloc] init];
  for (NSUInteger i = 0; i < length; ++i) {
    [str appendString:@"测试数据很长，"];
  }
  
  return str;
}

@end
```

#讲解

对于循环创建，我们需要记录下一个视图所依赖的控件，这里使用了`lastLabel`来记录。

我们要想让`scrollview`的`contentSize`随内容的变化而变化，那么就我们一定要添加注意添加约束：

```
  [self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
    
    // 让scrollview的contentSize随着内容的增多而变化
    make.bottom.mas_equalTo(lastLabel.mas_bottom).offset(20);
  }];
```

对于`scrollview`和`tableview`，我们不能使用`bottom`来计算其高，因为这个属性对于`scrollview`和`tableview`来说，不用用来计算高度的，而是用于计算`contentSize.height`的。我们要想随内容而变化，以便可滚动查看，就必须设置`bottom`约束。

#源代码


大家可以到笔者的`github`下载源代码：[MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

**温馨提示：**本节所讲内容对应于`ScrollViewController`中的内容

