# 前言

最近有不少朋友们、群友们经常问到UITextView如何加载HTML？UILabel能加载HTML吗？或者问后台接口返回来的是HTML格式的数据，我该怎么显示呢？怎么处理呢？

在这里，笔者针对这些朋友、群友们的反馈，尝试写了个小demo，希望能够帮助到大家吧！

#Demo效果截图

![image](http://www.henishuo.com/wp-content/uploads/2016/02/QQ20160228-0@2x.png)

这demo中是放在cell里面加载的，并且教大家如何自动计算行高。不过UITextView计算行高是有误差的，因为笔者没有使用更高级的处理，直接使用了sizeThatFits这个API来计算高度。而UITextView天生就不一样，它有上、下、左、右的间隔的，因此计算出来是有一点小偏差的。

本篇文章只讲如何加载，不讲如何精确计算！

#使用到NSAttributedString

通过它就可以设置加载HTML。但是，要让UILabel可以加载HTML，要求在iOS7之后才可以使用：

```
- (nullable instancetype)initWithData:(NSData *)data options:(NSDictionary<NSString *, id> *)options documentAttributes:(NSDictionary<NSString *, id> * __nullable * __nullable)dict error:(NSError **)error NS_AVAILABLE(10_0, 7_0);
```

其中，options中的指定key为:

```
UIKIT_EXTERN NSString * const NSDocumentTypeDocumentAttribute NS_AVAILABLE(10_0, 7_0);  
```

时，它可以选择的值有：

```
UIKIT_EXTERN NSString * const NSPlainTextDocumentType NS_AVAILABLE(10_0, 7_0);
UIKIT_EXTERN NSString * const NSRTFTextDocumentType NS_AVAILABLE(10_0, 7_0);
UIKIT_EXTERN NSString * const NSRTFDTextDocumentType NS_AVAILABLE(10_0, 7_0);
UIKIT_EXTERN NSString * const NSHTMLTextDocumentType NS_AVAILABLE(10_0, 7_0);
```

其中，NSHTMLTextDocumentType就是设置要加载HTML了。

# UILabel加载HTML

UILabel在iOS6.0后提供了一个属性用于设置各种呈现的样式：

```
@property(null_resettable,copy) NSAttributedString *attributedText NS_AVAILABLE_IOS(6_0);
```

虽然attributedText属性是iOS6就可以使用，但是对于加载HTML，要求是在iOS7以上才能使用：

```
// ios 7.0以后才能使用
NSData *data = [model.html dataUsingEncoding:NSUnicodeStringEncoding];
NSDictionary *options = @{NSDocumentTypeDocumentAttribute: NSHTMLTextDocumentType};
NSAttributedString *html = [[NSAttributedString alloc] initWithData:data
                                                            options:options
                                                 documentAttributes:nil
                                                              error:nil];
self.htmlLabel.attributedText = html;
```

# UITextView加载HTML

UITextView也提供了相关设置文本样式的属性：

```
@property(null_resettable,copy) NSAttributedString *attributedText NS_AVAILABLE_IOS(6_0);
```

与UILabel类似，虽然attributedText属性是iOS6就可以使用，但是对于加载HTML，要求是在iOS7以上才能使用：

```
// ios 7.0以后才能使用
NSData *data = [model.html dataUsingEncoding:NSUnicodeStringEncoding];
NSDictionary *options = @{NSDocumentTypeDocumentAttribute: NSHTMLTextDocumentType};
NSAttributedString *html = [[NSAttributedString alloc] initWithData:data
                                                            options:options
                                                 documentAttributes:nil
                                                              error:nil];
self.textView.attributedText = html;

// 加载HTML后，还要设置行高约束，否则高度就是0
CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width;
[self.textView mas_updateConstraints:^(MASConstraintMaker *make) {
  make.height.mas_equalTo([self.textView sizeThatFits:CGSizeMake(screenWidth - 20, CGFLOAT_MAX)].height);
}];
```

在加载好HTML后，也要设置其高度，但是要注意，sizeThatFits:这个API计算UITextView的高度是不精准的，有一定的误差。

# 最后

顺便说一下，属性中指定的类型null_resettable是什么鬼？这是新特性啦，从英文角度看就大概可以看出来意思是**可空、可重新设置值**。

# 源代码

大家只可以下载本篇文章中所关联的小demo，仅供参考！

下载地址：[CoderJackyHuang](https://github.com/CoderJackyHuang/TextVeiw-Label-HTML-DEMO) 欢迎关注笔者的GITHUB地址，关注标哥的技术博客！

