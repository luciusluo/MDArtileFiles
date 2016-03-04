#前言

对于刚毕业出来或者刚培训出来的`ios`开发人员，你们在短短的时间里学习了一系列的课程，也做了一些实战性训练，但是仅仅这样就真的可以去工作了吗？不是的。刚进公司时，也许你能在面试时靠自己的三寸不烂之舌高薪就职，但是如果能力不够，工作一段时间后，就会出各种问题，公司也会记录着每个人的成长情况。好一点的公司，都会关注每个员工的成长，会记录每位员工为公司所做的技术贡献等。

本人在公司中一直都是充当技术支持的角色，任何一位同事遇到不好解决或者不会解决的问题，都可以找我。如果是小问题，通常直接给方案如何去解决。如果是不容易解决的问题，通常会帮助同事分析可能的原因及给出解决的思路或者步骤。

#开发调试

在开发中一定需要进行各种调试，如果连调试都不会，如何去定位问题所在？在您还没有阅读过本篇文章前，你确定掌握了多少调试技巧？相信对你一定很有帮助！

* [iOS LLDB调试技巧精讲](http://www.henishuo.com/ios-lldb-debug-tech/)

#项目版本控制

在公司做开发，通常都是几个人甚至几十号人共同开发，当然也有些小公司是一个人自己顶着，做出来是什么样就怎么样。对于这种小公司，通常就是给刚出来的人练练手，掌握点经验以便更好的跳槽。大家说说这么理解是不是正确呢？个人认为是正常的。谁都不想被压榨，谁都想付出得到相应的回报。在小公司，一个人在学习，根本追不上团队开发学习成长得快。

那么，在公司中如何管理项目的开发呢？现在，各公司用于项目版本控制的通常有两种：

* git：使用`git`的公司是最多的，大多都采用`git`，像`github`、`OSChina`等都是支持免费的`git`仓库的。实际上，在公司几乎都是公司自己搭建一个`git`服务器。
* svn：对于`svn`版本控制，还是有一部分公司在使用的，原来我在两家公司使用过`svn`，说真的，比`git`使用起来困难多了。

对于公司开发必备这两样神器，你会用了吗？

如果你也想学习如何自己在远端服务器搭建`git`服务器，这里有一篇笔者搭建`git`服务器时的记录，以及写了git和svn在开发中的使用教程记录：

* [CentOS上搭建git服务器](http://www.henishuo.com/centos-git-install/)
* [ios开发中git的使用](http://www.henishuo.com/git-use-inwork/)
* [ios使用cornerstone的svn工具](http://www.henishuo.com/mac-cornerstone-svn-use/)

#Xcode必备神器

做`iOS`开发，一定离不开`Xcode`这个开发工具。那么，如何提高开发的效率，就是我们作为开发人员应该思考的问题。这里是笔者总结自己当前所使用的`Xcode`部分插件，可提高开发的效率。可以说，现在的我，已经离不开这些插件了。

* [Xcode插件：十大神器](http://www.henishuo.com/xcode-plugin/)

#第三方库管理

如今的`App`开发，怎么可以不使用`Cocoapods`来管理呢？这里有一篇文章讲解如何使用！

* [Cocoapods的使用](http://www.henishuo.com/cocoapods-use/)

#网络库学习


做开发，一定会用到网络接口。因此，我们必须要掌握如何调用接口。这里笔者写了一个基于`AFNetworking`封装的类，可直接放到工程中使用。直接使用`AFNetworking`还需要做一些配置，因此对于新入门的朋友来说，也是挺困难的一件事。看完这篇文章，直接使用即可。

* [基于AFNetworking2.5封装网络库](http://www.henishuo.com/base-on-afnetworking-wrapper/)
* [基于AFNetworking3.0网络封装](http://www.henishuo.com/base-on-afnetworking3-0-wrapper/)

#自动布局

开发几乎都需要使用自动布局，对于笔者一直以来都是使用纯代码自动布局。笔者极力推荐Objective-C的Masonry和Swift的SnapKit。

笔者写了Masonry使用的教程专题，当前共有14篇文章，专题入口：

* [iOS Masonry Autolayout实战](http://www.henishuo.com/category/autolayout/)

笔者开源了基于SnapKit自动计算行高且带缓存的扩展：

* [开源HYBSnapkitAutoCellHeight自动算行高Swift版](http://www.henishuo.com/snapkit-auto-cell-height/)

笔者开源了基于Masonry自动计算行高的扩展：

* [开源HYBMasonryAutoCellHeight自动计算行高Objective-C版](http://www.henishuo.com/masonry-cell-height-auto-calculate/)

#关注我


关注                | 账号              | 备注
-------------      | -------------     | ----------------
Swift/ObjC技术群一  | 324400294         |  群一若已满，请申请群二
Swift/ObjC技术群二  | 494669518         | 群二若已满，请申请群三
Swift/ObjC技术群三  | 461252383         | 群三若已满，会有提示信息
关注微信公众号       | iOSDevShares      | 关注微信公众号，会定期地推送好文章
关注新浪微博账号      |  [标哥Jacky](http://weibo.com/u/5384637337) | 关注微博，每次发布文章都会分享到新浪微博
关注标哥的GitHub     | [CoderJackyHuang](https://github.com/CoderJackyHuang) | 这里有很多的Demo和开源组件
关于我               | [进一步了解标哥](http://www.henishuo.com/about-biaoge/) | 如果觉得文章对您很有帮助，可捐助我！



