#前言

开发的时候，想要运行经常需要配置HOST才能访问，那么对于iOS不越狱并不能配置HOST，如何是好？

手机配置不了HOST，我们可以通过电脑配置HOST，然后设置手机设置代理为电脑的网络IP，这样就可以访问了~

#工具

想要通过代理访问，最好用的工具就是Charles了。这里不介绍如何使用Charles，只是介绍如何配置代理来访问。

笔者这里收集了Charles 3.11.2版本及其破解jar，请到GITHUB下载：[https://github.com/CoderJackyHuang/Charles_and_key](https://github.com/CoderJackyHuang/Charles_and_key)

将对应的jar包放到如下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/屏幕快照-2016-01-27-上午10.32.38.png)

注意，破解key里面有几大平台的，如果是mac系统，就选择mac文件夹里面的jar包，其它同理。

#开启Charles

安装好Charles了以后，打开它，设置一下port，默认是8888，通常使用默认即可。

打开网络偏好设置，看到自己的IP地址了吧：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/屏幕快照-2016-01-27-上午10.36.59.png)

#手机配置代理

iPhone手机->设置->选择某个wifi->进入详细界面，选择手动->配置上面看到的ip，端口号为Charles所设置的port，默认为8888设置一下就可以了。

#电脑配置HOST

电脑要配置一下HOST：

```
// 输入密码，得到操作权限
sudo -s 

// 在这个文件里添加对应的HOST配置
vi /etc/hosts
```

#最后

接下来所有的接口请求都会通过Charles，我们都能够看到所有的接口数据哦！

![image](http://www.henishuo.com/wp-content/uploads/2016/01/charles2016.png)

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383（注明年限，新手勿扰）]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文**：[http://www.henishuo.com/ios-charles/](http://www.henishuo.com/ios-charles/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)



