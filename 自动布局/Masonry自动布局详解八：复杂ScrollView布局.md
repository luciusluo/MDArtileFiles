>#Masonry自动布局详解八：复杂ScrollView布局

---
说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

>#支持原创，请[阅读原文](http://www.henishuo.com/masonry-complex-scrollview-layout/)

#效果图

---
本节详解`Masonry`的循环创建视图，并且可以展开与收缩的用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/4A4268AF-B930-43EE-8FE2-9EC05F975F59.jpg)

当我们点击某一行时，可以展开：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/D036FB39-C202-46EE-BC14-C03D254B0622.jpg)

#核心代码

---

看下代码：

```
@interface ScrollViewComplexController ()

@property (nonatomic, strong) UIScrollView *scrollView;
@property (nonatomic, strong) NSMutableArray *expandStates;

@end

@implementation ScrollViewComplexController

- (void)viewDidLoad {
  [super viewDidLoad];
 
  self.scrollView = [[UIScrollView alloc] init];
  self.scrollView.pagingEnabled = NO;
  [self.view addSubview:self.scrollView];
  self.scrollView.backgroundColor = [UIColor lightGrayColor];
  
  CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
  UILabel *lastLabel = nil;
  for (NSUInteger i = 0; i < 10; ++i) {
    UILabel *label = [[UILabel alloc] init];
    label.numberOfLines = 0;
    label.layer.borderColor = [UIColor greenColor].CGColor;
    label.layer.borderWidth = 2.0;
    label.text = [self randomText];
    label.userInteractionEnabled = YES;
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(onTap:)];
    [label addGestureRecognizer:tap];
    
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
      
      make.height.mas_equalTo(40);
    }];
    
    lastLabel = label;
    
    [self.expandStates addObject:[@[label, @(NO)] mutableCopy]];
  }
  
  [self.scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
    
    // 让scrollview的contentSize随着内容的增多而变化
    make.bottom.mas_equalTo(lastLabel.mas_bottom).offset(20);
  }];
}

- (NSMutableArray *)expandStates {
  if (_expandStates == nil) {
    _expandStates = [[NSMutableArray alloc] init];
  }
  
  return _expandStates;
}

- (UIColor *)randomColor {
  CGFloat hue = ( arc4random() % 256 / 256.0 );  //  0.0 to 1.0
  CGFloat saturation = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from white
  CGFloat brightness = ( arc4random() % 128 / 256.0 ) + 0.5;  //  0.5 to 1.0, away from black
  return [UIColor colorWithHue:hue saturation:saturation brightness:brightness alpha:1];
}

- (NSString *)randomText {
  CGFloat length = arc4random() % 150 + 5;
  
  NSMutableString *str = [[NSMutableString alloc] init];
  for (NSUInteger i = 0; i < length; ++i) {
    [str appendString:@"测试数据很长，"];
  }
  
  return str;
}

- (void)onTap:(UITapGestureRecognizer *)sender {
  UIView *tapView = sender.view;
  
  NSUInteger index = 0;
  for (NSMutableArray *array in self.expandStates) {
    UILabel *view = [array firstObject];
    
    if (view == tapView) {
      NSNumber *state = [array lastObject];
      
      if ([state boolValue] == YES) {
        [view mas_updateConstraints:^(MASConstraintMaker *make) {
          make.height.mas_equalTo(40);
        }];
      } else {
        UIView *preView = nil;
        UIView *nextView = nil;
        
        if (index - 1 < self.expandStates.count && index >= 1) {
          preView = [[self.expandStates objectAtIndex:index - 1] firstObject];
        }
        
        if (index + 1 < self.expandStates.count) {
           nextView = [[self.expandStates objectAtIndex:index + 1] firstObject];
        }

        [view mas_remakeConstraints:^(MASConstraintMaker *make) {
          if (preView) {
            make.top.mas_equalTo(preView.mas_bottom).offset(20);
          } else {
            make.top.mas_equalTo(20);
          }
          
          make.left.mas_equalTo(15);
          make.right.mas_equalTo(self.view).offset(-15);
        }];
        
        if (nextView) {
          [nextView mas_updateConstraints:^(MASConstraintMaker *make) {
            make.top.mas_equalTo(view.mas_bottom).offset(20);
          }];
        }
      }
      
      [array replaceObjectAtIndex:1 withObject:@(!state.boolValue)];
      
      [self.view setNeedsUpdateConstraints];
      [self.view updateConstraintsIfNeeded];

      [UIView animateWithDuration:0.35 animations:^{
        [self.view layoutIfNeeded];
      } completion:^(BOOL finished) {

      }];
      break;
    }
    
    index++;
  }
}

@end
```

#讲解

---
当我们要收起的时候，只是简单地设置其高度的约束为`40`，但是当我们要展开时，实现起来就相对麻烦了。因为我们需要重新添加约束，要重新给所点击的视图添加约束，就需要知道前一个依赖视图和后一个依赖视图的约束，以便将相关联的都更新约束。

当我们更新所点击的视图时，我们通过判断是否有前一个依赖视图来设置顶部约束：

```
if (preView) {
    make.top.mas_equalTo(preView.mas_bottom).offset(20);
} else {
    make.top.mas_equalTo(20);
}
```

除了这个之外，我们也需要更新后一个视图的约束，因为我们对所点击的视图调用了`mas_remakeConstraints`方法，就会移除其之前所添加的所有约束，所以我们必须重新将后者对当前点击的视图的依赖重新添加上去：

```
if (nextView) {
  [nextView mas_updateConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(view.mas_bottom).offset(20);
  }];
}
```

#源代码

---
大家可以到笔者的`github`下载源代码：[https://github.com/CoderJackyHuang/MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

> 温馨提示：本节所讲内容对应于`ScrollViewComplexController`中的内容

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**
