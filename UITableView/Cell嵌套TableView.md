#前言

最近有很多小伙伴们都问我有没有cell里面再嵌套tableview的demo，老说不知道怎么做，不知道怎么计算高度啊。其实这个很简单的，昨天晚上正好有点时间，写了这个demo。

本篇文章只讲如何在cell中嵌套UITableView，只是粗浅知识点，只教大家基本的如何去计算UITableview的高度。这里模拟评论写了个demo，增加或者删除一条评论都可以马上更新，得到正确的显示。

#效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/03/celltableview.gif)

这里使用了100条数据，但是界面并不卡。这里使用了笔者所开源的自动计算cell的高度来计算行高的，而且这是带缓存功能的。用于计算的cell是重用的，因此效率是非常高的。

详细使用教程，可阅读[开源HYBMasonryAutoCellHeight自动计算行高](http://www.henishuo.com/masonry-cell-height-auto-calculate/)


#实现思路

笔者尝试画了一张图，尽量反映出实现的思路：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160302-0@2x.png)


外层UITableView通过HYBMasonryAutoCellHeight计算cell的高度并缓存起来，而cell中所嵌套的UITableView（显示评论信息的表格）也是通过HYBMasonryAutoCellHeight来计算cell的高度并缓存。当评论TableView增加或者删除一条数据时，通过代理反馈到外层TableView，然后重装计算行高并更新缓存。

#数据建模

这里使用了两个模型类HYBTestModel是外层UITableView的数据源模型，而HYBCommentModel是评论UITableView的数据源模型。

###HYBTestModel

```
@interface HYBTestModel : NSObject

@property (nonatomic, copy) NSString *uid;
@property (nonatomic, copy) NSString *title;
@property (nonatomic, copy) NSString *desc;
@property (nonatomic, copy) NSString *headImage;

// 评论数据源
@property (nonatomic, strong) NSMutableArray *commentModels;

// 因为评论是动态的，因此要标识是否要更新缓存
@property (nonatomic, assign) BOOL shouldUpdateCache;

@end
```

其中，uid必须保证唯一，它是用来缓存高度的唯一标识符，通常model的id作为uniqueKey。增加shouldUpdateCache属性的目的是在增加或者删除评论时，以识别是否需要重新计算行高并刷新对应的缓存高度。

###HYBCommentModel

```
@interface HYBCommentModel : NSObject

@property (nonatomic, copy) NSString *cid;
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *reply;
@property (nonatomic, copy) NSString *comment;

@end
```

这里的cid是指评论id，它是作为缓存行高的key。


#外层HYBTestCell

这个是外层UITableView所需要使用的cell，它里面会嵌套着评论的UITableView。当评论内容发生变化时，通过代理来实现数据的刷新，行高的重新计算与缓存。

```
@class HYBTestModel;

@protocol HYBTestCellDelegate <NSObject>

- (void)reloadCellHeightForModel:(HYBTestModel *)model atIndexPath:(NSIndexPath *)indexPath;

@end

@interface HYBTestCell : UITableViewCell

@property (nonatomic, weak) id delegate;

- (void)configCellWithModel:(HYBTestModel *)model indexPath:(NSIndexPath *)indexPath;

@end
```

当我们配置其数据时，我们要计算出评论的tablview的高度。这里是通过HYBMasonryAutoCellHeight这个开源库来实现的。当配置数据时，通过遍历所有的评论模型，然后计算cell的行高，累加起来就是Tablview的高度，然后刷新tableview的约束。这样就可以正常地计算出整个外部cell的高度了。

```
- (void)configCellWithModel:(HYBTestModel *)model indexPath:(NSIndexPath *)indexPath {
  self.indexPath = indexPath;
  
  self.titleLabel.text = model.title;
  self.descLabel.text = model.desc;
  self.headImageView.image = [UIImage imageNamed:model.headImage];
  
  self.testModel = model;
  CGFloat tableViewHeight = 0;
  for (HYBCommentModel *commentModel in model.commentModels) {
    CGFloat cellHeight = [HYBCommentCell hyb_heightForTableView:self.tableView config:^(UITableViewCell *sourceCell) {
      HYBCommentCell *cell = (HYBCommentCell *)sourceCell;
      [cell configCellWithModel:commentModel];
    } cache:^NSDictionary *{
      return @{kHYBCacheUniqueKey : commentModel.cid,
               kHYBCacheStateKey : @"",
               kHYBRecalculateForStateKey : @(NO)};
    }];
    tableViewHeight += cellHeight;
  }
  
  [self.tableView mas_updateConstraints:^(MASConstraintMaker *make) {
    make.height.mas_equalTo(tableViewHeight);
  }];
  self.tableView.dataSource = self;
  self.tableView.delegate = self;
  [self.tableView reloadData];
}
```

