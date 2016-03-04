>#iOS NSLayoutAttributeLeft与NSLayoutAttributeLeading的区别

在使用`Masonry`时，有`mas_left`与`mas_leading`，同样有`mas_right`与`mas_trailing`，在中国都习惯左、右布局，使用`left/right`与`heading/trailing`是一样的。但是，在其它部分国家，开发者们的习惯不都是左、右或者前、后布局，还有右、左或者后、前布局的，因此是不一样的。

在中国，就放心的使用左、右或者前、后的方式吧。

>#支持原创，请[阅读原文](http://www.henishuo.com/nslayoutattributeleft-nslayoutattributeleading-differents/)

#Masonry属性

---

```
@property (nonatomic, strong, readonly) MASViewAttribute *mas_left;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_top;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_right;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_bottom;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_leading;
@property (nonatomic, strong, readonly) MASViewAttribute *mas_trailing;
```

我们看到这几个属性，但是我们并不需要都使用，因为在中国人的行为习惯中，大家都习惯从左到右的方式布局。我们完全可以不使用`mas_leading`和`mas_trailing`

对于苹果原生约束的枚举`NSLayoutAttribute`中的几个：

```
NSLayoutAttributeLeft = 1,
NSLayoutAttributeRight,
NSLayoutAttributeTop,
NSLayoutAttributeBottom,
NSLayoutAttributeLeading,
NSLayoutAttributeTrailing,
```

我们更常见的是使用`leading`和`trailing`而不是`left/right`。

#推荐

---
笔者所见过使用`Masonry`的写法中，几乎没有见过使用`mas_leading/mas_trailing`的，几乎都是使用`mas_left/mas_right`，当然也包括笔者。

因此，笔者也推荐大家使用`Masonry`时，使用`mas_left/mas_right`，但是使用`xib/storyboard`上的约束时，使用`heading/trailing`最多。


#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**

