#引言

今天抽时间教大家实现一个非常常见的效果。跟着笔者一起来学习，实现起来非常简单，只需要几行代码即可！

在很多的App中经常有上下滚动时用户头像也跟着变化，而用户头像是放在系统的导航条上的。可能有朋友们尝试过自定义导航view，其实没有必要，直接使用系统自带的导航即可！

通过本篇文章，您将学习以下知识点：

* 如何分析实现原理
* 如何实现缩放效果
* 如何将计算缩放系数

#效果图

在开始讲解原理之前，还是先上效果图。有图有真相，才能帮助大家阅读，提升阅读的效果。如下图所示，在往下滚动时，在一定范围内会放大头像，但是不会放得过大；在往上滚动时，在一定范围内会缩小头像，但是不会缩小得过小：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/scale.gif)


#实现原理

从效果图可以看到以下几点：

1. 向上移动头像会缩小，但是有下限
2. 向下移动头像会放大，但是有上限
3. 头像的起点y始终不变

所以，我们首先要知道如何缩放控件，也就是使用transform来实现。然后每次都需要更新头像的y坐标，以保证y值不变。既然缩小有下限，放大有上限，所以我们应该设置一个最小缩放系数及最大缩放系数。

要设置最小/最大缩放系数，我们就需要计算出来，但是如何计算呢？其实挺简单的，我们只需要设置下拉或者上拉需要处理缩放的最大距离，就可以计算出来了。

计算放大系数： MIN(1.5, 1 - offsetY / 300);  
计算缩小系数： MAX(0.45, 1 - offsetY / 300);  

假设正常状态下用户头像的大小为70，当放大到最大时，不得超过105；当缩小到最小时，不得小于31.5.则这个最大倍数1.5就是我们期望用户头像可放大的最大值除以正常状态下的值，即105/70.0=1.5。同样，最小倍数0.45计算公式为：31.5 / 70.0 = 0.45.

为了保证在缩放过程中，y坐标不变，那么就需要动态地更新y坐标，也就是在缩放时，将y坐标固定。

#代码实现

首先，我们给导航条添加一个titleView：

```
UIView *titleView = [[UIView alloc] init];
self.navigationItem.titleView = titleView;

self.headerImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"head.jpg"]];
self.headerImageView.layer.cornerRadius = 35;
self.headerImageView.layer.masksToBounds = YES;
self.headerImageView.frame = CGRectMake(0, 0, 70, 70);
// 这一句非常重要，保证用户头像水平居中
self.headerImageView.center = CGPointMake(titleView.center.x, 0);
[titleView addSubview:self.headerImageView];
```

这里为什么不直接将headerImageView作为titleView呢？因为titleView会自动被系统给设置大小了，而我们的头像是固定大小的，可以可自由调整的，因此我们只能作为一个单独的控件放在titleView上。

接下来最关键的代码就是在滚动时处理缩放用户头像的了：

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
  CGFloat offsetY = scrollView.contentOffset.y + scrollView.contentInset.top;
  
  CGFloat scale = 1.0;
  // 放大
  if (offsetY < 0) {
    // 允许下拉放大的最大距离为300
    // 1.5是放大的最大倍数，当达到最大时，大小为：1.5 * 70 = 105
    // 这个值可以自由调整
    scale = MIN(1.5, 1 - offsetY / 300);
  } else if (offsetY > 0) { // 缩小
    // 允许向上超过导航条缩小的最大距离为300
    // 为了防止缩小过度，给一个最小值为0.45，其中0.45 = 31.5 / 70.0，表示
    // 头像最小是31.5像素
    scale = MAX(0.45, 1 - offsetY / 300);
  }
  
  self.headerImageView.transform = CGAffineTransformMakeScale(scale, scale);

  // 保证缩放后y坐标不变
  CGRect frame = self.headerImageView.frame;
  frame.origin.y = -self.headerImageView.layer.cornerRadius / 2;
  self.headerImageView.frame = frame;
}
```

#小结

阅读完本篇文章，笔者相信大家已经可以掌握了，并可以自己动手实现了！算起来只需要几行代码就可以实现这种动效了，是不是很简单呢？

#源代码

本篇文章对应的源代码下载地址：[NavigationBarScaleViewDemo](https://github.com/CoderJackyHuang/NavigationBarScaleViewDemo)

