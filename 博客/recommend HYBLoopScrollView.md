>#推荐开源库：HYBLoopScrollView

---
App中不可或缺的广告轮播图组件，现在开源出来了，希望对大家有帮助！

![image](https://github.com/CoderJackyHuang/HYBLoopScrollView/blob/master/screen.png)

#有什么特性

---
用一个第三方库，首先需要了解这个三方库有什么特性，为什么值得使用它！

###特性1：无缝无限循环滚动

我相信每一个想要自己写这个无限滚动显示广告图片的开发者，都会遇到这么个问题：滚动到最后一张后，再切换到第一张时怎么动画效果这么难看呢？根本就是到末尾后就直接切换到第一张，因此效果很不友好。

`HYBLoopScrollView`就很好地解决了这个问题。这个库使用了`UICollectionView`的特性，很巧妙地实现了这个无限滚动的效果。

###特性2：直接使用block版本API

---
原来我也想使用别人的开源库，但是使用起来很困难，一大堆的`API`，维护起来太麻烦。因此，才决定自己写一套库来解决这个麻烦。

这里提供了两个创建控件的方法：

```
+ (instancetype)loopScrollViewWithFrame:(CGRect)frame imageUrls:(NSArray *)imageUrls;

+ (instancetype)loopScrollViewWithFrame:(CGRect)frame
                              imageUrls:(NSArray *)imageUrls
                           timeInterval:(NSTimeInterval)timeInterval
                              didSelect:(HYBLoopScrollViewDidSelectItemBlock)didSelect
                              didScroll:(HYBLoopScrollViewDidScrollBlock)didScroll;
```

看到连同`didSelect`参数和`didScroll`参数了吗？前者就是点击某个广告图片时的回调`block`，而后者就是滚动到某个广告时的回调，是不是很简单？

另外，还封装了定时器的`api`，可方便地暂停或继续开启：

```
/**
 *  Pause the timer. Usually you need to pause the timer when the view disappear.
 */
- (void)pauseTimer;

/**
 *  Start the timer immediately. If you has pause the timer, you may need to start 
 *  the timer again when the view appear.
 */
- (void)startTimer;
```

###特性3：支持cocoapods

---
说到第三方库，怎么能少了对`cocoapods`的支持呢？

当前维护的版本已经到了`version 2.1`，可通过下面的方法添加到`Podfile`中：

```
pod "HYBLoopScrollView", '~> 2.1'
```

#致谢

---
该开源库至今已经得到不少朋友的邮件反馈，才有了今天的版本。感谢所有支持我的朋友！！！

#源代码

---
如果不想使用`cocoapods`来安装，可以到github下载源代码，直接将`HYBLoopScrollView`文件夹拖到工程，不需要做任何配置！！！

下载地址：[https://github.com/CoderJackyHuang/HYBLoopScrollView](https://github.com/CoderJackyHuang/HYBLoopScrollView)