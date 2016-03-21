#前言

使用`Masonry`要兼容`iOS6.0`，正常情况下都是可以的。但是对于`UITableView`的`tableHeaderView`或者`tableFooterView`不能直接添加约束，否则在`iOS6.0`上必闪退。

####提示：若您的App不需要支持到iOS6.0，那么您没必要继续阅读这篇文章

#解决方案


```
- (void)configTableView {
  if (self.tableView != nil) {
    return;
  }
  
  self.tableView = [[UITableView alloc] init];
  [self.view addSubview:self.tableView];
  [self.tableView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.edges.mas_equalTo(self.view);
  }];
  
  NSArray *array = [self headerViewWithHeight:self.view.frame.size.height addToView:self.view];
    UIView *headerView = [array firstObject];
  [headerView layoutIfNeeded];
  UIButton *button = [array lastObject];
  CGFloat h = button.frame.size.height + button.frame.origin.y + 40;
  h = h < self.view.frame.size.height ? self.view.frame.size.height : h;
  
  [headerView removeFromSuperview];
  [self headerViewWithHeight:h addToView:nil];
}

- (NSArray *)headerViewWithHeight:(CGFloat)height addToView:(UIView *)toView {
  // 注意，绝对不能给tableheaderview直接添加约束，必闪退
  UIView *headerView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, height)];
  
  if (toView) {
    [toView addSubview:headerView];
  } else {
    self.tableView.tableHeaderView = headerView;
  }
  
  UIImageView *imgView = [[UIImageView alloc] init];
  [headerView addSubview:imgView];
  imgView.backgroundColor = [UIColor greenColor];
  imgView.layer.cornerRadius = 50;
  imgView.layer.masksToBounds = YES;
  imgView.layer.borderWidth = 0.5;
  imgView.layer.borderColor = [UIColor redColor].CGColor;
  
  CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
  
  UILabel *tipLabel = [[UILabel alloc] init];
  tipLabel.text = @"这里是提示信息，通常会比较长，可能会超过两行。为了适配6.0，我们需要指定preferredMaxLayoutWidth，但是要注意，此属性一旦设置，不是只在6.0上生效，任意版本的系统的都有作用，因此此值设置得一定要准备，否则计算结果会不正确。我们一定要注意，不能给tableview的tableHeaderView和tableFooterView添加约束，在6.0及其以下版本上会crash，其它版本没有";
  tipLabel.textAlignment = NSTextAlignmentCenter;
  tipLabel.textColor = [UIColor blackColor];
  tipLabel.backgroundColor = [UIColor clearColor];
  tipLabel.numberOfLines = 0;
  tipLabel.preferredMaxLayoutWidth = screenWidth - 30;
  [headerView addSubview:tipLabel];
  
  UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
  [button setTitle:@"Show detail" forState:UIControlStateNormal];
  [button setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
  [button setBackgroundColor:[UIColor blueColor]];
  button.layer.cornerRadius = 6;
  button.clipsToBounds = YES;
  button.layer.masksToBounds = YES;
  
  [headerView addSubview:button];
  
  [imgView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(80);
    make.centerX.mas_equalTo(headerView);
    make.width.height.mas_equalTo(100);
  }];
  
  [tipLabel mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.mas_equalTo(self.view).offset(15);
    make.right.mas_equalTo(self.view).offset(-15);
    make.top.mas_equalTo(imgView.mas_bottom).offset(40);
  }];
  
  [button mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_greaterThanOrEqualTo(tipLabel.mas_bottom).offset(80);
    make.left.mas_equalTo(tipLabel);
    make.right.mas_equalTo(tipLabel);
    make.height.mas_equalTo(45);
  }];
  
  return @[headerView, button];
}
```

关键解决方法是在这里：

```
NSArray *array = [self headerViewWithHeight:self.view.frame.size.height addToView:self.view];
UIView *headerView = [array firstObject];
[headerView layoutIfNeeded];
UIButton *button = [array lastObject];
CGFloat h = button.frame.size.height + button.frame.origin.y + 40;
h = h < self.view.frame.size.height ? self.view.frame.size.height : h;
  
[headerView removeFromSuperview];
[self headerViewWithHeight:h addToView:nil];
```

这里就是调用了两个创建`headerView`，其中第一次创建的目的是获取总高，如果提前就已经知道总高，那么就不需要调用两次了。但是，如果是动态的，我们事先不确定其总高，那么我们还是使用自动布局来计算总高。

第一次调用时，将`headerView`放在`self.view`上，得到总高后，再移除掉。

第二次调用的时候，就是直接写`self.tableView.tableHeaderView = headerView`就可以了。

#最后

总结工作中所出现过的点点滴滴，帮助他人的同时，对自己也很有帮助哦！大家不防也试试！
