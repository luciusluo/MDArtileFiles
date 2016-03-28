#概述

UICollectionView真的好强大，今天我们来研究一下这种很常见的卡片动画效果是如何实现了。本篇不能太深入地讲解，因为笔者也是刚刚摸索出点眉目，但是并没有深刻地理解。如果在讲解过程中，出现不对的地方，请及时反馈。

#效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/03/carddemo.gif)

#重写API

```
// 我们必须重写此方法，指定布局大小
// 每次layout invalidated或者重新query布局信息时，会调用
- (void)prepareLayout;

// 用于决定布局信息
// 我们必须重写它来实现布局信息
- (nullable NSArray<__kindof UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect; 

// 重写它来布局信息
- (nullable UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath;
```

还有一个非常关键的API，必须重写：

```
// return YES to cause the collection view to requery the layout for geometry information
// 当重新查询布局信息时，就会调用此API。要设置为YES，才能实现自定义布局。
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds; 
```

#自定义布局

```
//
//  HYBCardFlowLayout.m
//  CollectionViewDemos
//
//  Created by huangyibiao on 16/3/26.
//  Copyright © 2016年 huangyibiao. All rights reserved.
//

#import "HYBCardFlowLayout.h"

@interface HYBCardFlowLayout ()

@property (nonatomic, strong) NSIndexPath *mainIndexPath;
@property (nonatomic, strong) NSIndexPath *willMoveToMainIndexPath;

@end

@implementation HYBCardFlowLayout

- (void)prepareLayout {
  CGFloat inset = 32;
  self.itemSize = CGSizeMake(self.collectionView.frame.size.width - 2 * inset,
                             self.collectionView.frame.size.height * 3 / 4);
  self.sectionInset = UIEdgeInsetsMake(0, inset, 0, inset);
  self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
  
  [super prepareLayout];
}

#pragma mark - Override
- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds {
  return YES;
}

- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath {
  UICollectionViewLayoutAttributes *attribute = [super layoutAttributesForItemAtIndexPath:indexPath];
  
  [self setTransformForLayoutAttributes:attribute];
  
  return attribute;
}

- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect {
  NSArray *attributesSuper = [super layoutAttributesForElementsInRect:rect];
  // 一定要深复制一份，不能修改父类的属性，不然会有很多error打印出来
  NSArray *attributes = [[NSArray alloc] initWithArray:attributesSuper copyItems:YES];
  
  NSArray *visibleIndexPaths = [self.collectionView indexPathsForVisibleItems];
  
  if (visibleIndexPaths.count <= 0) {
    return attributes;
  } else if (visibleIndexPaths.count == 1) {
    self.mainIndexPath = [visibleIndexPaths firstObject];
    self.willMoveToMainIndexPath = nil;
  } else if (visibleIndexPaths.count == 2) {
    NSIndexPath *indexPath = [visibleIndexPaths firstObject];
    
    // 说明是往左滚动
    if (indexPath == self.mainIndexPath) {
      // 记录将要移进来的位置
      self.willMoveToMainIndexPath = visibleIndexPaths[1];
    } else {// 往右滚动
      self.willMoveToMainIndexPath = visibleIndexPaths[0];
      
      // 更新下一个成为main
      self.mainIndexPath = visibleIndexPaths[1];
    }
  }
  
  for (UICollectionViewLayoutAttributes *attribute in attributes) {
    [self setTransformForLayoutAttributes:attribute];
  }
  
  return attributes;
}

- (void)setTransformForLayoutAttributes:(UICollectionViewLayoutAttributes *)attribute {
  UICollectionViewCell *cell = [self.collectionView cellForItemAtIndexPath:attribute.indexPath];
  
  if (self.mainIndexPath && attribute.indexPath.section == self.mainIndexPath.section) {
    attribute.transform3D = [self tranformForView:cell];
  } else if (self.willMoveToMainIndexPath && attribute.indexPath.section == self.willMoveToMainIndexPath.section) {
    attribute.transform3D = [self tranformForView:cell];
  }
}

- (CATransform3D)tranformForView:(UICollectionViewCell *)view {
  // cell的起始偏移
  CGFloat w = self.collectionView.frame.size.width;
  CGFloat offset = [self.collectionView indexPathForCell:view].section * w;
  
  // 当前偏移
  CGFloat currentOffset = self.collectionView.contentOffset.x;
  
  // 计算偏移angle
  CGFloat angle = (currentOffset - offset) / w;
  
  CATransform3D t = CATransform3DIdentity;
  t.m34 = 1.0 / -500;
  
  if (currentOffset - offset >= 0) {
    t = CATransform3DRotate(t, angle, 1, 1, 0);
  } else {
    t = CATransform3DRotate(t, angle, -1, 1, 0);
  }
  
  return t;
}

@end
```

这里主要是要处理旋转。然后要处理切换cell的attribute设置。mainIndexPath属性用于记录当前显示的cell的位置。willMoveToMainIndexPath记录将要出现的cell的位置。


#结尾

这里在慢慢切换时，效果是挺好的，但是如果快速切换卡片，你会发现会有一点点不好之处，就是下一个cell突然出现的。

#源代码

[CollectionViewDemos](https://github.com/CoderJackyHuang/CollectionViewDemos)

**提示：**本篇文章的demo对应于工程中的**Demo2-CardLayout**分组。




