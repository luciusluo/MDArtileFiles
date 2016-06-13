#概述

今天教大家利用UITableView的section实现像QQ那样的展开与收起的效果。其实实现起来是非常简单的，不过还是写一篇文章给大家讲讲思路并给新手参考的源代码。

简单，但是不同的人来实现也许会有不同的实现方式，简单程度也会不一样哦。其实与AppStore中的分类页面中的效果挺像的，不过人家那并不是这种动画效果，而是更美的动画！

#文章内容

看完本篇文章，您将会学习到：

* [如何利用UITableView的section来实现展开与收起的效果]()
* [如何复用UITableView的header view]()
* [如何通过模型与UI结构一一对应，更好地解决cell复用问题]()

#效果图

千言万语不如一张gif图来得实在，请看下面的效果：

![image](http://www.henishuo.com/wp-content/uploads/2016/06/sectionAnimation.gif)

#设想

当看到这样的效果图时，您是怎么想的？如何实现呢？是否有现成的？动画是否满足需求？可扩展性够不够？

如果您对UITableView够熟悉，那么应该可以轻松想到UITableView的section，展开后就是UITableView的多个cell。

当然，如果不使用UITableView的section来实现，那么就需要自己解决复用的问题，反而麻烦化了。因此，本篇文章教大家如何利用UITableView来实现。

#建立模型

UI的层级结构会与模型的结构一样，这样会最简化。首先，我们定义分区模型，每个分区模型代表一个区，区下面可以有很多个cell：

```
// 分区模型
@interface HYBSectionModel : NSObject

@property (nonatomic, copy) NSString *sectionTitle;
// 是否是展开的
@property (nonatomic, assign) BOOL isExpanded;
// 分区下面可以有很多个cell对应的模型
@property (nonatomic, strong) NSMutableArray *cellModels;

@end

// 每个cell对应一个这样的model
@interface HYBCellModel : NSObject

@property (nonatomic, copy) NSString *title;

@end
```

如此建立好模型与UI的层级关系后，使用起来是非常简单的。

#自定义UITableViewHeaderFooterView

我们这里需要自定义UITableViewHeaderFooterView，因此需要子类化它。其实跟UITableViewCell的子类化是一样的，都有差不多的属性，使用起来也是一样一样的。

可能很多朋友没有见过有人使用这个东西。是的，在项目开发中，我也没有见过其他人使用它，全都是直接自定义一个UIView。这并不是不行，只是它不能与UITableView的复用联系起来。

我们自定义HYBHeaderView，那么注册复用headerFooterView就可以复用了：

```
[self.tableView registerClass:[HYBHeaderView class] forHeaderFooterViewReuseIdentifier:kHeaderIdentifier];
```

首先，我们先子类化UITableViewHeaderFooterView，我们需要传HYBSectionModel对象过来，代表一个分区头，通过model重写Setter方法，自动配置数据。因为展开与收起，我们添加了一个简单的动画，所以外部需要回调，我们给一个回调的闭包：

```
@class HYBSectionModel;

typedef void(^HYBHeaderViewExpandCallback)(BOOL isExpanded);

@interface HYBHeaderView : UITableViewHeaderFooterView

@property (nonatomic, strong) HYBSectionModel *model;
@property (nonatomic, copy) HYBHeaderViewExpandCallback expandCallback;

@end
```

可能会有人问，为什么HYBSectionModel使用@class来声明，而不是直接引入头文件呢？其实，我在项目开发中，通常都是使用@class在头文件声明，而在实现文件中才真正地引入，这样可以避免循环包含而导致编译错误的问题。

我们的实现文件内容是比较少的，我们通过重写setModel方法来配置数据。我在项目开发中，几乎都是采用这种配置数据方式，除非一个cell会有多个地方共用。如果一个cell被多个地方共用，且model有的相同，有的不一样，这才需要特定写API来区分。我们是通过tranform来添加旋转动画的，实现起来非常简单：

```
#import "HYBHeaderView.h"
#import "HYBSectionModel.h"

@interface HYBHeaderView ()

@property (nonatomic, strong) UIImageView *arrowImageView;
@property (nonatomic, strong) UILabel *titleLabel;

@end

@implementation HYBHeaderView

- (instancetype)initWithReuseIdentifier:(NSString *)reuseIdentifier {
  if (self = [super initWithReuseIdentifier:reuseIdentifier]) {
    CGFloat w = [UIScreen mainScreen].bounds.size.width;
    
    self.arrowImageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"expanded"]];
    self.arrowImageView.frame = CGRectMake(10, (44 - 8) / 2, 15, 8);
    [self.contentView addSubview:self.arrowImageView];
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button addTarget:self action:@selector(onExpand:) forControlEvents:UIControlEventTouchUpInside];
    [self.contentView addSubview:button];
    button.frame = CGRectMake(0, 0, w, 44);
    
    self.titleLabel = [[UILabel alloc] initWithFrame:CGRectMake(35, 0, 200, 44)];
    self.titleLabel.textColor = [UIColor whiteColor];
    self.titleLabel.backgroundColor = [UIColor clearColor];
    [self.contentView addSubview:self.titleLabel];
    self.contentView.backgroundColor = [UIColor purpleColor];
    
    UIView *line = [[UIView alloc] initWithFrame:CGRectMake(0, 44 - 0.5, w, 0.5)];
    line.backgroundColor = [UIColor lightGrayColor];
    [self.contentView addSubview:line];
  }
  
  return self;
}

- (void)setModel:(HYBSectionModel *)model {
  if (_model != model) {
    _model = model;
  }
  
  
  if (model.isExpanded) {
    self.arrowImageView.transform = CGAffineTransformIdentity;
  } else {
    self.arrowImageView.transform = CGAffineTransformMakeRotation(M_PI);
  }
  
  
  self.titleLabel.text = model.sectionTitle;
}

- (void)onExpand:(UIButton *)sender {
  self.model.isExpanded = !self.model.isExpanded;
  
  [UIView animateWithDuration:0.25 animations:^{
    if (self.model.isExpanded) {
      self.arrowImageView.transform = CGAffineTransformIdentity;
    } else {
      self.arrowImageView.transform = CGAffineTransformMakeRotation(M_PI);
    }
  }];
  
  if (self.expandCallback) {
    self.expandCallback(self.model.isExpanded);
  }
}

@end
```

我们在配置数据的时候，也会根据isExpanded状态来显示图片的方向，否则重用后就恢复原状了。

#创建UITableView并配置数据源

首先，我们先创建UITableView，并注册cell、headerView：

```
self.tableView = [[UITableView alloc] initWithFrame:self.view.bounds];
[self.view addSubview:self.tableView];
self.tableView.delegate = self;
self.tableView.dataSource = self;
[self.tableView registerClass:[UITableViewCell class]
     forCellReuseIdentifier:kCellIdentfier];
[self.tableView registerClass:[HYBHeaderView class] forHeaderFooterViewReuseIdentifier:kHeaderIdentifier];
```

然后，初始化数据源：

```
- (NSMutableArray *)sectionDataSources {
  if (_sectionDataSources == nil) {
    _sectionDataSources = [[NSMutableArray alloc] init];
    
    for (NSUInteger i = 0; i < 20; ++i) {
      HYBSectionModel *sectionModel = [[HYBSectionModel alloc] init];
      sectionModel.isExpanded = NO;
      sectionModel.sectionTitle = [NSString stringWithFormat:@"section: %ld", i];
      NSMutableArray *itemArray = [[NSMutableArray alloc] init];
      for (NSUInteger j = 0; j < 10; ++j) {
        HYBCellModel *cellModel = [[HYBCellModel alloc] init];
        cellModel.title = [NSString stringWithFormat:@"标哥出品：section=%ld, row=%ld", i, j];
        [itemArray addObject:cellModel];
      }
      sectionModel.cellModels = itemArray;
      
      [_sectionDataSources addObject:sectionModel];
    }
  }
  
  return _sectionDataSources;
}
```

**Tip：**通过在开发中，我都会采用重写getter方法来创建，包括UI也是一样的，对于不需要马上显示的，都采用延迟加载。

#实现UITableView的代理方法

对于我们这里，分区就是对应的数据源数组的元素的个数，而每个section的行数需要判断是展开还是收起。当是展开状态是，就是cellModels数组的元素个数，否则就是0。

我们这里需要通过dequeueReusableHeaderFooterViewWithIdentifier:kHeaderIdentifier来获取复用的headerView：

```
#pragma mark - UITableViewDataSource
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
  return self.sectionDataSources.count;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  HYBSectionModel *sectionModel = self.sectionDataSources[section];
  
  return sectionModel.isExpanded ? sectionModel.cellModels.count : 0;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:kCellIdentfier
                                                          forIndexPath:indexPath];
  HYBSectionModel *sectionModel = self.sectionDataSources[indexPath.section];
  HYBCellModel *cellModel = sectionModel.cellModels[indexPath.row];
  cell.textLabel.text = cellModel.title;
  
  return cell;
}

- (UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section {
  HYBHeaderView *view = [tableView dequeueReusableHeaderFooterViewWithIdentifier:kHeaderIdentifier];
  
  HYBSectionModel *sectionModel = self.sectionDataSources[section];
  view.model = sectionModel;
  view.expandCallback = ^(BOOL isExpanded) {
    [tableView reloadSections:[NSIndexSet indexSetWithIndex:section]
             withRowAnimation:UITableViewRowAnimationFade];
  };
  
  return view;
}

#pragma mark - UITableViewDelegate
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  return 44;
}

- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section {
  return 44;
}
```

**Tip：**在tableview的优化中，其中有一条就是复用cell、复用header/footer view。

#小结

通过本篇文章，是否实现到了：

* [如何利用UITableView的section来实现展开与收起的效果]()
* [如何复用UITableView的header view]()
* [如何通过模型与UI结构一一对应，更好地解决cell复用问题]()

如果还不会，不如再看一遍，然后下载代码来模仿实现一遍！行动才能说明一切！好了，如果有任何问题，可以加群交流。如果您想到了更好的实现方式，请一定要与我分享！

#源代码

如果想要下载源代码来参考参考，可以到这个链接处下载：

[标哥的技术博客GITHUB：CoderJackyHuang](https://github.com/CoderJackyHuang/SectionAnimation)

