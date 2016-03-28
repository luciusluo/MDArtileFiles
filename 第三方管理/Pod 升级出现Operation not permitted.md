#前言

最近突然调用pod search命令都是出问题，提示找不到方法什么的。于是笔者猜想这是pod版本的问题。因为上一次我手贱执行了安装beta版本的pod，导致现在老出问题。

于是我想尝试更新一下pod的版本到最新，可是出现了这样的提示：

```
[huangyibiao@huangyibiaodeMacBook-Pro ~] $ sudo gem install cocoapods
ERROR:  While executing gem ... (Errno::EPERM)
    Operation not permitted - /usr/bin/pod
```

尝试了多次还是一样。这有这可能是因为beta版本的原因。于是想起之前升级的时候pod所提示的命令pre，可以升级、安装beta版本。

#解决办法

```
[huangyibiao@huangyibiaodeMacBook-Pro ~] $ sudo gem install -n /usr/local/bin cocoapods --pre
```

执行上面的命令就可以升级到最新的beta版本。于是安装竟然真的成功了！看下面的执行过程：

```
Fetching: cocoapods-core-1.0.0.beta.4.gem (100%)
Successfully installed cocoapods-core-1.0.0.beta.4
Fetching: molinillo-0.4.4.gem (100%)
Successfully installed molinillo-0.4.4
Fetching: xcodeproj-1.0.0.beta.3.gem (100%)
Successfully installed xcodeproj-1.0.0.beta.3
Fetching: cocoapods-1.0.0.beta.4.gem (100%)
Successfully installed cocoapods-1.0.0.beta.4
Parsing documentation for cocoapods-core-1.0.0.beta.4
Installing ri documentation for cocoapods-core-1.0.0.beta.4
Parsing documentation for molinillo-0.4.4
Installing ri documentation for molinillo-0.4.4
Parsing documentation for xcodeproj-1.0.0.beta.3
Installing ri documentation for xcodeproj-1.0.0.beta.3
Parsing documentation for cocoapods-1.0.0.beta.4
Installing ri documentation for cocoapods-1.0.0.beta.4
4 gems installed
```

看到了吧，都是beta版本的，需要使用pre指定，不然总是提示权限问题。

#结尾

以后手不能太贱了，不然就得折腾自己了！

