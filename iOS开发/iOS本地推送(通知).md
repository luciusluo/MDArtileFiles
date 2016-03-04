#前言

这个是很久之前在CSDN上发布过的文章了，现在整理放到新的个人博客上。本篇文章是讲如何操作本地推送，若使用过程中出现任何问题，可以加群提出或者在评论中提出。

#兼容iOS版本

在iOS8之后，以前的本地推送写法可能会出错，接收不到推送的信息，
如果出现以下信息：

1. Attempting to schedule a local notification <br>
2. with an alert but haven't received permission from the user to display alerts<br>
3. with a sound but haven't received permission from the user to play sounds


说明在iOS8下没有注册，所以需要额外添加对IOS8的注册方法，API中有下面这个方法：

```
// Registering UIUserNotificationSettings more than once results in previous settings being overwritten.  
- (void)registerUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings NS_AVAILABLE_IOS(8_0);  
```

这个方法是8.0之后才能使用的，所以需要判断一下系统的版本。

#本地通知三步法

* 第一步：注册本地通知
* 第二步：处理通知回调
* 第三步：取消某个推送或者全部推送

##注册本地通知

```
// 设置本地通知  
+ (void)registerLocalNotification:(NSInteger)alertTime {  
  UILocalNotification *notification = [[UILocalNotification alloc] init];  
  // 设置触发通知的时间  
  NSDate *fireDate = [NSDate dateWithTimeIntervalSinceNow:alertTime];  
  NSLog(@"fireDate=%@",fireDate);  
    
  notification.fireDate = fireDate;  
  // 时区  
  notification.timeZone = [NSTimeZone defaultTimeZone];  
  // 设置重复的间隔  
  notification.repeatInterval = kCFCalendarUnitSecond;  
    
  // 通知内容  
  notification.alertBody =  @"该起床了...";  
  notification.applicationIconBadgeNumber = 1;  
  // 通知被触发时播放的声音  
  notification.soundName = UILocalNotificationDefaultSoundName;  
  // 通知参数  
  NSDictionary *userDict = [NSDictionary dictionaryWithObject:@"开始学习iOS开发了" forKey:@"key"];  
  notification.userInfo = userDict;  
    
  // ios8后，需要添加这个注册，才能得到授权  
  if ([[UIApplication sharedApplication] respondsToSelector:@selector(registerUserNotificationSettings:)]) {  
    UIUserNotificationType type =  UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound;  
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:type  
                                                                             categories:nil];  
    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];  
    // 通知重复提示的单位，可以是天、周、月  
    notification.repeatInterval = NSCalendarUnitDay;  
  } else {  
    // 通知重复提示的单位，可以是天、周、月  
    notification.repeatInterval = NSDayCalendarUnit;  
  }  
    
  // 执行通知注册  
  [[UIApplication sharedApplication] scheduleLocalNotification:notification];  
}  
```

##处理通知回调

处理通知回调，这个是在appdelegate中的代理方法回调：

```
// 本地通知回调函数，当应用程序在前台时调用  
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {  
  NSLog(@"noti:%@",notification);  
    
  // 这里真实需要处理交互的地方  
  // 获取通知所带的数据  
  NSString *notMess = [notification.userInfo objectForKey:@"key"];  
  UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"本地通知(前台)"  
                                                  message:notMess  
                                                 delegate:nil  
                                        cancelButtonTitle:@"OK"  
                                        otherButtonTitles:nil];  
  [alert show];  
    
  // 更新显示的徽章个数  
  NSInteger badge = [UIApplication sharedApplication].applicationIconBadgeNumber;  
  badge--;  
  badge = badge >= 0 ? badge : 0;  
  [UIApplication sharedApplication].applicationIconBadgeNumber = badge;  
    
  // 在不需要再推送时，可以取消推送  
  [HomeViewController cancelLocalNotificationWithKey:@"key"];  
}  
```

#取消某个推送

```
// 取消某个本地推送通知  
+ (void)cancelLocalNotificationWithKey:(NSString *)key {  
  // 获取所有本地通知数组  
  NSArray *localNotifications = [UIApplication sharedApplication].scheduledLocalNotifications;  
    
  for (UILocalNotification *notification in localNotifications) {  
    NSDictionary *userInfo = notification.userInfo;  
    if (userInfo) {  
      // 根据设置通知参数时指定的key来获取通知参数  
      NSString *info = userInfo[key];  
        
      // 如果找到需要取消的通知，则取消  
      if (info != nil) {  
        [[UIApplication sharedApplication] cancelLocalNotification:notification];  
        break;  
      }  
    }  
  }  
} 
```

#源代码

大家可以到我的GITHUB下载Demo，地址为：[https://github.com/CoderJackyHuang/LocalPush](https://github.com/CoderJackyHuang/LocalPush)

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)


#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
