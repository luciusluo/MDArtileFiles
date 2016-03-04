#前言

RemoteControl，这里就翻译为远程控制吧。

远程控制是为用户提供操作App多媒体的。远程控制事件源于外部附件或由系统显示的传输控制，并通过媒体播放器框架的类传送到应用程序。播放音频或视频内容的应用程序使用这些事件来开始和停止播放，更改曲目，甚至速度的项目。所有的媒体应用程序应该支持这些事件。

除了支持远程控制事件，应用程序可以使用媒体播放器框架，以提供播放信息的曲目。该系统在适当的地方显示播放信息，如锁屏和控制中心。有关媒体播放器框架类的更多信息，见媒体播放器框架参考[Media Player Framework Reference](https://developer.apple.com/library/ios/documentation/MediaPlayer/Reference/MediaPlayer_Framework/).


#Remote Control功能

RemoteControl可以用来在不打开app的情况下控制app中的多媒体播放，主要包括：

1. 锁屏界面双击Home键后出现的播放操作区域
2. iOS7之后控制中心的播放操作区域
3. iOS7之前双击home键后出现的进程中向左滑动出现的播放操作区域
4. AppleTV，AirPlay中显示的播放操作区域
5. 耳机线控
6. 车载系统的设置


#让App支持Remote Control

要接收远程控制事件，需要做到以下的几点：

1. 接收者必须可以成为第一响应者；
2. 接收者必须显示声明接收RemoteControl事件；
3. App必须是Now Playing App.

若应用程序还提供了正在播放的信息，使用`MPNowPlayingInfoCenter`对象在适当的时候更新信息。

#效果图

类似网易云音乐锁屏状态下的Now Playing显示信息：

![image](http://www.henishuo.com/wp-content/uploads/2016/01/IMG_0742.png)

#环境准备

要使App支持Remote Control，需要设置一下info.plist文件，添加required background modes，并添加一个值为：

```
App plays audio or streams audio/video using AirPlay
```

接下来，播放音频之前先要设置AVAudioSession模式：

```
AVAudioSession *session = [AVAudioSession sharedInstance];
[session setCategory:AVAudioSessionCategoryPlayAndRecord
       withOptions:AVAudioSessionCategoryOptionDefaultToSpeaker
             error:nil];
```

#扩展UIApplication作为第一响应者

![image](http://www.henishuo.com/wp-content/uploads/2016/01/49769148_1.jpg)

在iOS上，响应者链如上图，由图可知，若其它视图没有处理，最终会传到UIApplication，因此使用UIApplication作为第一响应者是最合适的。

扩展头文件声明，我们在监听到对应的事件时，发送通知。

```
//
//  UIApplication+RemoteControl.h
//  IOSAudioRemoteControl
//
//  Created by huangyibiao on 15/3/25.
//  Copyright (c) 2015年 huangyibiao. All rights reserved.
//

#import <UIKit/UIKit.h>

// 播放
extern NSString *kRemoteControlPlayTapped;
// 暂停
extern NSString *kRemoteControlPauseTapped;
// 停止
extern NSString *kRemoteControlStopTapped;
// 前一首
extern NSString *kRemoteControlPreviousTapped;
// 后一首
extern NSString *kRemoteControlNextTapped;
// 其它
extern NSString *kRemoteControlOtherTapped;

@interface UIApplication (RemoteControl)

/**
 *  注册对remote control事件的监听
 *
 *  @param observer 监听者
 *  @param selector 回调
 */
- (void)observeRemoteControl:(id)observer selector:(SEL)selector;

@end
```

在实现文件中，我们需要处理监听：

```
//
//  UIApplication+RemoteControl.m
//  IOSAudioRemoteControl
//
//  Created by huangyibiao on 15/3/25.
//  Copyright (c) 2015年 huangyibiao. All rights reserved.
//

#import "UIApplication+RemoteControl.h"

const NSString *kRemoteControlPlayTapped = @"kRemoteControlPlayTapped";
const NSString *kRemoteControlPauseTapped = @"kRemoteControlPauseTapped";
const NSString *kRemoteControlStopTapped = @"kRemoteControlStopTapped";
const NSString *kRemoteControlPreviousTapped = @"kRemoteControlForwardTapped";
const NSString *kRemoteControlNextTapped = @"kRemoteControlBackwardTapped";
const NSString *kRemoteControlOtherTapped = @"kRemoteControlOtherTapped";

@implementation UIApplication (RemoteControl)

- (BOOL)canBecomeFirstResponder {
  return YES;
}

- (void)remoteControlReceivedWithEvent:(UIEvent *)event {
  switch (event.subtype) {
    case UIEventSubtypeRemoteControlPlay:
      [self postNotification:kRemoteControlPlayTapped];
      break;
      case UIEventSubtypeRemoteControlPause:
      [self postNotification:kRemoteControlPauseTapped];
      break;
    case UIEventSubtypeRemoteControlStop:
      [self postNotification:kRemoteControlStopTapped];
      break;
      case UIEventSubtypeRemoteControlNextTrack:
      [self postNotification:kRemoteControlNextTapped];
      break;
      case UIEventSubtypeRemoteControlPreviousTrack:
      [self postNotification:kRemoteControlPreviousTapped];
      break;
    default:
      [self postNotification:kRemoteControlOtherTapped];
      break;
  }
}

- (void)postNotification:(const NSString *)notificationName {
  [[NSNotificationCenter defaultCenter]
   postNotificationName:(NSString *)notificationName object:nil];
}

- (void)observeRemoteControl:(id)observer selector:(SEL)selector {
  NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
  
  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlNextTapped object:nil];
  
  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPauseTapped object:nil];

  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlStopTapped object:nil];
  
  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPreviousTapped object:nil];
  
  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlPlayTapped object:nil];
  
  [center addObserver:observer selector:selector name:(NSString *)kRemoteControlOtherTapped object:nil];
}

@end
```

#在播放audio视图中监听通知

在播放音频的界面，可以是在UIViewController里，也只可以是某个UIView里，因此在对应的类在创建或者初始化时调用监听：

```
UIApplication *app = [UIApplication sharedApplication];
[app observeRemoteControl:self
                 selector:@selector(onRemoteControleStateChanged:)];
```

然后处理一下事件回调：

```
#pragma mark - remote control
- (void)onRemoteControleStateChanged:(NSNotification *)notification {
  if ([notification.name isEqualToString:kRemoteControlPlayTapped]) {
     [self play];
  } else if ([notification.name isEqualToString:kRemoteControlPauseTapped]) {
     [self pause];
  } else if ([notification.name isEqualToString:kRemoteControlNextTapped]) {
    [self playNext];
  } else if ([notification.name isEqualToString:kRemoteControlPreviousTapped]) {
   
  } else if ([notification.name isEqualToString:kRemoteControlStopTapped]) {
    [self _requestPauseAudioPlaying:nil];
  } else if ([notification.name isEqualToString:kRemoteControlOtherTapped]) {
    
  }
}
```

#配置NowPlaying显示数据

要想在当前播放中心中显示当前播放的音频的信息，需要通过`MPNowPlayingInfoCenter`类，它是一个单例。我们要显示什么就在它的`nowPlayingInfo`属性中设置相关配置数据。

要使用这个类及其相关属性定义，需要引入这两个头文件：

```
#import <MediaPlayer/MPNowPlayingInfoCenter.h>
#import <MediaPlayer/MPMediaItem.h>
```

这里只设置几项，更多设置，请到MPMediaItem.h头文件中查看：

```
NSMutableDictionary *dict = [[NSMutableDictionary alloc] init];
  
// 设置歌曲播放时长
if (self.time != nil && self.time.length != 0) {
	[dict setObject:@(self.time.doubleValue) forKey:MPMediaItemPropertyPlaybackDuration];
}

// 设置歌曲名称
if (self.title != nil && self.title.length != 0) {
	[dict setObject:self.title forKey:MPMediaItemPropertyTitle];
}
  
// 设置锁屏时的图片
if (self.iconView.image && [self.iconView.image isKindOfClass:[UIImage class]]) {
	MPMediaItemArtwork *artwork = [[MPMediaItemArtwork alloc] initWithImage:self.iconView.image];
	[dict setObject:artwork forKey:MPMediaItemPropertyArtwork];
}

[MPNowPlayingInfoCenter defaultCenter].nowPlayingInfo = dict;
```

到此就可以实现我们远程控制的功能。不过这里只考虑实现，并不考虑什么时候接收，什么时候不再接收，而这里的做法时什么时候都接收远程控制。这此事就留给大家去学习吧！

#源代码

大家可以到我的GITHUB下载demo：[https://github.com/CoderJackyHuang/IOSAudioRemoteControl](https://github.com/CoderJackyHuang/IOSAudioRemoteControl)

#参考

[苹果官方文档：Remote Control Events](https://developer.apple.com/library/prerelease/tvos/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Remote-ControlEvents/Remote-ControlEvents.html#//apple_ref/doc/uid/TP40009541-CH7-SW3)

#关注我


如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，阅读原文：**[http://www.henishuo.com/ios-remote-control/](http://www.henishuo.com/ios-remote-control/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)

