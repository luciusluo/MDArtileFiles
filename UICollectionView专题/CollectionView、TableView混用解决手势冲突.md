#前言

最近在重构某个模块，以后别人封装的所谓的基类就像一坨死一样，看见就恶心，相信同行的你们能够明白那种心情。为什么要重构？并不是真的因为它像一坨死，而是因为这个模块是用户使用最频繁的，而且出现了不少bug，最重要的是这bug还是p1级别的致命bug。

曾经经过了几天的压力测试都没有复现出来，但是用户却频繁反馈，这就是决定重构的原因了。重构的界面是这样的：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/IMG_0166.png)

当UICollectionView中的每个cell放的是一个controller.view而这个controller.view又放一个UITableVIew时，这时候将collectionView的滚动方向设置为横向就可以了。

但是，如果我们设置了bounces为YES,那么右滑返回手势就没有了，怎么办？

#实现思路

共使用了四个控制器类：

* ContentController：手势冲突当前所在的控制器，使用UICollectionView，每个cell对应于一个控制器的view
* SiteController1：标签一对应的控制器
* SiteController2：标签二对应的控制器
* SiteController3：标签三对应的控制器

#配置UICollectioView


```
// config collection view
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
layout.itemSize = CGSizeMake(kScreenWidth, kScreenHeight - 64 - tabViewHeight);
layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
layout.minimumLineSpacing = 0;
layout.minimumInteritemSpacing = 0;
self.collectionView = [[UICollectionView alloc] initWithFrame:CGRectMake(0, 64 + tabViewHeight, kScreenWidth, kScreenHeight - 64 - tabViewHeight)
                                       collectionViewLayout:layout];
[self.view addSubview:self.collectionView];
self.collectionView.backgroundColor = kWColor;
self.collectionView.pagingEnabled = YES;
self.collectionView.showsHorizontalScrollIndicator = NO;
  
[self.collectionView registerClass:[UICollectionViewCell class]
      forCellWithReuseIdentifier:kPatientCellIdentifier];
[self.collectionView registerClass:[UICollectionViewCell class]
      forCellWithReuseIdentifier:kUnreadCellIdentifier];
[self.collectionView registerClass:[UICollectionViewCell class]
      forCellWithReuseIdentifier:kAllCellIdentifier];
      
// 不能将bounces设置为NO，否则右滑返回手势就没有了
// self.collectionView.bounces = NO;
```

当我们滚动到标签三时，再滑动就会超出范围，此时会显示部分空白，这体验不太好，不希望可以再滑动了。同样，当滑动到标签一时，再右滑时，不希望显示空白部分，而是触发右滑返回手势。

#解决方案

解决方案就是实现UIScrollView的代理方法，当超出屏宽\*2时，限制在屏宽\*2的位置处。同样，当小于0时，就限制在0处，这样就解决了出现空白的问题。同时，这样就不会关闭用户响应，因此系统的右滑返回手势仍然可以触发。

经过这么一折腾，大家明白如何解决的了吗？

```
#pragma mark -- UIScrollViewDelegate
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
  // 限制不能超出屏宽*2
  if (scrollView.contentOffset.x > 2 * kScreenWidth) {
    [scrollView setContentOffset:CGPointMake(2 * kScreenWidth, 0)];
    return;
  }
  
  // 限制不能超过0
  if (scrollView.contentOffset.x < 0) {
    [scrollView setContentOffset:CGPointMake(0, 0)];
    return;
  }
  
  int itemIndex = (scrollView.contentOffset.x +
                   self.collectionView.hdf_width * 0.5) / self.collectionView.hdf_width;
  itemIndex = itemIndex % [self collectionView:self.collectionView numberOfItemsInSection:0];
  
  CGFloat x = scrollView.contentOffset.x - self.collectionView.hdf_width;
  NSUInteger index = fabs(x) / self.collectionView.hdf_width;
  CGFloat fIndex = fabs(x) / self.collectionView.hdf_width;
  
  if (fabs(fIndex - (CGFloat)index) <= 0.00001) {
    // ....切换按钮
  }
}
```

#最后

本篇文章没有demo，因为这是在公司项目中所写的，这里只是将简单的部分总结写出来，希望后来人可以快速填埋这个坑。

本篇文章所涉及的问题，也许与您所遇到的问题有些区别，但是希望本篇文章可以给您联想！！！

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！