###计算评论cell的高度

因为评论cell的内容不会改变，要么被删除了，要么就是添加新的评论，因此不需要清除缓存，将
kHYBRecalculateForStateKey对应的值设置为NO即可。因为我们在上面一步配置外部cell的数据的时候，已经调用过下面计算行高的方法来计算了并且会自动缓存起来，所以这一步的调用其实只是直接获取缓存的高度，因为效率是非常高的。

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  HYBCommentModel *model = [self.testModel.commentModels objectAtIndex:indexPath.row];
  
  return [HYBCommentCell hyb_heightForTableView:self.tableView config:^(UITableViewCell *sourceCell) {
    HYBCommentCell *cell = (HYBCommentCell *)sourceCell;
    [cell configCellWithModel:model];
  } cache:^NSDictionary *{
    return @{kHYBCacheUniqueKey : model.cid,
             kHYBCacheStateKey : @"",
             kHYBRecalculateForStateKey : @(NO)};
  }];
}
```

###增加、删除评论

选中某条评论的时候，就增加一条评论，然后通过代理反馈到控制器类刷新数据。同样，当删除一条评论的时候，也通过代理反馈到控制器类，刷新数据：

```
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
  [tableView deselectRowAtIndexPath:indexPath animated:YES];
  
  // 添加一条数据
  HYBCommentModel *model = [[HYBCommentModel alloc] init];
  model.name = @"标哥";
  model.reply = @"标哥的技术博客";
  model.comment = @"哈哈，我被点击后自动添加了一条数据的，不要在意我~";
  model.cid = [NSString stringWithFormat:@"commonModel%ld",  self.testModel.commentModels.count + 1];
  [self.testModel.commentModels addObject:model];
  
  if ([self.delegate respondsToSelector:@selector(reloadCellHeightForModel:atIndexPath:)]) {
    self.testModel.shouldUpdateCache = YES;
    [self.delegate reloadCellHeightForModel:self.testModel atIndexPath:self.indexPath];
  }
}

- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath {
  return YES;
}

- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath {
  if (editingStyle == UITableViewCellEditingStyleDelete) {
    [self.testModel.commentModels removeObjectAtIndex:indexPath.row];
    
    if ([self.delegate respondsToSelector:@selector(reloadCellHeightForModel:atIndexPath:)]) {
      self.testModel.shouldUpdateCache = YES;
      [self.delegate reloadCellHeightForModel:self.testModel atIndexPath:self.indexPath];
    }
  }
}
```

#控制器刷新数据

当增加或者删除一条评论的时候，testModel会将shouldUpdateCache设置为YES,以便在reload对应的某一行之后，行高可以重新计算并更新缓存的高度而不是使用之前所缓存起来的高度：

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  HYBTestModel *model = [self.datasource objectAtIndex:indexPath.row];
  
  CGFloat h = [HYBTestCell hyb_heightForTableView:tableView config:^(UITableViewCell *sourceCell) {
    HYBTestCell *cell = (HYBTestCell *)sourceCell;
    [cell configCellWithModel:model indexPath:indexPath];
  } cache:^NSDictionary *{
    NSDictionary *cache = @{kHYBCacheUniqueKey : model.uid,
             kHYBCacheStateKey  : @"",
             kHYBRecalculateForStateKey : @(model.shouldUpdateCache)};
    model.shouldUpdateCache = NO;
    return cache;
  }];

  return h;
}

#pragma mark - HYBTestCellDelegate
- (void)reloadCellHeightForModel:(HYBTestModel *)model atIndexPath:(NSIndexPath *)indexPath {
  [self.tableView reloadRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationFade];
}
```

#源代码

大家可以到笔者的GITHUB下载Demo运行起来看看效果，代码中对于防止反复计算问题是没有处理的，为了demo的足够简单，不添加任何控制逻辑。在真正开发中，要想性能更高，需要自己处理~

下载地址：[CoderJackyHuang：CellEmbedTableView](https://github.com/CoderJackyHuang/CellEmbedTableView.git)

#最后

给大家推荐笔者所开源的自动计算行高的库，使用起来是很方便的！

* [开源HYBMasonryAutoCellHeight自动计算行高](http://www.henishuo.com/masonry-cell-height-auto-calculate/)

欢迎大家star，若有问题及时反馈与作者！

#阅读原文

文章内容更新只会在原文处更新，请阅读原文：[http://www.henishuo.com/ios-cell-embed-tableview/](http://www.henishuo.com/ios-cell-embed-tableview/)