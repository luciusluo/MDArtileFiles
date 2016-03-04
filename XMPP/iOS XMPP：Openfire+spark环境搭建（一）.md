#前言

本篇文章是整理以前在CSDN上所发表的文章，更新文章内容，更详细地描述操作，以方便大家阅读和理解。


#下载Openfire+Spark

首先到官网下载openfire+spark：

下载地址：[http://www.igniterealtime.org/downloads/index.jsp](http://www.igniterealtime.org/downloads/index.jsp)

如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/20150323092656005.png)

如果您的电脑是MAC电脑，要注意选择MAC版下载dmg文件；如果是windows就下载windows版本。双击运行dmg文件，安装完成后，到finder->系统偏好设置->openfire->开启，默认是开启的，然后点击进入管理页面，首先进入需要配置，如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/20150323093242966.png)


#配置服务器：

如下图所示，选择简体中文：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/05112100-bf50028050b04bc29723b869732b15d7.png)


###服务器设置

配置域的时候，使用本机127.0.0.1，如果使用localhost，我这里出现用spark时，无法创建服务器,但是ping localhost与127.0.0.1是一样的，所以这才还是使用127.0.0.1吧！如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/05112442-e66c30fe8aae45cab131a376352589b7.png)


###特性设置

如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/05134329-46acad61845c487e84bcd2d401cfc4ec.png)

###数据库设置

这里不使用标准数据库，使用自带的就可以了。

###安装完成

然后下一步，输入邮件，然后是密码和新密码，也就是进入管理员界面的密码，出现下面这个界面，就是说明配置成功了：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/05150730-d9571894a99c49f0ac988658399f15eb.png)

点击登录到管理控制台，输入admin账号和刚才设置的密码。

如果使用admin和刚才输入的密码无法登录，很有可能是没有权限访问，那么
从finder前往：/usr/local，找到openfire文件夹，如果出现红色-号，表示没有权限访问，
在显示简介，修改权限，就可以了。

###修改服务器

如果要修改服务器，在登录到管理控制台后，主页左下角有个编辑属性，就可以修改服务器，修改后需要重启openfire.


#安装Spark

安装spark非常简单，双击一路安装就可以了。然后就要注册一个账号了，如下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/20150323094532562.png)

点击“（A）账号“去注册一个号，服务器输入127.0.0.1（上面设置了这个IP作为服务器的地址），
创建成功后就可以登录了。然后到openfire后台用户组一看，是不是多了一个账号。

#最后

本篇教程到此也就完成了，之所以要写下这篇教程，就是因为这也折腾不少我的时间，安装虽然简单，但是登录总是不能成功，百度N久都没有找到办法，最后自己才查出原因。

写下本篇文章，以帮助大家跳出那个坑坑~~~


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

