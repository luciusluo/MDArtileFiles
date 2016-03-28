#前言

Xcode真机无发布证书打包IPA包，怎么做呢？今天遇到了这么个小问题。不过所打出来的包要求越狱手机才能安装哦！

#解决办法

**第一步：**如下图，修改成release模式

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-0@2x.png)

**第二步：**如下图，选择设备来编译

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-1@2x.png)

**第三步：**如下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-2@2x.png)

然后进入到app：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-3@2x-e1458635530497.png)

进去就可以看到.app文件了。

**第四步：**拖动到itunes，如下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-4@2x-e1458635775876.png)

选择它，然后右键:show in finder：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160322-5@2x-e1458635879420.png)

#完结

所生成的IPA包只能是越狱手机才能安装哦！

