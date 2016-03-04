

#前言

相信很多同学都遇到这么一个问题：在`storyboard`上如何使用`scrollview`自动根据内容的增长而自动使其`contentSize`而变化，以使之可滚动。或者如何使用**纯代码**实现`scrollview`上使添加的控件在超出显示屏幕时可滚动，也就是根据内容自动计算出其`contentSize`的问题。

在这里，将使用`storyboard`和`Masonry`纯代码实现`scrollview`自动布局。如果有说得不正确的地方，欢迎指出!

**说明：**本人向来不使用xib或者storyboard开发，几乎都是使用纯代码的autolayout，也就是使用`Masonry`。不管是使用`Swift`开发或者是`Objective-C`开发，都会基于`Masonry`或`SnapKit`写一套控件，以简化开发。

#ScrollView自动布局

我们实现下面的效果图：

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD3pecN4ytLM6qIapriaJ2KiacFhGvy9zz3BbNj2wEbmKXU2cgA18LiaIWcZTIvVrxH4h7aSAHia3JZ1xg/0?wx_fmt=jpeg)

####第一步：添加`UIScrollView`并布局约束为点满整个`self.view`
点击右下角的第二个小图标，添加约束如下图：

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD3pecN4ytLM6qIapriaJ2KiacyDNqibQrel8FVkada9RFf4qwNDmZ5q20Q8FWa5IrqMGKWptHYSCLx3g/0?wx_fmt=png)

设置`UIScrollView`的上、下、左、右与`self.view`的上、下、左、右都一致，相当于点满了整个`self.view`。

**注意：**`Constraints to margin`尽量不要勾选，也就是是否使用margin的意思。如果学习过HTML，应该更容易理解。在HTML中有margin、padding，对于一个盒（Box），除了content外，还有padding、margin。其中padding是与内容的间隔，而margin是与外部的间隔。在这里，如果勾选了，就会相差8个像素。个人在使用中都会去掉这个勾选。

####第二步：添加红色的标签(redLabel)
首先设置这个标签的相关前景色、背景色、numberOfLines=0，添加文本等，然后选中这个标签，像上面第一步中的图片指示，点击右下角小图标，在弹出框中添加约束。

添加约束为：左、上、右与`self.view`的左、上、右相等，内容会自动换行，高度会自动得到。这里可以使左、上与`UIScrollView`的左、上相等，但是不能使用右（trailing）不能等于`UIScrollView`的右（trailing）。

**注意：**在使用布局子控件到`UIScrollView`上时，由于`UIScrollView`的right是不确定的，因为`UIScrollView`有`contentSize`，即内容的宽和内容的高。

####第三步：添加绿色标签(blueLabel)
与第二步差不多，添加前景色等，添加约束为：使blueLabel的leading和width分别等于redLabel的leading和width，再使blueLabel的top等于redLabel的bottom再加上20（间隔）。

####第四步：添加图片（imgView）

添加图片很简单，设置约束为：width和height值都设置为250，top为blueLabel的bottom再加上20，然后设置水平居中就可以了。看下图：

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD3pecN4ytLM6qIapriaJ2Kiackcm6A4bDaFOj006FAUVhULCmVTTFVcCiaWFkAXrl9Py5HueVj6LqjRw/0?wx_fmt=png)

####第五步：添加白色标签（whiteLabel）

与第三步差不多，只是top为imgView的bottom再加上20就可以了。

####第六步：添加红色按钮

按钮颜色等配置就不说了，只说如何添加约束。这里需要注意一点，需要设置bottom，以确定scrollview的contentSize.height。添加约束为：leading、trailing、bottom分别与scrollview的leading、trailing、bottom+20相等，再设置其height、top就可以了。这里使button的bottom=scrollview.bottom，也就可以确定scrollview的contentSize.height了.

#平分布局

下面我们来实现下面的布局，使控件平分显示。

