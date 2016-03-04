#前言


还在手动计算`UITableViewCell`的行高吗？还在每次都因为需求变化一点就要大量调整`cell`的高度而烦恼吗？现在教大家如何通过`Masonry`的自动布局来实现自动计算`cell`的行高！！！

在`github`没有找到基于`Masonry`自动计算行高的库，倒是找到了使用`xib/storyboard`在添加约束来自动计算行高的库，如： `UITableView-FDTemplateLayoutCell`

本人非常推崇`Masonry`来实现代码的自动布局，因此项目中都是使用`Masonry`布局的，为了自动计算行高，决定写一个扩展，以达到自动计算的效果，如此一来，开发者不用再关心那些非固定行高的`cell`的动态计算问题了。

#设置关键依赖


要想自动计算出`cell`的行高，我们还需要指定以哪个视图作为`cell`的最后一个视图，比如我们最后要添加一条线，我们可以以这条线作为`hyb_lastViewInCell`，如果这条线还需要距离底部一定距离，那么可以设置`hyb_bottomOffsetToCell`：

```
/**
 * 必传设置的属性，也就是在cell中的contentView内最后一个视图，用于计算行高
 * 例如，创建了一个按钮button作为在cell中放到最后一个位置，则设置为：self.hyb_lastVieInCell = button;
 * 即可。
 * 默认为nil，如果在计算时，值为nil，会crash
 */
@property (nonatomic, strong) UIView *hyb_lastViewInCell;

/**
 * 可选设置的属性，默认为0，表示指定的hyb_lastViewInCell到cell的bottom的距离
 * 默认为0.0
 */
@property (nonatomic, assign) CGFloat hyb_bottomOffsetToCell;
```

#计算行高API

要计算行高，只需要在`UITableView`的计算行高的代理方法中调用此API即可，下面的API是不缓存行高的：

```
/**
 * 通过此方法来计算行高，需要在config中调用配置数据的API
 *
 * @param tableView 必传，为哪个tableView缓存行高
 * @param config     必须要实现，且需要调用配置数据的API
 *
 * @return 计算的行高
 */
+ (CGFloat)hyb_heightForTableView:(UITableView *)tableView config:(HYBCellBlock)config;
```

若想要缓存行高，请使用此API：

```
/**
 *	@author 黄仪标, 16-01-22 23:01:56
 *
 *	此API会缓存行高
 *
 *	@param tableView 必传，为哪个tableView缓存行高
 *	@param config 必须要实现，且需要调用配置数据的API
 *	@param cache  返回相关key
 *
 *	@return 行高
 */
+ (CGFloat)hyb_heightForTableView:(UITableView *)tableView
                           config:(HYBCellBlock)config
                            cache:(HYBCacheHeight)cache;

// 其中，需要使用指定的key传值
/**
 *	@author 黄仪标, 16-01-22 21:01:09
 *
 *	唯一键，通常是数据模型的id，保证唯一
 */
FOUNDATION_EXTERN NSString *const kHYBCacheUniqueKey;

/**
 *	@author 黄仪标, 16-01-22 21:01:57
 *
 *	对于同一个model，如果有不同状态，而且不同状态下高度不一样，那么也需要指定
 */
FOUNDATION_EXTERN NSString *const kHYBCacheStateKey;

/**
 *	@author 黄仪标, 16-01-22 21:01:47
 *
 *	用于指定更新某种状态的缓存，比如当评论时，增加了一条评论，此时该状态的高度若已经缓存过，则需要指定来更新缓存
 */
FOUNDATION_EXTERN NSString *const kHYBRecalculateForStateKey;
```

#实现例子


效果图如下：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/cellheight.jpg)

### 创建cell
我们看下实现`-initWithStyle: reuseIdentifier:`方法，因为我们要自动计算`cell`行高会自动调用此方法，因此一定要实现此方法来布局：

```
- (nonnull instancetype)initWithStyle:(UITableViewCellStyle)style
                      reuseIdentifier:(nullable NSString *)reuseIdentifier {
  if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {
    self.mainLabel = [[UILabel alloc] init];
    [self.contentView addSubview:self.mainLabel];
    self.mainLabel.numberOfLines = 0;
    [self.mainLabel mas_makeConstraints:^(MASConstraintMaker *make) {
      make.left.mas_equalTo(15);
      make.top.mas_equalTo(20);
      make.right.mas_equalTo(-15);
      make.height.mas_lessThanOrEqualTo(80);
    }];
    CGFloat w = [UIScreen mainScreen].bounds.size.width;
    // 应该始终要加上这一句
    // 不然在6/6plus上就不准确了
    self.mainLabel.preferredMaxLayoutWidth = w - 30;
    
    self.descLabel = [[UILabel alloc] init];
    [self.contentView addSubview:self.descLabel];
    self.descLabel.numberOfLines = 0;
    [self.descLabel sizeToFit];
    [self.descLabel mas_makeConstraints:^(MASConstraintMaker *make) {
      make.left.mas_equalTo(15);
      make.right.mas_equalTo(-15);
      make.top.mas_equalTo(self.mainLabel.mas_bottom).offset(15);
    }];
    // 应该始终要加上这一句
    // 不然在6/6plus上就不准确了
    self.descLabel.preferredMaxLayoutWidth = w - 30;
    self.descLabel.userInteractionEnabled = YES;
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]
                                   initWithTarget:self
                                   action:@selector(onTap)];
    [self.descLabel addGestureRecognizer:tap];
    
    self.button = [UIButton buttonWithType:UIButtonTypeSystem];
    [self.contentView addSubview:self.button];
    [self.button setTitle:@"我是cell的最后一个" forState:UIControlStateNormal];
    [self.button setBackgroundColor:[UIColor greenColor]];
    [self.button mas_makeConstraints:^(MASConstraintMaker *make) {
      make.left.mas_equalTo(15);
      make.right.mas_equalTo(-15);
      make.height.mas_equalTo(45);
      make.top.mas_equalTo(self.descLabel.mas_bottom).offset(40);
    }];
    
    // 必须加上这句
    self.hyb_lastViewInCell = self.button;
    self.hyb_bottomOffsetToCell = 20;
    self.isExpandedNow = YES;
  }
  
  return self;
}
```

