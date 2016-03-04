>iOS极光推送

---

稍稍研究了一下极光推送，其实是非常简单的，不过这个过程也出现了一些问题。
对于应用在前台时，需要额外处理一下。
关于极光推送，由于在`iOS8`之后，有了新的`API`，因此极光也给我们提供了适配的`API`。
下面我就把对极光推送相关API的封装提取出来，希望对大家有帮助，同时也当是总结。

#封装成工具类

---
下面是对极光推送而封装的一个工具类：

```
//
//  HYBJPushHelper.h
//  JPushDemo
//
//  Created by 黄仪标 on 14/11/20.
//  Copyright (c) 2014年 黄仪标. All rights reserved.
//

#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

/*!
 * @brief 极光推送相关API封装
 * @author huangyibiao
 */
@interface HYBJPushHelper : NSObject

// 在应用启动的时候调用
+ (void)setupWithOptions:(NSDictionary *)launchOptions;

// 在appdelegate注册设备处调用
+ (void)registerDeviceToken:(NSData *)deviceToken;

// ios7以后，才有completion，否则传nil
+ (void)handleRemoteNotification:(NSDictionary *)userInfo completion:(void (^)(UIBackgroundFetchResult))completion;

// 显示本地通知在最前面
+ (void)showLocalNotificationAtFront:(UILocalNotification *)notification;

@end

//
//  HYBJPushHelper.m
//  JPushDemo
//
//  Created by 黄仪标 on 14/11/20.
//  Copyright (c) 2014年 黄仪标. All rights reserved.
//

#import "HYBJPushHelper.h"
#import "APService.h"

@implementation HYBJPushHelper

+ (void)setupWithOptions:(NSDictionary *)launchOptions {
  // Required
#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_7_1
  // ios8之后可以自定义category
  if ([[UIDevice currentDevice].systemVersion floatValue] >= 8.0) {
    // 可以添加自定义categories
    [APService registerForRemoteNotificationTypes:(UIUserNotificationTypeBadge |
                                                   UIUserNotificationTypeSound |
                                                   UIUserNotificationTypeAlert)
                                       categories:nil];
  } else {
#if __IPHONE_OS_VERSION_MAX_ALLOWED < __IPHONE_8_0
    // ios8之前 categories 必须为nil
    [APService registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
                                                   UIRemoteNotificationTypeSound |
                                                   UIRemoteNotificationTypeAlert)
                                       categories:nil];
#endif
  }
#else
  // categories 必须为nil
  [APService registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
                                                 UIRemoteNotificationTypeSound |
                                                 UIRemoteNotificationTypeAlert)
                                     categories:nil];
#endif
  
  // Required
  [APService setupWithOption:launchOptions];
  return;
}

+ (void)registerDeviceToken:(NSData *)deviceToken {
  [APService registerDeviceToken:deviceToken];
  return;
}

+ (void)handleRemoteNotification:(NSDictionary *)userInfo completion:(void (^)(UIBackgroundFetchResult))completion {
  [APService handleRemoteNotification:userInfo];
  
  if (completion) {
    completion(UIBackgroundFetchResultNewData);
  }
  return;
}

+ (void)showLocalNotificationAtFront:(UILocalNotification *)notification {
  [APService showLocalNotificationAtFront:notification identifierKey:nil];
  return;
}

@end
```

下面就是测试一个推送功能了：

```
//
//  AppDelegate.m
//  JPushDemo
//
//  Created by 黄仪标 on 14/11/20.
//  Copyright (c) 2014年 黄仪标. All rights reserved.
//

#import "AppDelegate.h"
#import "JPushHelper/HYBJPushHelper.h"

@interface AppDelegate ()

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
  // Override point for customization after application launch.
  
  [HYBJPushHelper setupWithOptions:launchOptions];
  
  self.window.backgroundColor = [UIColor whiteColor];
  [self.window makeKeyAndVisible];
  return YES;
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [HYBJPushHelper registerDeviceToken:deviceToken];
  return;
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  [HYBJPushHelper handleRemoteNotification:userInfo completion:nil];
  return;
}

#if __IPHONE_OS_VERSION_MAX_ALLOWED >= __IPHONE_7_0
// ios7.0以后才有此功能
- (void)application:(UIApplication *)application didReceiveRemoteNotification
                   :(NSDictionary *)userInfo fetchCompletionHandler
                   :(void (^)(UIBackgroundFetchResult))completionHandler {
  [HYBJPushHelper handleRemoteNotification:userInfo completion:completionHandler];
  
  // 应用正处理前台状态下，不会收到推送消息，因此在此处需要额外处理一下
  if (application.applicationState == UIApplicationStateActive) {
   UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"收到推送消息"
                                                   message:userInfo[@"aps"][@"alert"]
                                                  delegate:nil
                                         cancelButtonTitle:@"取消"
                                         otherButtonTitles:@"确定", nil];
    [alert show];
  }
  return;
}
#endif

- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
  [HYBJPushHelper showLocalNotificationAtFront:notification];
  return;
}

- (void)application:(UIApplication *)app didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
  NSLog(@"Error in registration. Error: %@", err);
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
  [application setApplicationIconBadgeNumber:0];
  return;
}

@end
```

真机运行，然后到官网去发一个通知，就可以收到了！


#Good luck!

#源代码

---
小伙伴们可以下载工程了：[https://github.com/CoderJackyHuang/JPushDemo](https://github.com/CoderJackyHuang/JPushDemo)

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**