#概述

突然想起小美到家App中的一个列表效果，反正正好最近在研究collectonview，正在拿它个效果来练练手。今天教大家如何实现竖直滚动的列表缩放式效果，体验果然提高了不少。

#实现效果

![image](http://www.henishuo.com/wp-content/uploads/2016/03/vcardscale.gif)

#缘由

10个月前的事了，那时候在一个很小的创业公司里做美容O2O的，然后研究了很多同行的App，看着人家的效果却不知如何入手，只怪当初太菜。

利用周末研究了一下，果然还是实现出来了。哈哈，时隔这么久还是不能忘记这份“耻辱”啊。不过上周末可不只是实现了这种效果哦，还有好几种效果的。大家可以看文章末尾的推荐阅读。

#实现思路

从效果图可以看到变化是，越是往中间滚动的item显示最大，越显眼。而越是往前面，或者越是后面的，反而显示越小，这样就形成了视觉差。

实现的思路就是通过重写在可见范围内的所有item的方法：

```
- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect
```

通过这个API可以获取到原始属性，然后利用公式来计算缩放。

#难点

如何计算缩放系数。这里实现的效果的公式如下：

```
CGFloat zoom = 1 + 0.15 * (1 - fabs(widthForScale));
```

其中，widthForScale是最难把握的系数。

如何计算widthForScale呢？方法是利用item的中心y与当前collectionview的contentOffset.y来计算：

```
CGFloat distance =  obj.center.y - fabs(offset) - self.itemSize.width;
CGFloat widthForScale = distance / self.itemSize.height;
```

其中，offset这么计算出来的：

```
CGRect visibleRect = CGRectZero;
visibleRect.origin = self.collectionView.contentOffset;
visibleRect.size = self.collectionView.frame.size;
  
CGFloat offset = CGRectGetMinY(visibleRect);
```

最大的难点是distance的计算，如何拿捏呢？这需要慢慢分析。

假设有三个item显示，那么第一个item的缩放要比第二个item（主显示的item）要小，然后第二个item的缩放要比第三个要大。

第二个item（主）比第一个多了一个item，而第三个又比第二个多了一个item，所以再减去一个item的宽就可以了。

#核心代码

```
- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect {
  NSArray *superAttributes = [super layoutAttributesForElementsInRect:rect];
  NSArray *attributes = [[NSArray alloc] initWithArray:superAttributes copyItems:YES];
  
  CGRect visibleRect = CGRectZero;
  visibleRect.origin = self.collectionView.contentOffset;
  visibleRect.size = self.collectionView.frame.size;
  
  CGFloat offset = CGRectGetMinY(visibleRect);
  
  [attributes enumerateObjectsUsingBlock:^(UICollectionViewLayoutAttributes *obj, NSUInteger idx, BOOL * _Nonnull stop) {
    CGFloat distance =  obj.center.y - fabs(offset) - self.itemSize.width;
    CGFloat widthForScale = distance / self.itemSize.height;
    
      CGFloat zoom = 1 + 0.15 * (1 - fabs(widthForScale));
      obj.transform3D = CATransform3DMakeScale(zoom, 1.0, 1.0);
  }];
 
  return attributes;
}
```

系数0.15自由调整。

#结尾

这里没有实现分页的特性，笔者还是尝试了的，不过因为一个item不是占满全屏，所以计算起来总是达不到笔者期望的效果。大家若能在笔者的基础上研究出来，请主动通知笔者，谢谢！

#源代码

[CollectionViewDemos](https://github.com/CoderJackyHuang/CollectionViewDemos)

**提示：**本篇文章的demo对应于工程中的**Demo4-CardLayout-vertical-scale**分组。


#推荐阅读

* [CollectionView网格布局](http://www.henishuo.com/collectionview-grid-layout/)
* [CollectionView旋转水平卡片布局](http://www.henishuo.com/collectiontion-card-layout/)
* [CollectionView缩放水平卡片布局](http://www.henishuo.com/collectionview-flowlayout-card-scale/)


