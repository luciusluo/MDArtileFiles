#前言

今天有个小需求，在点击导航条上的返回按钮之前要调用某个API，并弹出UIAlertView来显示，根据用户的选项判断是否是返回还是继续留在当前控制器。举个简单的例子，当点击导航条上的左上角返回按钮时，就调用我们的API来提示是否知道，点击知道则返回，点击不知道则继续留在当前控制器。

那么问题来了，导航自带的右滑返回手势在点击系统的返回按钮时，不会没有办法处理，那是自动的，因此就要想办法改成leftBarButtonItem了，但是使用了leftBarButtonItem就没有了右滑返回手势。

鱼和熊掌不可兼得？笔者自有办法！

笔者尝试写个demo来验证有什么办法可以解决，尝试了以下四种：

* 只在当前controller遵守UIGestureRecognizerDelegate并设置代理为self
* 将UIGestureRecognizerDelegate放在公共基类控制器遵守并设置代理为self，然后子类重写代理方法
* 将UIGestureRecognizerDelegate放在公共导航类HYBNavigationController里遵守，并设置代理为导航类，然后重写push/pop相关的所有方法
* 将UIGestureRecognizerDelegate放在公共导航类HYBNavigationController里遵守，并设置代理为导航类，但是，只遵守-gestureRecognizerShouldBegin:代理方法

#方案一（不可行）

方案一：只在当前controller遵守UIGestureRecognizerDelegate并设置代理为self

为什么不可行呢？当想不测试怎么知道呢？光想是很难考虑全面的。于是写了个小demo来测试。

我们在该controller里这样写：

```
- (void)viewDidLoad {
  [super viewDidLoad];
  
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
  [button setTitle:@"返回" forState:UIControlStateNormal];
  [button addTarget:self action:@selector(onBack) forControlEvents:UIControlEventTouchUpInside];
  [button sizeToFit];
  [button setTitleColor:[UIColor blueColor] forState:UIControlStateNormal];
  UIBarButtonItem *btnItem = [[UIBarButtonItem alloc] initWithCustomView:button];
  self.navigationItem.leftBarButtonItem = btnItem;
  
  // 关键行
  self.navigationController.interactivePopGestureRecognizer.delegate = self;
}
```

一旦设置了代理为self，那么使用leftBarButtonItem后就可以实现点击回调，而且右滑手势还在。

但是，self.navigationController那可是导航控制器对象的的代理被修改当某个控制器对象了，当这个控制器类被释放后，那么代理就为nil了，如此就再也没有右滑返回手势了。

那么可能有人会想，在-viewDidAppear:里设置代理为self，在-viewDidDisappear:时设置代理成原来的代理对象呢？同样不可以。当A push到B，B push到C，然后从C返回后，代理就不再是最初的导航代理了。

所以，该方案不可行。

#方案二（不可行）

方案二：将UIGestureRecognizerDelegate放在公共基类控制器遵守并设置代理为self，然后子类重写代理方法

笔者尝试将UIGestureRecognizerDelegate放在HYBBaseViewControlle里遵守，然后实现代理，默认返回YES，表示支持右滑返回。如果要让某个控制器不支持右滑返回或者在返回前先执行什么操作，可以通过重写此代理方法来实现。

当只在一个控制器里时，这是可以实现的。但是，当这个控制器被释放了以后，代理对象就变成了nil了，因此代理是对于导航条对象的，不属性单个控制器的。

#方案三（可行，但复杂）

方案三：将UIGestureRecognizerDelegate放在公共导航类HYBNavigationController里遵守，并设置代理为导航类，然后重写push/pop相关的所有方法。

如实现如何下：

```
//
//  HYBNavigationController.m
//  NavRightPanGestureDemo
//
//  Created by huangyibiao on 16/2/22.
//  Copyright © 2016年 huangyibiao. All rights reserved.
//

#import "HYBNavigationController.h"
#import "HYBBaseViewController.h"

@interface HYBNavigationController () <UIGestureRecognizerDelegate>

@property (nonatomic, assign) BOOL enableRightGesture;

@end

@implementation HYBNavigationController


- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.enableRightGesture = YES;
  self.interactivePopGestureRecognizer.delegate = self;
}

- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer { 
  return self.enableRightGesture;
}

- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated {
  if ([viewController isKindOfClass:[HYBBaseViewController class]]) {
    if ([viewController respondsToSelector:@selector(gestureRecognizerShouldBegin)]) {
      HYBBaseViewController *vc = (HYBBaseViewController *)viewController;
      self.enableRightGesture = [vc gestureRecognizerShouldBegin];
    }
  }
  
  [super pushViewController:viewController animated:YES];
}

- (NSArray<UIViewController *> *)popToRootViewControllerAnimated:(BOOL)animated {
     self.enableRightGesture = YES; 
  return [super popToRootViewControllerAnimated:animated];
}

- (UIViewController *)popViewControllerAnimated:(BOOL)animated {
  if (self.viewControllers.count == 1) {
    self.enableRightGesture = YES;
  } else {
    NSUInteger index = self.viewControllers.count - 2;
    UIViewController *destinationController = [self.viewControllers objectAtIndex:index];
    if ([destinationController isKindOfClass:[HYBBaseViewController class]]) {
      if ([destinationController respondsToSelector:@selector(gestureRecognizerShouldBegin)]) {
        HYBBaseViewController *vc = (HYBBaseViewController *)destinationController;
        self.enableRightGesture = [vc gestureRecognizerShouldBegin];
      }
    }
  }
  
  return [super popViewControllerAnimated:animated];
}

- (NSArray<UIViewController *> *)popToViewController:(UIViewController *)viewController animated:(BOOL)animated {
  if (self.viewControllers.count == 1) {
    self.enableRightGesture = YES;
  } else {
    UIViewController *destinationController = viewController;
    if ([destinationController isKindOfClass:[HYBBaseViewController class]]) {
      if ([destinationController respondsToSelector:@selector(gestureRecognizerShouldBegin)]) {
        HYBBaseViewController *vc = (HYBBaseViewController *)destinationController;
        self.enableRightGesture = [vc gestureRecognizerShouldBegin];
      }
    }
  }
  
  return [super popToViewController:viewController animated:animated];
}

@end
```

