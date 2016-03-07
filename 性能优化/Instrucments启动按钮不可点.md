#前言

本来想要体验体验怎么通过Instrument中的Core Animation来看看FPS的，可是今天突然发现我的手机选择了但是那个启动按钮被置为不可点击的状态。

这里将所遇到的问题记录下来，也许下一个遇到同样的问题的人，就是你！


#尝试的过程

真是讨厌死了，我的手机怎么会不能使用呢？难道是我的手机版本太低？于是我尝试将手机升级到了9.2（之前是9.0.1），结果还是一样。

当我想要切换为我的测试手机时，弹出了一个窗口提示：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/ciAwX.png)

这是什么鬼？遇到这个问题，当然第一反映是点击**Open Xcode**了，点击了之后没有反映。起初还以为是要等待一会，结果等了好久还是没有反映。然后再试一次，结果还是一样。

于是，我尝试将连接拔掉，然后手机关机、重启，同时将Xcode关掉，然后重新打开Xcode，然后重新将手机与电脑连接，再打开Instrument，点击Core Animation，切换到当前的手机，发现可以正常点击了。

#解决方法

若遇到与笔者同样的问题，请尝试手机重启，Xcode重启，手机重新连接电脑。

后来，我在StackOverFlow上搜索到解决方案，如下：

* Unplug the device from your Mac & power down the device completely (hold the power button for several seconds; slide to power off).
* Close Xcode and Instruments.
* Restart the device & once it has booted completely re-connect it to your Mac.
* Re-launch Xcode. Here, my device showed as disabled and Xcode indicated that the device was not available for use.
* Open your project; clean (Shift+Command+K), Build (Command+B), Profile (Command+I).
* After Instruments launched I noticed that the device was enabled. Upon selecting it, a message was displayed with the title "Enable this device for development?" and message "This will open Xcode and enable this device for development." (Note that this only happened to me the first time I went through this process even though I had already been using the device for development - whereas some users have also reported that they are not presented with this dialogue.)
Enable this device for development?
* Click "Open Xcode". Here Xcode did not prompt me for anything nor was anything displayed - no additional messages indicating anything had been done or that the device was or was not available for development. Opening the Devices window, the device appeared to be available. (I have not been presented with this option for subsequent occurrences.)
* Now I was able to select the device in Instruments and profile it.

其实上面所描述的与笔者的尝试方法是一样的。所以，大家在遇到问题的时候，要先尝试去解决。

#参考

* [Unable to profile app on device with iOS 9.0.1](http://stackoverflow.com/questions/32878283/unable-to-profile-app-on-device-with-ios-9-0-1-using-xcode-7-7-0-1-or-7-1-beta/33035618#33035618)
