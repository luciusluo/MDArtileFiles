
#前言

最近还是有不少朋友老问Swift版的自动计算行高怎么做，大家使用SnapKit来自动布局时，都希望能够自动地计算出行高，不用每次都自己去算一篇。

本篇介绍笔者所开源的基于SnapKit这套自动布局库而写的一个扩展，用于自动计算行高。最重要的是，只要约束正确，就可以实现自动计算行高，而且当我们需要动态修改约束时，只要统一放在配置数据的API那里修改约束一样可以计算出正确的高度。

#demo效果

千言万语，不如一个demo效果图，看完效果图再继续往下看：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/snapkitcellheight.gif)

#库名HYBSnapkitAutoCellHeight

名称命名为HYBSnapkitAutoCellHeight，主要是区别于Masonry版本。关于Masonry自动计算行高的版本，大家可以阅读：[HYBMasonryAutoCellHeight](http://www.henishuo.com/masonry-cell-height-auto-calculate/)，它是Objective-C写的。

#设计思路

既然我们使用的是自动布局，那么就可以利用自动布局来准确地计算出位置。要想通过自动布局立即得到位置和大小，那么就需要调用`layoutIfNeeded`方法，这样就可以获取所有控件的`frame`了。

既然可以通过此方法获取到所有控件的`frame`了，那么我们若指定某一个视图为最后一个视图，作为参考，那么就可以直接通过获取该视图的frame来计算高度了。但是，如果我们并不是与指定的视图的位置平齐怎么办？没有关系，我们提供了另外一个属性用于设置最后一个参考视图与cell的最底部的间隔是多少。

我们指定如下两个属性：

```
public var hyb_lastViewInCell: UIView? {
  get {
    let lastView = objc_getAssociatedObject(self, &__hyb_lastViewInCellKey);
    return lastView as? UIView
  }
  
  set {
    objc_setAssociatedObject(self,
      &__hyb_lastViewInCellKey,
      newValue,
      .OBJC_ASSOCIATION_RETAIN_NONATOMIC);
  }
}

public var hyb_bottomOffsetToCell: CGFloat? {
  get {
    let offset = objc_getAssociatedObject(self, &__hyb_bottomOffsetToCell);
    return offset as? CGFloat
  }
  
  set {
    objc_setAssociatedObject(self,
      &__hyb_bottomOffsetToCell,
      newValue,
      .OBJC_ASSOCIATION_ASSIGN);
  }
}
```

这两个是通过runtime添加的属性，因为扩展并不能添加属性。由于swift中关联值所使用的key是个指针，因为我们的参数应该是传一个地址，比如我们定义的key是`__hyb_lastViewInCellKey`，它是String类型的，而String类型是结构体类型，但是实际应该是要传一个指针，因此需要传这个字符串的地址过去。

#单API

对外只提供了一个API，而且还是类方法：

```
/**
 唯一的类方法，用于计算行高
 
 - parameter indexPath:	index path
 - parameter config:		在config中调用配置数据方法等
 
 - returns: 所计算得到的行高
 */
public class func hyb_cellHeight(forIndexPath indexPath: NSIndexPath, config: ((cell: UITableViewCell) -> Void)?) -> CGFloat {
  let cell = self.init(style: .Default, reuseIdentifier: nil)
  
  if let block = config {
    block(cell: cell);
  }
  
  return cell.hyb_calculateCellHeight(forIndexPath: indexPath)
}
```

比如，在demo中我们这样调用的：

```
// MARK: UITableViewDelegate
func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
  let model = self.dataSource[indexPath.row]
  
  return TestCell.hyb_cellHeight(forIndexPath: indexPath, config: { (cell) -> Void in
    let itemCell = cell as? TestCell
    itemCell?.config(testModel: model)
  })
}
```

是不是非常简单呢？这样就不用担心行高的问题了。之前我所有的项目中都是使用自己封装的OC版本基于Masonry写的扩展，很好用的哦！

#如何在cell中自动布局

我们下面来看看要使用这个扩展，应该如何布局呢？它有四大必须，一个可选条件：

1. 必须在`override init(style: UITableViewCellStyle, reuseIdentifier: String?)`中布局
2. 必须指定`self.hyb_lastViewInCell`
3. 对于UILabel控件，必须指定`preferredMaxLayoutWidth`，而且它的值要与自动布局所指定计算得到的宽必须保持一致，否则都会导致计算有误差。之前Masonry版本的，指定这个属性只是为了适配iOS6，但是对于SnapKit版本，必须要指定。看来现在是SnapKit版本还不够完善。
4. 可选指定`self.hyb_bottomOffsetToCell`，默认为0

下面是demo中一个例子，这里是按顺序添加约束的。

```
override init(style: UITableViewCellStyle, reuseIdentifier: String?) {
  super.init(style: style, reuseIdentifier: reuseIdentifier)
  
  self.contentView.addSubview(headImageView)
  headImageView.snp_makeConstraints { (make) -> Void in
    make.left.top.equalTo(15)
    make.width.height.equalTo(80)
  }
  headImageView.image = UIImage(named: "head")
  
  // title
  self.contentView.addSubview(titleLabel)
  titleLabel.numberOfLines = 0
  titleLabel.font = UIFont.systemFontOfSize(26)
  titleLabel.snp_makeConstraints { (make) -> Void in
    make.left.equalTo(headImageView.snp_right).offset(15)
    make.right.equalTo(-15)
    make.top.equalTo(headImageView)
  }
  
  // 若不指定preferredMaxLayoutWidth属性，则计算会不准备，使用Masonry时，指定此属性
  // 是特别适配iOS6的，不过使用SnapKit则必须指定，否则自动计算的高度会不准确
  let width = UIScreen.mainScreen().bounds.size.width
  titleLabel.preferredMaxLayoutWidth = width - 30 - 15 - 80;
  
  self.contentView.addSubview(descLabel)
  descLabel.numberOfLines = 0
  descLabel.font = UIFont.systemFontOfSize(22)
  descLabel.snp_makeConstraints { (make) -> Void in
    make.left.right.equalTo(titleLabel)
    make.top.equalTo(titleLabel.snp_bottom).offset(15)
  }
  descLabel.preferredMaxLayoutWidth = titleLabel.preferredMaxLayoutWidth
  
  descLabel.userInteractionEnabled = true
  let tap = UITapGestureRecognizer(target: self, action: Selector("onTapDesc"))
  descLabel.addGestureRecognizer(tap)
  
  self.contentView.addSubview(blogSummaryLabel)
  blogSummaryLabel.numberOfLines = 0
  blogSummaryLabel.font = UIFont.systemFontOfSize(22)
  blogSummaryLabel.snp_makeConstraints { (make) -> Void in
    make.left.right.equalTo(descLabel)
    make.top.equalTo(descLabel.snp_bottom).offset(15)
  }
  blogSummaryLabel.preferredMaxLayoutWidth = titleLabel.preferredMaxLayoutWidth
  
  let tap1 = UITapGestureRecognizer(target: self, action: Selector("onTapBlog"))
  blogSummaryLabel.userInteractionEnabled = true
  blogSummaryLabel.addGestureRecognizer(tap1)
  
  self.contentView.addSubview(okButton)
  okButton.setTitleColor(UIColor.whiteColor(), forState: .Normal)
  okButton.setTitle("计算高度的参考", forState: .Normal)
  okButton.backgroundColor = UIColor.greenColor()
  okButton.snp_makeConstraints { (make) -> Void in
    make.right.equalTo(-15)
    make.height.equalTo(45)
    make.top.equalTo(blogSummaryLabel.snp_bottom).offset(15)
  }
  
  blogSummaryLabel.backgroundColor = UIColor.redColor()
  descLabel.backgroundColor = UIColor.greenColor()
  
  let lineLabel = UILabel()
  lineLabel.backgroundColor = UIColor.lightGrayColor()
  self.contentView.addSubview(lineLabel)
  lineLabel.snp_makeConstraints { [unowned self] (make) -> Void in
    make.height.equalTo(1)
    make.left.equalTo(15);
    make.right.equalTo(0)
    make.bottom.equalTo(self.contentView)
  }
  
  // 指定最后一个视图，作为计算高度的参考
  self.hyb_lastViewInCell = okButton
  self.hyb_bottomOffsetToCell = 15
}
```

#如何动态修改约束

当cell要有动态地展开、收缩等功能时，每次都自己去计算？每次修改约束很复杂？不，不，不！其实很容易的。下面看看笔者是如何做到的。demo提供了展开与收起的功能，因此使用两个变量来记录是展开还是收起，而且还使用了是否是同一种状态的判断，防止每次都重新更新，这样体验会好很多哦。因为我们这里只是简单地对label操作，展开时有多少字就显示多少，因此不能手动指定其高度。收起时，只能指定其高度，因此在收起时通过更新约束API就可以添加高度约束了。但是对于展开并不能直接使用更新约束API去掉收起操作所指定的高度，因此只能使用移除之前的所有约束而重新添加约束的API来完成：

```
// MARK: Public
func config(testModel model: TestModel) {
  titleLabel.text = model.title
  descLabel.text = model.desc
  blogSummaryLabel.text = model.blog
  
  if model.isExpand1 != self.isExpand1 {
    self.isExpand1 = model.isExpand1
    
    if self.isExpand1 {
      descLabel.snp_remakeConstraints(closure: { (make) -> Void in
        make.left.right.equalTo(titleLabel)
        make.top.equalTo(titleLabel.snp_bottom).offset(15)
      })
    } else {
      descLabel.snp_updateConstraints(closure: { (make) -> Void in
        make.height.lessThanOrEqualTo(55)
      })
    }
  }
  
  if model.isExpand2 != self.isExpand2 {
    self.isExpand2 = model.isExpand2
    
    if self.isExpand2 {
      blogSummaryLabel.snp_remakeConstraints(closure: { (make) -> Void in
        make.left.right.equalTo(descLabel)
        make.top.equalTo(descLabel.snp_bottom).offset(15)
      })
    } else {
      blogSummaryLabel.snp_updateConstraints(closure: { (make) -> Void in
        make.height.lessThanOrEqualTo(55)
      })
    }
  }
}
```

#Version 1.0

* 扩展UITableViewCell，增加自动计算行高的扩展API

#Version 1.1

* 扩展UITableView，增加对指定tableview的对应的model数据高度缓存
* 增加可缓存高度的API，调用方式如下：

```
// MARK: UITableViewDelegate
func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
  let model = self.dataSource[indexPath.row]
  
  var stateKey = "";
  if model.isExpand1 && model.isExpand2 {
    stateKey = "expand1&expand2"
  } else if !model.isExpand1 && !model.isExpand2 {
    stateKey = "unexpand1&unexpand2"
  } else if model.isExpand1 {
    stateKey = "expand1&unexpand2"
  } else {
    stateKey = "unexpand1&expand2"
  }
  
  return TestCell.hyb_cellHeight(forIndexPath: indexPath, config: { (cell) -> Void in
    let itemCell = cell as? TestCell
          itemCell?.config(testModel: model)
    }, cache: { () -> (key: String, stateKey: String, cacheForTableView: UITableView) in
      return (String(model.modelId), stateKey, tableView)
  })
}
```

其中，cache闭包要求返回一个元组，指定key,stateKey,tableview，且key必须保证唯一，对应使用model的id作为key。而stateKey是用于存储/获取不同状态下的model对应的数据就有的高度。最后，tableview是给哪个tableview缓存。当tableview被释放了，缓存数据也会随着自动被释放掉。

#version 1.1.2

* 新增对某种状态更新缓存的API。主要是对于像QQ空间里的说说这样，在评论中增加一条评论，其高度也会变化，因此需要更新缓存。

```
/**
带缓存功能、自动计算行高
   
- parameter indexPath:					index path
- parameter config:						在回调中配置数据
- parameter cache:							指定缓存key/stateKey/tableview
- parameter stateKey:					stateKey表示状态key
- parameter shouldUpdate       是否要更新指定stateKey中缓存高度，若为YES,不管有没有缓存 ，都会重新计算
- parameter cacheForTableView: 指定给哪个tableview缓存
   
- returns: 行高
*/
public class func hyb_cellHeight(forIndexPath indexPath: NSIndexPath,
config: ((cell: UITableViewCell) -> Void)?,
updateCacheIfNeeded cache: ((Void) -> (key: String, stateKey: String, shouldUpdate: Bool, cacheForTableView: UITableView))?) -> CGFloat 
```

#最后

写代码不易，开源更不易，且用且珍惜！当使用过程中出现bug时，请一定要反馈到原作者这里，一起来维护和完善此开源小项目。

#源代码

支持cocoapods，大家可以通过下面的命令放到Podfile中：

```
pod 'HYBSnapkitAutoCellHeight', '~> 1.1.2'
```

或者直接到GITHUB下载源代码，将HYBSnapkitAutoCellHeight文件夹放到工程中：[HYBSnapkitAutoCellHeight开源](https://github.com/CoderJackyHuang/HYBSnapkitAutoCellHeight)

#看Objective-C版

笔者提供了Objective-C基于Masonry扩展的版本：[开源HYBMasonryAutoCellHeight自动计算行高](http://www.henishuo.com/masonry-cell-height-auto-calculate/)

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**[原文出自：标哥的技术博客](http://www.henishuo.com/snapkit-auto-cell-height/)**


#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
