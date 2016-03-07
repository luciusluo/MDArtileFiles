#前言

开发中常用到的设置UILabel的文本样式代码片段：

```
NSMutableAttributedString *str = [[NSMutableAttributedString alloc] initWithString:@"Using NSAttributed String，try your best to test attributed string text"];

[str addAttribute:NSForegroundColorAttributeName
            value:[UIColor blueColor]
            range:NSMakeRange(0,5)];
[str addAttribute:NSForegroundColorAttributeName
            value:[UIColor redColor]
            range:NSMakeRange(6,12)];
[str addAttribute:NSForegroundColorAttributeName
            value:[UIColor greenColor]
            range:NSMakeRange(19,6)];
[str addAttribute:NSFontAttributeName
            value:[UIFont fontWithName:@"Arial" size:30.0]
            range:NSMakeRange(0, 5)];
[str addAttribute:NSFontAttributeName
            value:[UIFont fontWithName:@"Arial" size:30.0]
            range:NSMakeRange(6, 12)];
[str addAttribute:NSFontAttributeName
            value:[UIFont fontWithName:@"Arial" size:30.0]
            range:NSMakeRange(19, 6)];

UILabel *attrLabel = [[UILabel alloc] initWithFrame:CGRectMake(20, 150, 320 - 40, 90)];
attrLabel.attributedText = str;
attrLabel.numberOfLines = 0;
```

#说明

NSMutableAttributedString类可以添加各种样式，常用的设置key有：

* NSForegroundColorAttributeName 设置前景色，也就是文本颜色
* NSFontAttributeName 设置字体
* NSBackgroundColorAttributeName 设置背景色

更多内容点击NSMutableAttributedString进去查看声明。