注意到这两行代码了吗：

```
self.hyb_lastViewInCell = self.button;
self.hyb_bottomOffsetToCell = 20;
```

先是设置哪个视图作为`cell`的最后一个视图，然后设置了最后一个参考视图与`cell`的底部的距离。其中，`self.hyb_lastViewInCell`属性是必须要设置的，否则会抛出异常。

###外部调用计算行高

在调用时，`config`传回来了`cell`对象，需要在调用处调用方法来配置好数据，才能正确地计算出`cell`的行高。通常是这样调用的：

```
- (CGFloat)tableView:(nonnull UITableView *)tableView heightForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
  HYBNewsModel *model = nil;
  if (indexPath.row < self.dataSource.count) {
    model = [self.dataSource objectAtIndex:indexPath.row];
  }
  
  NSString *stateKey = nil;
  if (model.isExpand) {
    stateKey = @"expanded";
  } else {
    stateKey = @"unexpanded";
  }
  
  return [HYBNewsCell hyb_heightForTableView:tableView config:^(UITableViewCell *sourceCell) {
    HYBNewsCell *cell = (HYBNewsCell *)sourceCell;
    // 配置数据
    [cell configCellWithModel:model];
  } cache:^NSDictionary *{
    return @{kHYBCacheUniqueKey: [NSString stringWithFormat:@"%d", model.uid],
             kHYBCacheStateKey : stateKey,
             // 如果设置为YES，若有缓存，则更新缓存，否则直接计算并缓存
             // 主要是对社交这种有动态评论等不同状态，高度也会不同的情况的处理
             kHYBRecalculateForStateKey : @(NO) // 标识不用重新更新
             };
  }];
}
```

###外部显示cell

外部显示cell跟平时写的一样，没有什么特别要设置的：

```
- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView
                 cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
  static NSString *cellIdentifier = @"CellIdentifier";
  HYBNewsCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
  
  if (!cell) {
      cell = [[HYBNewsCell alloc] initWithStyle:UITableViewCellStyleDefault
                                reuseIdentifier:cellIdentifier];
    
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
  }
  
  HYBNewsModel *model = nil;
  if (indexPath.row < self.dataSource.count) {
    model = [self.dataSource objectAtIndex:indexPath.row];
  }
  [cell configCellWithModel:model];
  
  cell.expandBlock = ^(BOOL isExpand) {
    model.isExpand = isExpand;
    [tableView reloadRowsAtIndexPaths:@[indexPath]
                     withRowAnimation:UITableViewRowAnimationFade];
  };
  
  return cell;
}
```

###温馨提示

对于UILabel类型，一定要设置属性preferredMaxLayoutWidth，否则在iOS6上会有问题，另外在设备6/6plus等上计算不准确。

设置preferredMaxLayoutWidth的值一定要与约束所计算出来的值一样，否则也会有问题。其实使用起来很简单的。

#version 2.0.0 

* 本版本做了比较大的改动，为了提高计算行高的效率，降低消耗，使用了重用cell来计算行高的方式，因为计算行高时，永远只会创建一个。
* 与version 1.0.0版本相比，API发生了些变化，原来参数中的indexPath改成了tableview

#使用


这个组件是开源的，而且是支持`cocoapods`的，因此大家若是使用了`cocoapods`来管理项目第三方库，可以这样使用：

```
pod 'HYBMasonryAutoCellHeight', '~> 2.0.0'
```

如果项目未使用`cocoapods`，直接下载源代码，然后将`HYBMasonryAutoCellHeight`文件夹拖入工程即可使用！


#源代码


大家可以到`github`下载源代码来看看，内部实现很简单，当然要实现自动计算行高也是有系统方法的，不一定需要像笔者这样来实现。

下载源代码：[https://github.com/CoderJackyHuang/HYBMasonryAutoCellHeight](https://github.com/CoderJackyHuang/HYBMasonryAutoCellHeight)


#看SWIFT版

笔者还提供了Swift版：[开源HYBSnapkitAutoCellHeight自动算行高](http://www.henishuo.com/snapkit-auto-cell-height/)

#最后

相信大家都明白一个道理，使用我自动化计算行高的方式方便我们开发，提高开发效率，但是会牺牲少许的性能。使用autolayout布局来计算行高，自然不会有纯代码计算行高的方式的性能高。大家在使用时，前期开发通常都会直接使用这种自动布局的方式来计算行高，因为大多数情况下，tableview展示的数据并不都是非常复杂的富文本、图片等数据，性能上没有什么问题。

像新浪微博这样的列表，自然要优化，那么更多的采用纯代码的方式来提前计算行高，并缓存起来，这样在滚动时直接使用行高（只是猜测）。


