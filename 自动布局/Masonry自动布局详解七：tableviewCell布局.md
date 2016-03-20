#前言


说到`iOS`自动布局，有很多的解决办法。有的人使用`xib/storyboard`自动布局，也有人使用`frame`来适配。对于前者，笔者并不喜欢，也不支持。对于后者，更是麻烦，到处计算高度、宽度等，千万大量代码的冗余，对维护和开发的效率都很低。

笔者在这里介绍纯代码自动布局的第三方库：`Masonry`。这个库使用率相当高，在全世界都有大量的开发者在使用，其`star`数量也是相当高的。

#效果图

本节详解`Masonry`的以动画的形式更新约束的基本用法，先看看效果图：

![image](http://www.henishuo.com/wp-content/uploads/2015/11/6.gif)

我们这里初始按钮是一个很小的按钮，点击就不断放大，最大就放大到全屏幕。

#核心代码


看下代码：

```
#import "TableViewController.h"
#import "TestCell.h"

@interface TableViewController () <UITableViewDataSource, UITableViewDelegate>

@property (nonatomic, strong) UITableView *tableView;
@property (nonatomic, strong) NSMutableArray *dataSource;

@end

@implementation TableViewController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.tableView = [[UITableView alloc] init];
  self.tableView.separatorStyle = UITableViewCellSeparatorStyleNone;
  self.tableView.delegate = self;
  self.tableView.dataSource = self;
  [self.view addSubview:self.tableView];
  [self.tableView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
  }];
  
  for (NSUInteger i = 0; i < 10; ++i) {
    TestModel *model = [[TestModel alloc] init];
    model.title = @"测试标题，可能很长很长，反正随便写着先吧！";
    model.desc = @"描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，描述内容通常都是很长很长的，";
    
    [self.dataSource addObject:model];
  }
  
  [self.tableView reloadData];
}

- (NSMutableArray *)dataSource {
  if (_dataSource == nil) {
    _dataSource = [[NSMutableArray alloc] init];
  }
  
  return _dataSource;
}

#pragma mark - UITableViewDataSource
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
  return self.dataSource.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  static NSString *cellIdentifier = @"CellIdentifier";

  TestCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
  
  if (!cell) {
    cell = [[TestCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];
  }
  
  cell.indexPath = indexPath;
  cell.block = ^(NSIndexPath *path) {
    [tableView reloadRowsAtIndexPaths:@[path] withRowAnimation:UITableViewRowAnimationFade];
  };
  TestModel *model = [self.dataSource objectAtIndex:indexPath.row];
  [cell configCellWithModel:model];
  
  return cell;
}

- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  TestModel *model = [self.dataSource objectAtIndex:indexPath.row];
  
  return [TestCell heightWithModel:model];
}


@end
```

#讲解

---
我们来看看这个计算行高的代码，看起来是不是很像配置数据的代理方法呢？

```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
  TestModel *model = [self.dataSource objectAtIndex:indexPath.row];
  
  return [TestCell heightWithModel:model];
}
```

我们看看`TestCell`的声明，提供了一个计算行高的类方法：

```

typedef void (^TestBlock)(NSIndexPath *indexPath);

@interface TestCell : UITableViewCell

@property (nonatomic, strong) UILabel *titleLabel;
@property (nonatomic, strong) UILabel *descLabel;
@property (nonatomic, strong) NSIndexPath *indexPath;

@property (nonatomic, copy) TestBlock block;

- (void)configCellWithModel:(TestModel *)model;

+ (CGFloat)heightWithModel:(TestModel *)model;

@end
```

我们看一下计算行高的实现：

```
+ (CGFloat)heightWithModel:(TestModel *)model {
  TestCell *cell = [[TestCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@""];
  [cell configCellWithModel:model];
  
  [cell layoutIfNeeded];
  
  CGRect frame =  cell.descLabel.frame;
  return frame.origin.y + frame.size.height + 20;
}
```

我们只是创建了一个`cell`然后配置数据，然后调用`layoutIfNeeded`更新约束，以便获取到`frame`。当我们获取到以后，我们就可以计算出最后的`cell`真正的高度了。

关于计算`cell`的行高，笔者开源了一个扩展类，当然在公司内部也是大家都在使用的，可以减少大量的代码。如果需要使用，可以到这里下载：[https://github.com/CoderJackyHuang/HYBMasonryAutoCellHeight](https://github.com/CoderJackyHuang/HYBMasonryAutoCellHeight)或者直接使用`cocoapods`安装。

#源代码

大家可以到笔者的`github`下载源代码：[MasonryDemo](https://github.com/CoderJackyHuang/MasonryDemo)

**温馨提示：**本节所讲内容对应于`TableViewController`中的内容
