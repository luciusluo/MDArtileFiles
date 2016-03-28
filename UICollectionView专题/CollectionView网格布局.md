#概述

说句老实话，UICollectionView真的太强大了，而且要掌握高级部分是相当困难的。至少笔者是这么认为的，如果觉得自己比较厉害，可以轻而易举地掌握UICollectionView的使用的，希望可以总结点经验！

本篇文章是在练习如何使用UICollectionView进行网格布局。网格布局是非常常见的UI布局，在很多的App中都这么设计过。本篇文章只讲如何实现风格布局，demo中有一些是优化方面的，会单独写一篇文章讲解如何优化网格布局中的图片。

#实现效果

![image](http://www.henishuo.com/wp-content/uploads/2016/03/griddemo.gif)

#FlowLayout

本demo中需要使用到UICollectionViewFlowLayout，因此先讲讲这个类：

```
// iOS6.0以后才有的
NS_CLASS_AVAILABLE_IOS(6_0) @interface UICollectionViewFlowLayout : UICollectionViewLayout

// 行之间的最小间距
@property (nonatomic) CGFloat minimumLineSpacing;
// item之间的最小间距
@property (nonatomic) CGFloat minimumInteritemSpacing;

// 如果cell的大小是固定的，应该直接设置此属性，就不用实现代理
@property (nonatomic) CGSize itemSize;

// 这是8.0后才能使用，评估item的大小
@property (nonatomic) CGSize estimatedItemSize NS_AVAILABLE_IOS(8_0); 

// 支持两种滚动方向，水平滚动和竖直功能
// 因此不要再想要使用横向tableview，直接使用collectionview就okb
@property (nonatomic) UICollectionViewScrollDirection scrollDirection;

// header参考大小
@property (nonatomic) CGSize headerReferenceSize;
// footer参考大小
@property (nonatomic) CGSize footerReferenceSize;
// section的inset，用于设置与上、左、底、右的间隔
@property (nonatomic) UIEdgeInsets sectionInset;

// 9.0以后才有的属性，用于设置header/footer与tableview的section效果一样。
// 可以悬停
@property (nonatomic) BOOL sectionHeadersPinToVisibleBounds NS_AVAILABLE_IOS(9_0);
@property (nonatomic) BOOL sectionFootersPinToVisibleBounds NS_AVAILABLE_IOS(9_0);

@end
```

这个类是用于决定UICollectionView的item的布局的。

#创建CollectionView

```
// 创建UI布局
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];

// 设置成固定的大小


layout.itemSize = CGSizeMake((kScreenWidth - 30) / 2, (kScreenWidth - 30) / 2 + 20);

// 行间距最小设置为10
layout.minimumLineSpacing = 10;
// 列间距最小设置为10
layout.minimumInteritemSpacing = 10;
  
self.collectionView = [[UICollectionView alloc] initWithFrame:self.view.bounds
                                       collectionViewLayout:layout];
[self.view addSubview:self.collectionView];

// 注册cell
[self.collectionView registerClass:[HYBGridCell class]
      forCellWithReuseIdentifier:cellIdentifier];
// 设置代理
self.collectionView.delegate = self;
self.collectionView.backgroundColor = [UIColor blackColor];
// 设置数据源代理
self.collectionView.dataSource = self;
```

#实现代理 

```
#pragma mark - UICollectionViewDataSource & UICollectionViewDelegate
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView
                  cellForItemAtIndexPath:(NSIndexPath *)indexPath {
  HYBGridCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:cellIdentifier
                                                                         forIndexPath:indexPath];
  HYBGridModel *model = self.datasource[indexPath.item];
  [cell configCellWithModel:model];
  
  return cell;
}

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section {
  return self.datasource.count;
}
```

#HYBGridCell

创建一个对应的cell，这里呢实现只是简单的添加两个控件而已：

```
- (instancetype)initWithFrame:(CGRect)frame {
  if (self = [super initWithFrame:frame]) {
    self.imageView = [[UIView alloc] init];
    self.imageView.frame = CGRectMake(0, 0, self.frame.size.width, self.frame.size.width);
    [self.contentView addSubview:self.imageView];
    
    self.titleLabel = [[UILabel alloc] init];
    self.titleLabel.frame = CGRectMake(0, self.frame.size.height - 20, self.frame.size.width, 20);
    self.titleLabel.numberOfLines = 0;
    self.titleLabel.font = [UIFont systemFontOfSize:16];
    self.titleLabel.textColor = [UIColor whiteColor];
    self.titleLabel.backgroundColor = [UIColor blackColor];
    self.titleLabel.layer.masksToBounds = YES;
    [self.contentView addSubview:self.titleLabel];
  }
  
  return self;
}
```

#结尾

Demo中有些代理是用于在讲优化那篇文章再讲，这里没有抛出来。本篇的目的就是学习如何布局网格。

#源代码

[CollectionViewDemos](https://github.com/CoderJackyHuang/CollectionViewDemos)

**提示：**本篇文章的demo对应于工程中的Demo1-GridCollectionView分组。