这是通过重写所有的pop/push相关方法，通过判断是否要求支持右滑来设置。然后，我们要让某个控制器类在右滑返回或者点击返回之前，先调用我们的API判断，如下：

```
#import "HYBBController.h"

@implementation HYBBController

- (void)viewDidLoad {
  [super viewDidLoad];
  
  UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
  [button setTitle:@"返回" forState:UIControlStateNormal];
  [button addTarget:self action:@selector(onBack) forControlEvents:UIControlEventTouchUpInside];
  [button sizeToFit];
  [button setTitleColor:[UIColor blueColor] forState:UIControlStateNormal];
  UIBarButtonItem *btnItem = [[UIBarButtonItem alloc] initWithCustomView:button];
  self.navigationItem.leftBarButtonItem = btnItem;
}

- (BOOL)gestureRecognizerShouldBegin {
  [self onBack];
  return NO;
}

- (void)onBack {
  UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"标哥的技术博客"
                                                      message:@"知道博客地址是什么吗？"
                                                     delegate:self
                                            cancelButtonTitle:@"不知道"
                                            otherButtonTitles:@"知道", nil];
  [alertView show];
}


#pragma mark - UIAlertViewDelegate
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
  if (buttonIndex == 0) {
    
  } else {
    if ([self.navigationItem.title isEqualToString:@"VC6"]) {
      NSUInteger index = self.navigationController.viewControllers.count - 3;
      UIViewController *vc = [self.navigationController.viewControllers objectAtIndex:index];
      [self.navigationController popToViewController:vc animated:YES];
    } else {
      [self.navigationController popViewControllerAnimated:YES];
    }
  }
}

@end
```

这种方案确实实现了我们的需求。但是，有没有更简单的方案呢？今天可能是眼睛有点困的原因，在研究的时候没有意识到第四种方案。在我准备写这篇文章的时候，我再认识地理了一遍逻辑，发现还有非常简单的一种方案可以实现我的需求。

#方案四（可靠，最优）


方案四：将UIGestureRecognizerDelegate放在公共导航类HYBNavigationController里遵守，并设置代理为导航类，但是，只遵守-gestureRecognizerShouldBegin:代理方法。

```
@interface HYBNavigationController () <UIGestureRecognizerDelegate>

@end

@implementation HYBNavigationController


- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.interactivePopGestureRecognizer.delegate = self;
}

- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer {
  BOOL ok = YES; // 默认为支持右滑反回
  if ([self.topViewController isKindOfClass:[HYBBaseViewController class]]) {
    if ([self.topViewController respondsToSelector:@selector(gestureRecognizerShouldBegin)]) {
      HYBBaseViewController *vc = (HYBBaseViewController *)self.topViewController;
     ok = [vc gestureRecognizerShouldBegin];
    }
  }
  
  return ok;
}

@end
```

使用方法与第三种方案一样，是不是非常地简化了？看来是元宵给我的礼物啊，突然想到这样的办法。以前一直没有研究过interactivePopGestureRecognizer属性，这个属性是iOS7以后才有的，因此在项目中一直不能直接使用leftBarButtonItem处理，除非那个界面不要右滑返回。

现在，一切都明了了，想要使用leftBarButtonItem在公共基类控制器中统一调用API来设置就非常简单了，右滑返回手势也可以正常使用~

还等什么，赶紧试试吧！

#最后

如果你所使用的项目也有这样的需求，不防试试吧！笔者提供了demo的，因此可以先下载demo来看看效果哦！经过多次测试，笔者认为这是可行的方案，大家若在使用中出现问题，还请反馈与笔者，我也想了解是什么情况，当然也要找解决方案，共同进步嘛。

#源代码

请大家到GITHUB下载吧：[CoderJackyHuang](https://github.com/CoderJackyHuang/NavGestureDemo)

原文出处：[标哥的技术博客](http://www.henishuo.com/nav-leftbarbuttonitem-gesture/)

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






