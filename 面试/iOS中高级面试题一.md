#前言

以下问题为朋友们问我的问题，笔者在这里统一根据个人理解和所学提供参考，如有任何疑问或出错之处，请在评论处评论或者直接联系笔者。谢谢！

#1.两个App之间如何互调，又如何传参？


**具体问题如下：**

由一个app1跳转到app2之后，app2完成某项任务之后，怎么把app2的完成信息传到app1（自己的程序是app1），传的是什么类型的数据，怎么进行解析？

**参考答案：**

笔者针对这个问题，写了一篇文章并提供demo下载：[iOS App之间如何通信](http://www.henishuo.com/ios-app-communication/)

#2.Socket在传输过程中用的是什么类型的数据：结构体还是文本？如果是文本，那么怎么进行编码和解码？


**参考答案：**

笔者学习相关socket知识，写了这几篇文章，可以参考：

[iOS Socket理论知识](http://www.henishuo.com/ios-socket-theory/)

[iOS Socket UDP编程-C语言版](http://www.henishuo.com/ios-socket-udp-c-version/)

[iOS Socket TCP编程-C语言版](http://www.henishuo.com/ios-socket-tcp-c-version/)

[iOS Socket编程-Objective-C原生API版]()

[iOS Socket编程-Objective-C基于CocoaAsyncSocket版]()

[iOS Socket编程-Swift原生API版]()

[iOS Socket编程-Swift基于AsyncSocket版]()

#3、图片压缩处理问题

**参考答案：**

关于图片的压缩处理，在ios中常用的方法是先处理像素再处理尺寸。大家可以参考笔者所写的这篇处理压缩处理文章：

[iOS 图片压缩处理](http://www.henishuo.com/ios-image-compressed/)
#4、在做图片优化处理的时候，怎么将一个原图比较小（或大）的image放到一个固定的imageview中，如何减少失真度，除了这个，图片优化还有哪些方法？

**参考答案：**

最直接的办法就是使用UIImageView的contentMode，让其自动适应，但是这样会消耗一定的性能。

如果要优化，可以通过手动将原图处理成UIImageView大小的图，再给它呈现。
#5、如何高效的对各种控件（label、button）进行切圆角、透明度、加边框（备注：不使用layer的方法）？

**参考答案：**

[iOS 高效添加圆角效果实战讲解](http://www.jianshu.com/p/f970872fdc22)
# 6、多线程在实际现实中有哪些应用？（网络操作和大量图片处理不算）

**参考答案：**

通常耗时的操作都会放在子线程里处理，然后再回到主线程来显示。下面举几个例子：

1. 我们要从数据库提取数据还要将数据分组后显示，那么就会开个子线程来处理，处理完成后才去刷新UI显示。
2. 拍照后，会在子线程处理图片，完成后才回到主线程来显示图片。拍照出来的图片太大了，因此要做处理。
3. 音频、视频处理会在子线程来操作
4. 文件较大时，文件操作会在子线程中处理
5. 做客户端与服务端数据同步时，会在后台闲时自动同步

# 7、如果app比较大，怎么样减少app的大小？

**参考答案：**

1. 将build setting中的Optimization Level设置为Fastest, Smallest [-Os]，在发布模式下，默认就是这样设置的
2. 将build setting 中的Strip Debug Symbols During Copy设置为YES，在发布模式下，默认就是这样设置的
3. 资源文件查找出所有未使用的，去掉这些永远不会使用的资源文件
4. 对嵌入App的音频进行压缩处理

就想到这些了，大家再补充吧~
# 8、你在迭代开发中是怎么处理版本兼容问题？

**参考答案：**

版本迭代一定要注意兼容老版本，比如新增了字段或者去掉了某些不再使用的字段，不能引起应用闪退。我们这里只谈程序代码兼容新老版本问题，不考虑业务。因为业务是要求后台来兼容的，通常接口会有版本号控制，用于兼容不同版本的客户端。

对于任何一个App，当可以升级的时候，不会是所有用户就立刻去升级，通常会有很大一部分的用户是不愿意立刻升级的。原因会有很多种，比如我这种的就不会频繁升级，因为对于我来说，这个App并不是天天用，没有必要升级。

那么，我们在iOS开发时，如何去兼容老版本的，保证新版本的增加或者删减不会影响到老版本呢？其实这个问题似乎并不是说有没有新、老版本问题，更重要的是程序的健壮性问题。

对于我们做前端的，永远不要相信后台一定会按照原先约定返回我们想要的数据结构以及所有字段。

假设接口返回来的数据是这样的，我们需要通过类型判断，确保不会因为接口变化返回无效数据而引起闪退。当然，当接口返回的数据结构与我们原先约定的不一样时，通常是因为后台出错了，因此为了程序更健壮，我们应该要容得下后台的错误：

```
AFHTTPRequestOperation *op = [self PostRequestWithUrl:url params:params completion:^(id responseObject) {
  BOOL isSuccess = NO;
  if ([responseObject isKindOfClass:[NSDictionary class]]) {
    
    NSDictionary *response = responseObject[@"response"];
    if ([response isKindOfClass:[NSDictionary class]]) {
      
      NSArray *resultList = response[@"resultList"];
      if ([resultList isKindOfClass:[NSArray class]]) {
        NSArray *listModels = [HYBCosmesisModel objectArrayWithKeyValuesArray:resultList];
        isSuccess = YES;
        completion(listModels);
      }
    }
  }
  
  if (!isSuccess) {
    // 表示出错
    completion(nil);
  }
} errorBlock:^(NSError *error) {
  errorBlock(error);
}];
```

而我们在使用的时候，对于字符串、数组、字典都应该要做一下类型判断和空处理。比如：

```
if ([response isKindOfClass:[NSDictionary class]]) {
  NSString *value = response[@"blogName"];
  if (!kIsEmptyString(value)) {
     // Do my job
  }
  
  NSArray *array = response[@"array"];
  if ([array isKindOfClass:[NSArray class]]) {
    // Do my job 
  }
}
```

对于业务方面的话，由后台来做版本控制，通过在接口做添加公共参数，比如version=@“2.2”来判断客户端的版本以及接口需要调用的是哪个版本的接口。
# 9、你和后端服务器是怎么进行交互的

**参考答案：**

这个问题非常简单，但是对于新手就不太清楚了。在很多小公司里，前端好像什么都不需要管，只等后台给你一个接口及参数说明就可以了，根本不清楚后端为什么要这么设计这个接口。

那笔者也来聊聊如何与后端服务器交互：

1. 在需求确定，定下了开发的周期后，就需要准备开发了。
2. iOS、安卓及后端各端主要负责人员在了解完需求后，开始分析本期需求需要哪些接口，是否需要新的接口，是否需要改动原有的接口等。在三端统一后，后端接口负责人确定哪天出接口文档及接口假数据。为什么要三端一起定接口呢？因为即使是后端接口负责人，也不一定对原有的业务和原有的接口全部都了解，任何一方的了解加起来才能确定是否可行。
3. 如果App还没有开发过，首先开发一款新的App，那么iOS、安卓端的架构师，或者主要开发负责人，需要与后端接口负责人共同分析需求，然后初步写出第一版本接口文档，然后各方再各自好好看看、分析分析接口是否合理，参数是否合理，结构是否合理等。这数据的结构会决定着整个网络框架的搭建。

好了，就扯谈这些吧~
# 10、怎么用GCD加载多张图片之后，把图片放到融合到一张图片里？

**参考答案：**

使用Dispatch Group追加block到Global Group Queue,这些block如果全部执行完毕，就会执行Main Dispatch Queue中的结束处理的block。

当放到group中的所有请求都完成时，才会回调dispatch_group_notify的block：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    // 合并图片
}); 
```
# 11、使用GCD的时候，如何在一个group里添加几个任务的依赖关系（这几个任务放在一个组中）**参考答案：**

这个不太清楚想问什么，笔者翻看了看GCD中dispatch_group_t里，也没有什么可以设置同一个组内的任务依赖关系的，就看到dispatch_group_wait这个API：

```
long
dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout);
```这个API是等待group中的所有任务都执行完毕才能继续往下执行其它任务。它是同步地等待任务执行完毕。比如，A、B、C、D四个任务，要求A、B执行完毕后，C、D才能开始执行，那么可以通过这样做：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, queue, ^{ /* 任务A */ }); 
dispatch_group_async(group, queue, ^{ /* 任务B */ }); 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    dispatch_group_async(group, queue, ^{ /* 任务C */ });
	dispatch_group_async(group, queue, ^{ /* 任务D */ });
}); 
```

或者可以这样：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, queue, ^{ /* 任务A */ }); 
dispatch_group_async(group, queue, ^{ /* 任务B */ }); 
// 同步等待A、B执行
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
dispatch_release(group);

// 重新创建组
group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /* 任务C */ });
dispatch_group_async(group, queue, ^{ /* 任务D */ });
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    // C、D执行完毕后，想干嘛就干嘛去吧 
}); 
```不知道笔者对题目的理解是否到位，上面的代码是随手写的，可能单词会写错~~~# 11、使用SDWebImage的时候，从服务器请求回来的头像URL没有变化，但是用户已经修改过头像，由于缓存的原因，不能显示出最新修改的用户的头像。在不去掉缓存的条件下，如何显示出最新的头像，给出策略。


**参考答案：**

这个问题，笔者在另外一篇文章中写到，然后也在微博上收集了很多朋友们的想法。请大家稳步去阅读吧！

[iOS如何在用户修改头像后正常显示](http://www.henishuo.com/ios-image-change-reload-cache/)
