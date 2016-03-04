#前言

关于本地图片UIImage的加载问题，还是需要注意的。不同的加载处理方式，在效率和性能上还是有差异的。

今天，我们来讲讲UIImage的加载应该选择什么样的API来加载！

#两种API

这两种API分别是：

* -imageNamed: 默认加载图片成功后会**内存**中缓存图片，这个方法用一个指定的名字在系统缓存中查找并返回一个图片对象。如果缓存中没有找到相应的图片对象，则从指定地方加载图片然后缓存对象并返回这个图片对象。通常是加载bundle中的图片资源！
* -initWithContentsOfFile: 仅仅加载图片而不在内存中缓存下来，那么每次获取时都会重新去加载。

#使用场景

* -imageNamed: 是读取到内存后会缓存下来，下次再读取时直接从缓存中获取，因此访问效率要比较高。对于图片资源比较小，使用比较频繁的图片，通常会选择使用此种方式来加载。当然，若不需要考虑性能时，直接使用此种方式也是可以的。

* -initWithContentsOfFile: 当图片资源比较大，或者图片资源只使用一次就不再使用了，那么使用此种方式是最佳方式。当应用程序需要加载一张比较大的图片并且是一次性使用的，那么是没有必要去缓存这个图片的，用-imageWithContentsOfFile:是最为经济的方式，这样不会因为UIImage元素较多情况下，CPU会被逐个分散在不必要的缓存上而浪费过多CPU时间。另外，当我们的图片不是PNG图片时，我们通常会选择此种方式来加载。

大量使用-initWithContentsOfFile:方式来加载图片，会增加CPU的开销，所以我们需要根据特定场景慎重选择图片加载的方式。即使UIImage较小，但使用UIImage元素较多时，问题会有所凸显哦！

#代码使用

* 对于-imageNamed: 这个API的调用就非常简单了，直接就是：

```
UIImage *image = [UIImage imageNamed:@"logo"];

// 在开发中，通常都定义了快捷调用的宏
#define kImgName(name) [UIImage imageNamed:name]

// 使用时就更简化了
UIImage *image = kImgName(@"logo");
```

* 对于-initWithContentsOfFile:的使用就相对复杂了一点点：

```
NSString *filePath = [[NSBundle mainBundle] pathForResource:@"logo" ofType:@"png"];
UIImage *image = [[UIImage alloc] initWithContentsOfFile:filePath];

// 但是在开发中，笔者通常会定义成宏，简化调用
#define kResourcePath(name, type) ([[NSBundle mainBundle] pathForResource:name ofType:type])
#define kImgFromFile(name, type) [[UIImage alloc] initWithContentsOfFile:kResourcePath(name, type)]

// 然后，调用也变得很简化了~
UIImage *image = kImgFromFile(@"logo", @"png");
```

#最后

阅读到此，是否对UIImage有更深入地了解了？我相信这篇文章能更帮助您更好在地开发中使用图片加载。本篇文章所有内容不代表全正确，若有错误之处，请联系笔者或者在评论中指出！

#推荐阅读

* [iOS图片压缩处理](http://www.henishuo.com/ios-image-compressed/)

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



