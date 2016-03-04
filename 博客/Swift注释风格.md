#前言
---

良好的注释，有助于开发和维护，请正视注释！在`Swift2.0`之前与之后是不一样的，这里是基于`Swift2.0`的。

#看看ObjectiveC中常用的注释

---
看看下面是笔者对`UIActionSheet`封装的一个通用方法的注释：

```
/**
* @author huangyibiao
*
* Block版本的actionSheet，具体如何使用请参考UIActionSheet
*
* @param inView 父视图
* @param title 标题
* @param cancelTitle 取消按钮标题
* @param destructiveTitle destructive按钮标题
* @param otherTitles 其它按钮的标题
* @param callback 按钮点击的回调
*
* @note 最多支持30项
* @see UIActionSheet
* @see HDFActionSheetClickedButtonBlock
*
* @return 所呈现的UIActionSheet视图
*/
+ (UIActionSheet *)hdf_showInView:(UIView *)inView
                           title:(NSString *)title
                     cancelTitle:(NSString *)cancelTitle
                destructiveTitle:(NSString *)destructiveTitle
                     otherTitles:(NSArray *)otherTitles
                        callback:(HDFActionSheetClickedButtonBlock)callback;
```

上面只是一个API的注释例子，让我们分析：

* API功能性描述：可以使用@brief标识，也可以省略
* 参数说明：使用@param 参数名称 参数功能描述
* 作者说明：使用@author 作者名称
* 特别说明：使用@note 注意事项说明
* 参考说明：使用@see 类型名称
* 返回值：    使用@return 返回值说明
这是比较标准的写法了。

#在Swift2.0中的注释方式

---
在swift中就不推荐使用/\*\*/的方式来注释API头了，因为我们发现输入/\*\*/时，不会自动对齐了，这就显得难看了。更多地，我们会使用`///`来注释。

```
///  Get the url of category of technology
///
///  - parameter currentPage: Current page index
///  - parameter pageSize:    How many rows to load
///
///  - returns: The absolute url
static func technologyUrl(currentPage: Int, pageSize: Int) ->String {
  let url = "ArticleServer/queryArticleListByCategory/2"
   
  return baseUrl + "\(url)/\(currentPage)/\(pageSize)"
}
```

由此简单的API注释可以看出来：

* 注释使用`///`的方式来添加，苹果的API都使用了这种方式
API功能性描述，直接添加说明即可
* 参数说明：使用 - `parameter` 参数名: 参数功能说明
* 返回值说明：使用 - `returns`: 返回说明

当然，我们仍然可以使用/\*\* \*/的方式，只是与苹果的方式保持一致，我相信会更好一些。事实上并不只是这些，在Swift中注释是支持markdown语法的，因此只要您会Markdown语法，就可以添加超链接，图片，GIF图片等。看看下面一张截图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/0.jpeg)

这是一张GIF图，是可以看到动画的哦。

在参数中添加图片，只需要遵守Markdown语法即可轻松添加：`![image](图片链接)`，这是固定的格式。
在参数中添加链接，只需要`[显示的名称](超链接)`
这是不是很强大呢？当组件化时，只需要将图片放在注释中，外部开发人员只需要按下option+鼠标点击就可以看到效果了，比千言万语要容易懂。

注意：对于一个API头会采用上面的注释方式，但是在API内部，我们仍然会使用//或者/\*\*/来注释某一行。对于代码块注释，我们也可以采用Markdown方式来注释，添加图片，链接等。

##自动生成注释插件
作为开发人员，怎么能不使用Xcode的插件呢，这有助于我们飞得更快。
在这之前，我们先安装一个插件管理器，在终端输入：

```
curl -fsSL https://raw.githubusercontent.com/supermarin/Alcatraz/deploy/Scripts/install.sh | sh
```

安装完成后，就可以搜索`VVDocument`安装。

看下图，重启`Xcode`后，在`Xcode`的菜单栏`window`中有一个`Package Manager`，点击就可以打开`Alcatraz`的界面，在搜索框中输入`VVDocument`就可以搜索出来了，然后点击`Install`即可，安装后需要重装启动`Xcode`。如下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-12-上午11.25.41.png)

重启`Xcode`后，在`Window`中有一个叫`VVDocument`的，点击它，就会显示如下界面，在这里就可以设置触发方式等，如下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-12-上午11.26.36.png)

上图为本人的设置界面，个人习惯以输入`///`来触发自动生成注释。

#***原文来自笔者专属博客：***[阅读原文](http://www.henishuo.com/swift-comment-api-style/)

#关注我

---
**微信公众号：[iOSDevShares](http://www.henishuo.com/)**<br>
**有问必答QQ群：[324400294](http://www.henishuo.com/)**
