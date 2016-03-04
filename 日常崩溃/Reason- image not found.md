#前言

今天想帮徒弟fix个bug的，结果一真机运行时，出现了闪退，而且是必闪退。用模拟器是没有问题的，不过在真机好像是有问题，不确定是否是所有机型。使用iPhone6运行是必闪退的，在此记录下来，以便后来者可直接找到解决方案。

#崩溃日志

```
dyld: Library not loaded: @rpath/Pods.framework/Pods
  Referenced from: /var/mobile/Containers/Bundle/Application/3A970A35-A52F-45D7-A4D5-55C849349C75/QuanJie.app/QuanJie
  Reason: image not found
```

这是镜像找不到，而这里是Pods.framework找不到，而引起的必闪问题。

#解决办法一

老外的一段话：

Okay, fixed. The issue was the Pods.framework was set to required and not optional. It wasn't being copied to device by the .sh script and so the app crashed. Setting this to optional fixed the issue.

大概意思是说，这个问题是Pods.framework设置成required而不是optional。在运行时，它会被.sh脚本拷贝到设备上，因此导致app崩溃。将required修改为optional就可以fix这个问题了。

在哪里设置？如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/QQ20160229-0@2x-1.png)

#解决办法二

本篇文章发表到微博后，有不少朋友们关注了，说明大家也曾经遇到这样的问题。有朋友回复说，还有一种方式可以解决：

在Target->Generals->Embedded Binaries添加所提示找不到的framework。如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160301-0@2x.png)


#参考

* [CocoaPods issues 3586](https://github.com/CocoaPods/CocoaPods/issues/3586)


