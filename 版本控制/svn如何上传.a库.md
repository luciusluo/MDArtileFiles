#前言


在`MAC系统`下，`SVN`不会自动上传`.a`库，导致经常报错，都是因为`.a`库没有上传上去，于是其他同事一拉代码就各种报错。当年在第一家公司工作的时候，使用的是svn，出现了很多的问题，主要还是个人不了解它的使用。这里给大家讲讲可能大家也会遇到的问题。

#解决方案


解决方法其实很简单，就是使用终端命令来操作。具体操作步骤为：

* 第一步：打开终端 
* 第二步：输入`cd 空格 .a库所在的目录`（拖动目录到终端即可） 
，例如，要上传`SinaWeiboSDK`包里的`.a`库：

```
cd /Users/huangyibiao/Documents/公司的项目/XiaoYaoUser/XiaoYaoUser/OpensourceLibraries/SDKs/ShareSDK/Extend/SinaWeiboSDK 
```

* 第三步：输入`svn add .a库名称`，例如：
 
```
 svn add libSinaWeiboSDK.a
```

此时，如果终端提示：

```
A  (bin)  .a库名称
```

例如这里上传的是`libSinaWeiboSDK.a`，会出现下面的提示：

```
A  (bin)  libSinaWeiboSDK.a
```

如果出现上面这一步的捍不，就说明成功地上传了`.a`库到`SVN`远端仓库了，这时到`svn`图形管理软件看一下，就出现了该库，再`commit`就可以上传了。

#推荐SVN图形软件


曾经所在的两家公司都是使用`SVN`，其中一家公司购买了`CornerStone`这个软件，因此可以正常使用。如果公司没有购买收费的`CornerStone`这个软件，还就使用破解版吧。


