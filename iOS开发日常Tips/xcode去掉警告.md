#引言


在我们的项目中，通常使用了大量的第三方代码，这些代码可能很复杂，我们不敢改动他们，可是作者已经停止更新了,当sdk升级或者是编译器升级后，这些遗留的代码可能会出现许许多多的警告，那么我们有没有办法去掉这些烦人的警告，不然一个工程几百个警告，你看着怎么都不爽吧。我们怎么去掉警告呢？

* 最直接、最一劳永逸、最安全的方式，直接找到警告的那段代码，改为不警告。这个方式最安全。

可是它有一个问题，就是，当我们很多文件都有这种类型的警告的时候，我们就需要改动很多很多的源码了, 对于不是我们写的源码，有可能随时会更新的，我们这种方式，显然就不太可取了。

* 使用编译器提供的宏来操作，这个方式在我们的工程中会大量的看到：

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored"-Wdeprecated-declarations"

// 写在这个中间的代码,都不会被编译器提示-Wdeprecated-declarations类型的警告
dispatch_queue_tcurrentQueue =dispatch_get_current_queue();
#pragma clang diagnostic pop
```

这种方式的问题,同第一个差不多,也是要修改源代码的实现的,对于第三方,我们肯定是不想改动它的,尤其是一些更新很频繁的第三方,一般警告出现后不久,作者就更新了,我们在此做这样的操作,就显得浪费了.并且在 添加arm64支持的时候,一下出现几百个某种类型的警告,改起来也是相当费时费力的啊!

比如我们的工程,打开了`arm64`,然后编译：

![image](http://cc.cocimg.com/api/uploads/20141218/1418867357768119.png)

#关闭某一个指定文件的某种指定类型的警告


这里拿一个具体工程来说吧.比如我们工程里有一个文件`PresencePacket`：

![image](http://cc.cocimg.com/api/uploads/20141218/1418867427502880.png)

其实关闭某个指定文件的某种类型的警告很简单,就如同我们以前给某一个文件添加`ARC`支持或者不支持的时候那样 添加 忽略/显示 某种类型警告

![image](http://cc.cocimg.com/api/uploads/20141218/1418867735947216.png)

双击 文件, 在其中添加`-Wno-shorten-64-to-32 ` (这个关键在就是让编译器忽略 `Implicit conversion loses integer precision: 'NSInteger' (aka 'long') to 'int32_t' (aka 'int')` 警告)

![image](http://cc.cocimg.com/api/uploads/20141218/1418867761707782.png)

添加完成后,再编译,那么`PresencePacke`t文件中的  `Implicit conversion loses integer precision: 'NSInteger' (aka 'long') to 'int32_t' (aka 'int’)`警告就没有了,是不是很简单,很方便.

这种方式,已经是大大的减少了工作量了,只需要在指定的文件的编译中添加`-Wno-shorten-64-to-32`就可以了.那么有没有什么方式可以让编译器忽略整个工程中的 指定类型的警告呢?

#关闭工程中指定类型的警告


这个最简单了, 工程的`target`有一个`Other Warning Flags`

![image](http://cc.cocimg.com/api/uploads/20141218/1418867786316385.png)

在其中添加`-Wno-shorten-64-to-32`

![image](http://cc.cocimg.com/api/uploads/20141218/1418867823201657.png)

再重新编译,哈哈,整个文件中的`Implicit conversion loses integer precision: 'NSInteger' (aka 'long') to 'int32_t' (aka 'int’)` 警告全部消失了!!!!

#查看警告类型


大家可能很疑惑,上面的`-Wno-shorten-64-to-32` 是怎么来的,我怎么知道`Implicit conversion loses integer precision: 'NSInteger' (aka 'long') to 'int32_t' (aka 'int’)` 警告 就是 -Wno-shorten-64-to-32类型呢?这里,其实不需要记忆的,当工程中有这种类型警告的时候

在警告窗口,某个警告上,我们右击,显示出右键菜单,选择其中的`Reveal in Log`

![image](http://cc.cocimg.com/api/uploads/20141218/1418867853307660.png)

则会显示：

![image](http://cc.cocimg.com/api/uploads/20141218/1418867871817135.png)

注意到其中`[-Wshorten-64-to-32]`,在这个括号中的就是 这种警告的类型`-W`是前缀,这个前缀表示的是 打开这种类型的警告 如果我们是要关闭某种类型的警告的话, 要将`-W`换成`-Wno-`,这样就得到了`-Wno-shorten-64-to-32`了.

#后记

对于我们使用`cocoapod`引入的第三方,我们可以在`podfile`文件中 增加一句  `inhibit_all_warnings!` 来要`pod`的工程不显示任何警告，例如

```
link_with 'SecondHouseBrokerAPP','SecondHouseBrokerCOM'
platform :ios,'6.0'
inhibit_all_warnings!
 
 
pod 'CocoaAsyncSocket'
pod 'Reachability'
pod 'ProtobufObjC'
pod 'SDWebImage'
pod 'FMDB'
pod 'GPUImage'
pod 'CXPhotoBrowser'
pod 'CocoaLumberjack'
```

还有就是,上面的方法也适合其它类型的警告!!!

