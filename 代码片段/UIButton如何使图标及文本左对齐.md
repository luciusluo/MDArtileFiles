#概述

今天在做需求的时候，发现需求中需要显示一个搜索icon及文本。当显示placehoder时，文本及icon居中显示；而当有输入文字时，需要显示为左对齐且文字高亮。

刚开始尝试了会UISearchBar，但是效果图中还要设置背景。尝试了使用UISearchBar，修改背景无效。然后查看UI层级关系，尝试遍历所有子控件，将UISearchBarSearchTextField修改背景颜色，但是只是一点点，并不能达到所需要的效果。

后来，网上各种说法，说将UISearchBarBackground移除就可以，但是都没有生效，至少达不到我要的效果。也许大家有解决办法，不仿将代码贴出来晒晒？

#初始效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/05/QQ20160518-0@2x.png)

#变化后效果图

![image](http://www.henishuo.com/wp-content/uploads/2016/05/QQ20160518-1@2x.png)

#解决办法

**使用UIButton即可实现！**

1. 默认设置，请大家忽略笔者所写的扩展API，这是笔者为公司专门定制的。这里只是设置title及icon：

```
// 搜索按钮
UIButton *searchButton = [UIButton hdf_buttonWithTitle:@"搜索疾病、医生、医院、科室等" superView:self.view constraints:^(MASConstraintMaker *make) {
	make.left.mas_equalTo(15);
	make.right.mas_equalTo(-15);
	make.top.mas_equalTo(7);
	make.height.mas_equalTo(30);
} touchup:^(UIButton *sender) {
	HDFConsultSearchViewController *vc = [[HDFConsultSearchViewController alloc] init];
	[weakObject.navigationController pushViewController:vc animated:YES];
}];
self.searchButton = searchButton;

self.searchButton.titleLabel.font = kBodyFont;
[searchButton setTitleColor:kG3Color forState:UIControlStateNormal];
[searchButton setImage:kImageWithName(@"义诊_搜索框_搜索icon")
            forState:UIControlStateNormal];
searchButton.layer.masksToBounds = YES;
searchButton.layer.cornerRadius = 4.0;
searchButton.backgroundColor = kG6Color;
self.searchButton.titleLabel.textAlignment = NSTextAlignmentLeft;
```

2. 当有输入时，通过自动布局来修改UIButton本身的imageView及titleLabel这两个属性：

```
[self.searchButton.imageView mas_remakeConstraints:^(MASConstraintMaker *make) {
	make.left.mas_equalTo(10);
	make.centerY.mas_equalTo(self.searchButton);
	make.width.height.mas_equalTo(12);
}];
[self.searchButton.titleLabel mas_remakeConstraints:^(MASConstraintMaker *make) {
	make.left.mas_equalTo(self.searchButton.imageView.mas_right);
	make.right.mas_equalTo(-10);
	make.top.bottom.mas_equalTo(0);
}];
```

3. 别忘了设置一下文本对齐方式来左对齐哦：

```
self.searchButton.titleLabel.textAlignment = NSTextAlignmentLeft;
```

#小结

这里只是尝试了使用Masonry通过自动布局来实现的，相信使用苹果原生的自动布局约束也是可以的，大家可以尝试哦！

这只是开发中的一个小技巧哦，以后不要再自己添加一个UIImageView和titleLabel了，就使用其本身的属性就可以做到了！



