#前言

普通的view布局是很简单的，只是添加上下左右就可以确定其位置及大小，可是使用Scrollview自动布局就很复杂了，
因为scrollview是没有固定的高度和宽度的，因为其宽度和高度是由其内容的大小所决定的，也就是所谓的contentSize所决定。
如果要使用自动布局，那么Scrollview的内容的大小不能依赖于scrollview的尺寸，否则就无法确定，就会发出警告。

这个是我们的效果图。下面看看怎么做的:

![image](http://img.blog.csdn.net/20150331115959734?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd29haWZlbjMzNDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


1. 根据官方提供的文档，如果我们直接把子控件放到Scrollview上，分别布局，是很难布局的，因此我们使用一个
`containerView`，这个`containerView`放所有的子控件，那么只要布局好所有的子控件与`containerView`之间的布局关系，
那么剩下的就是`containerView`与`Scrollview`之类的布局关系，这也就是所谓的**化零为整策略**。

2. 首先放一个`scrollview`到`view`上，然后设置`scollview`的约束：与`view`的`left`,`top`,`right`,`bottom`分别为0,84,0,0。
3. 放一个`view`作为`containerView`到`scrollview`中，作为子`view`，作为参考系：在右侧`document`中的`Label`加入标识：`containerView`，这个不一写需要添加，其实不加也没有关系，只是加了就更明确其用意。然后给`containerView`添加与`scrollview`的约束：上下左右分别设置为0。
4. 添加4个内容视图：分别加个标识为：`content1`,`content2`,`content3`,`content4`
给这个子视图添加约束中，`leading`和`trailing`约束分别是与`scrollview`的约束，而不是与`containerview`的约束，虽然`containerView`是这几个子视图的父视图。添加的约束：`leading`20，`trailing`20。
5. 给`content1`添加固定的宽和高：`280`，`100`。之所以要固定的宽和高，是因为不固定就无法确定`containerView`的宽高，也就无法确定`scrollview`的内容大小。然后添加`bottom`20，也就是与`content2`相距20
6、给`content2`添加`top`20,`bottom`20，固定的高`100`，这不需要设置固定的宽了，因为`content1`已经设置了。
同理，给`content3`添加`top`20,`bottom`20,固定的高`100`,给`content4`添加`top`20,`bottom`100(to supperview)，
这样`containerView`的宽就由`content1~content4`确定了，而高因为`content1`与`containerView`顶端相距`20`，`content4`的`bottom`与`containerView`相距100，也就确定了`containerview`的高度。而`containerview`也就是`scrollview`的内容，那么确定了`contaienrview`的内容大小，也就是确定了`scrollview`的内容的大小，所以自动布局也就完成了。

#总结


对于`scrollview`的自动布局，使用一个参考视图来布局，化零为整（把一堆子控件放到一个统一的视图中，即`containerView`），可以减少自动布局的难度，添加约束时，子视图的`leading`和`trailing`约束是相对于`scrollview`的，而不是`containerview`。

#Quetion


这样自动布局只是对宽度和高度确定（`content1`的宽和高被设置成固定的）才完成自动布局，如果
是动态的宽高呢，又该如何添加约束？完成这个`demo`后，提出了此问题，然后高手出来解答，然后在回复中
说明一下，或者给相关博客的链接，谢谢！！！！！

#源代码


下载源代码：[https://github.com/CoderJackyHuang/ScrollViewAutoLayout](https://github.com/CoderJackyHuang/ScrollViewAutoLayout)

#参考


* [https://developer.apple.com/library/ios/technotes/tn2154/_index.html](https://developer.apple.com/library/ios/technotes/tn2154/_index.html)
* [http://blog.csdn.net/kmyhy/article/details/39929117](http://blog.csdn.net/kmyhy/article/details/39929117)

#建议

本人推荐纯代码自动布局，可控性强，能适应需求快速变更。推荐使用`Masonry`第三方库，这个是`OjbectiveC`版本的，如果您是使用`Swift`开发，推荐使用`SnapKit`！！！

#推荐阅读

* [Masonry自动布局专题](http://www.henishuo.com/category/autolayout/)



