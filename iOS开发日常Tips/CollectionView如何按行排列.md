#概述

记得前段时间有小伙伴问过UICollectionView如何让它按行排列，也就是当前行排满了再换到下一行来排列。当前没在意，也没有回复他！给下期需求封装一个日历小控件，UI还没有出来，不过功能大致会差不到哪里去，所以大家忽略效果图中显示的UI。

问题需要是很简单的，不过既然让我遇到了，那就记录下来吧，给后面的人的提个醒吧！

先看看效果是这样的：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160411-0@2x.png)

#解决方案

看到这样子是不是觉得好奇怪？当然挺有奇怪的啦！

我们在创建FlowLayout时是这样的：

```
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
layout.scrollDirection = UICollectionViewScrollDirectionHorizontal;
```

我们指定了水平滚动，于是就变成了上面的效果图的样子。

如何解决呢？方法是：

```
UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc] init];
layout.scrollDirection = UICollectionViewScrollDirectionVertical;
```

然后效果就变成了我们想要的样子了：

![image](http://www.henishuo.com/wp-content/uploads/2016/04/QQ20160411-1@2x.png)

#结尾

给新手们的福利，请高手们不要问我这么简单为什么还要写下博客！原因很简单，记录下来并比不记录下来的要强！