![image](https://mmbiz.qlogo.cn/mmbiz/sia5QxFVcFD3pecN4ytLM6qIapriaJ2KiacnuI2zHiaqhk5wVWJoHeszCqaDpJEIeT7mItQt3TgLhuUKjiaLUT8TYhA/0?wx_fmt=jpeg)

####第一行：如何平分这两个控件使其等宽、等高、间隔20，与self.view的左、右间隔10像素。

* 首先，同时选中这两个控件，先添加下面的约束：设置这两个控件Equal widths,Equal heights。
* 设置左边的控件的约束为：leading为10，height为80，trailing为右边控件的leading-20，top为0.
* 设置右边的控件的约束为：trailing为-10，top=左边控件的top就可以实现了。

**注意：**这里设置左边的控件的trailing为右边控件的leading，再减去20个像素（间隔），这样才能确定其宽。

####第二行：如何三等分显示，使中间两个间隔都为20，外边两个与边界的间隔都为10

* 首先，同时选中这三个控件，添加约束：等宽、等高。
* 设置button3的top距离上方控件的底部20像素，再设置button4、button5的top都与button3的top一致。设置button3的leading为10，trailing为button4的leading-20.
* 设置button4的trailing为button5的leading-20
* 设置button5的trailing为-10

####第三行：如何将两个控件的高度等分
第三行中左边是一个图片，右边是两个按钮，一上一下且等宽等高，中间间隔10像素。

* 首先，固定左边的图片，设置约束：宽、高都为250，leading为10，top为20.
* 将button6和button7设置为等宽、等高
* 设置button6的leading为10，trailing为-10，top为左边图片的top
* 设置button7的leading和trailing都为button6的leading和trailing
* 最后，同时选中button6和button7，按住ctrl，拖向左边的图片，设置AspectRadio，设置其比例为40：100，这样每个按钮的高度就是图片的高度的40%。当然这里我们也可以直接设置button7的bottom=左边的图片的bottom，这样也可以达到目的。

#基于Masonry的Autolayout

下面我们给self.view上放一个点满的scrollview:

``` 
UIScrollView *scrollView = [[UIScrollView alloc] init];
[self.view addSubview:scrollView];
```

然后，我们放一个说明标签，具体说明看代码注释：

```
// 放一个说明标签
UILabel *tipLabel = [[UILabel alloc] init];
tipLabel.backgroundColor = [UIColor redColor];
tipLabel.numberOfLines = 0;
tipLabel.textColor = [UIColor whiteColor];
tipLabel.textAlignment = NSTextAlignmentLeft;
[scrollView addSubview:tipLabel];
[tipLabel mas_makeConstraints:^(MASConstraintMaker *make) {
  make.left.top.mas_equalTo(10);
  // 注意：这里直接使用weakSelf.view，相当于weakSelf.view.mas_right
  // 另外：由于要与weakSelf.view.mas_right距离10像素，因此这里值为-10。
  make.right.mas_equalTo(weakSelf.view).offset(-10);
}];
tipLabel.text = @"这个标签是使用Masonry完成的纯代码自动布局。这个标签的约束添加方式为：使左、上与父视图的左、上分别相等，使右边与self.view的右边的相距-10，就可以确定其宽。这里不能使用使右等于scrollview的右，因为scrollview是可以滚动的，其右是不确定的。";
```

接下来，在`tipLabel`下面又放一个标签，具体说明看代码注释：

```
UILabel *codeLabel = [[UILabel alloc] init];
codeLabel.backgroundColor = [UIColor redColor];
codeLabel.numberOfLines = 0;
codeLabel.textColor = [UIColor whiteColor];
codeLabel.textAlignment = NSTextAlignmentLeft;
[scrollView addSubview:codeLabel];
[codeLabel mas_makeConstraints:^(MASConstraintMaker *make) {
  // 使与tipLabel的左、右分别对齐 ，也就确定了宽
  make.left.right.mas_equalTo(tipLabel);
  
  make.top.mas_equalTo(tipLabel.mas_bottom).offset(40);
  // 注意：这里直接使用weakSelf.view，相当于weakSelf.view.mas_right
  // 另外：由于要与weakSelf.view.mas_right距离10像素，因此这里值为-10。
}];
codeLabel.text = @"本标签的约束添加方式为：使左、右与上面的标签的左、右分别对齐，由此就可确定左、右和宽，再使顶部top等于上面的标签的bottom再加上40个像素。\n欢迎扫一扫我的微信公众号二维码，关注公众号，或者直接搜索iOSDevShares。若想加QQ群，请加：324400294";
```

下面我们在上面的标签之下，放一个图片。对于图片，如果图片本身大小是正好合适的，那么我们可以使用`sizeToFit`方法来生成适应，但是我们图片过大或者过小，那么我们可以在约束中指定为固定的大小：

```
UIImageView *codeImageView = [[UIImageView alloc] init];
codeImageView.image = [UIImage imageNamed:@"微信公众号.jpg"];
[scrollView addSubview:codeImageView];
[codeImageView mas_makeConstraints:^(MASConstraintMaker *make) {
  make.centerX.equalTo(weakSelf.view);
  make.top.mas_equalTo(codeLabel.mas_bottom).offset(20);
  make.size.mas_equalTo(CGSizeMake(250, 250));
}];

// 由于图片过大，需要限制。如果是图片刚好，可以不设置大小的约束，使用下面的方式。
//  [codeImageView sizeToFit];
```

使用代码又是如何平分控件的呢？下面看看这两个标签，其内容是可变的，不知道哪个的内容多，但是其宽是一样的，只是高度会随内容而动态变化。

```
// 平分两个控件
UILabel *avgLabel1 = [[UILabel alloc] init];
avgLabel1.backgroundColor = [UIColor redColor];
avgLabel1.numberOfLines = 0;
avgLabel1.textColor = [UIColor whiteColor];
avgLabel1.textAlignment = NSTextAlignmentCenter;
[scrollView addSubview:avgLabel1];
avgLabel1.text = @"本控件的约束添加方式为：使left与父视图的left相距10像素，使top=上面的图片的bottom再加40像素，使right=右边这个标签的left再减去20个像素（间隔），使height=80。";

UILabel *avgLabel2 = [[UILabel alloc] init];
avgLabel2.backgroundColor = [UIColor redColor];
avgLabel2.numberOfLines = 0;
avgLabel2.textColor = [UIColor whiteColor];
avgLabel2.textAlignment = NSTextAlignmentCenter;
[scrollView addSubview:avgLabel2];
avgLabel2.text = @"本控件的约束添加方式为：使right=self.view的right再减去10像素，然后再设置宽、top都与左右的视图一样，就可以实现水平平分了。本控件的约束添加方式为：使right=self.view的right再减去10像素，然后再设置宽、top都与左右的视图一样，就可以实现水平平分了。";

[avgLabel1 mas_makeConstraints:^(MASConstraintMaker *make) {
  make.left.mas_equalTo(10);
  make.top.mas_equalTo(codeImageView.mas_bottom).offset(40);
  make.right.mas_equalTo(avgLabel2.mas_left).offset(-20);
}];
[avgLabel2 mas_makeConstraints:^(MASConstraintMaker *make) {
  make.right.mas_equalTo(weakSelf.view.mas_right).offset(-10);
  make.width.top.mas_equalTo(avgLabel1);
}]
```

最后，看看我们是如何使scrollview随内容而变化，也就是可以滚动。另外，上面的两个控件是等宽的，但是其高都是不定的，到底以哪个的bottom作为scrollView的bottom呢？这里使用了很巧妙的方法。

```
// 使用edges使点满整个self.view
[scrollView mas_makeConstraints:^(MASConstraintMaker *make) {
  make.edges.equalTo(weakSelf.view);
  
  // 如果这两个标签的内容都是不确定的，也就是不确定哪个的内容更多，那么可以这么设置。
  // 这样就可以保证使用内容最多的标签作为scrollview的contentSize参考。
  // 用于确定scrollview的contentSize.height
  make.bottom.mas_greaterThanOrEqualTo(avgLabel1.mas_bottom).offset(40);
  make.bottom.mas_greaterThanOrEqualTo(avgLabel2.mas_bottom).offset(40);
}];
```

#源代码

想要下载源代码，请移步github下载，内有Swift版的工程和ObjC版的工程：<br/>
[https://github.com/CoderJackyHuang/ScrollViewAutolayoutDemo](https://github.com/CoderJackyHuang/ScrollViewAutolayoutDemo)



