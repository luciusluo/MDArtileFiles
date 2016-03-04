#前言


一直以来都让我很头痛的一个问题：系统自带的导航条，在标题文字很长时，进入到下一个界面，而下一个界面的标题也很长时，就会出现标题不居中显示。

曾经，我尝试过很多种办法，但是都没有从根上解决问题。下面笔者分别说说用过哪些方案。

#方案一（不可行）


这个方案是不使用系统自带默认的`backButtonItem`，而是使用`leftBarButtonItem`。

这样做的**好处**是：解决了本界面标题过长，而上一个界面的标题也很长时，本界面的标题不居中显示的问题。

这样做的**坏处**是：系统自带的右滑返回手势就没有了。

#方案二（不可行）


这里我们自定义一个继承于`UINavigationController`的类，然后所有使用导航类的地方都使用我们所定义的导航控制器类。在这个类中，我们通过配置全局的导航条相关全局属性。

```
- (void)config {
  NSString *backImageName = @"default_back";
  
  if (kIsIOS7OrLater) {
    UIImage *image = [UIImage imageNamed:backImageName];
    CGFloat w = image.size.width;
    if (kScreenWidth <= 320) {
      w = 30;
    }
    self.navigationBar.backIndicatorImage = image;
    UIImage *backButtonImage = [image resizableImageWithCapInsets:UIEdgeInsetsMake(0, w, 0, -w)];
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:backButtonImage
                                                      forState:UIControlStateNormal
                                                    barMetrics:UIBarMetricsDefault];
    // 将返回按钮的文字position设置不在屏幕上显示
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(NSIntegerMin, NSIntegerMin) forBarMetrics:UIBarMetricsDefault];
    self.interactivePopGestureRecognizer.enabled = YES;
  } else {
#if __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_7_0
    UIBarButtonItem *item = [[UIBarButtonItem alloc] init];
    UIImage *image = [UIImage imageNamed:backImageName];
    [item setBackButtonBackgroundImage:[image resizableImageWithCapInsets:UIEdgeInsetsMake(0, image.size.width, 0, 0)] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [item setBackgroundImage:image forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [item setBackButtonTitlePositionAdjustment:UIOffsetMake(NSIntegerMin, NSIntegerMin) forBarMetrics:UIBarMetricsDefault];
    self.navigationItem.backBarButtonItem = item;
#endif
  }
  
  return;
}
```

因此这里，我们需要使用自定义的返回箭头，而且不显示返回按钮的文字。因此，我们需要设置如下：

```
[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(NSIntegerMin, NSIntegerMin) forBarMetrics:UIBarMetricsDefault];
```

但是，仅仅部分标题是正常显示，还是有一些地方有不居中的。因此，我们还需要再添加一些代码解决。

下面，我们使用了全局设置`UIBarButtonItem`的字体大小为`1`：

```
NSDictionary *attributes = @{NSFontAttributeName : [UIFont systemFontOfSize:1]};
[[UIBarButtonItem appearance] setTitleTextAttributes:attributes
                                            forState:UIControlStateNormal];
```

这样确实解决了我们的问题。标题没有出现不居中显示的了，但是又引发了新的问题。因为我们是全局将`UIBarButtonItem`的字体大小修改为1，那么所有使用了这个控件的地方，都会不显示了。除非我们每个使用的地方再重新设置其字体大小。

因此，此方案也不可行。

#方案三（可行，但不太好）


应用中始终隐藏系统自带的导航，然后自定义一个`UIView`作为导航，这样就解决居中显示的问题，但是没有了返回手势，因此需要借助第三方库追加右滑返回手势功能，但是体验不如原生的好。

#方案四（当前最终方案）



自定义一个继承于`UINavigationController`的类，然后所有使用导航类的地方都使用我们所定义的导航控制器类。在这个类中，我们通过配置全局的导航条相关全局属性。

```
- (void)config {
  NSString *backImageName = @"default_back";
  
  if (kIsIOS7OrLater) {
    UIImage *image = [UIImage imageNamed:backImageName];
    CGFloat w = image.size.width;
    if (kScreenWidth <= 320) {
      w = 30;
    }
    self.navigationBar.backIndicatorImage = image;
    UIImage *backButtonImage = [image resizableImageWithCapInsets:UIEdgeInsetsMake(0, w, 0, -w)];
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:backButtonImage
                                                      forState:UIControlStateNormal
                                                    barMetrics:UIBarMetricsDefault];
    // 将返回按钮的文字position设置不在屏幕上显示
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(NSIntegerMin, NSIntegerMin) forBarMetrics:UIBarMetricsDefault];
    self.interactivePopGestureRecognizer.enabled = YES;
  } else {
#if __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_7_0
    UIBarButtonItem *item = [[UIBarButtonItem alloc] init];
    UIImage *image = [UIImage imageNamed:backImageName];
    [item setBackButtonBackgroundImage:[image resizableImageWithCapInsets:UIEdgeInsetsMake(0, image.size.width, 0, 0)] forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [item setBackgroundImage:image forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [item setBackButtonTitlePositionAdjustment:UIOffsetMake(NSIntegerMin, NSIntegerMin) forBarMetrics:UIBarMetricsDefault];
    self.navigationItem.backBarButtonItem = item;
#endif
  }
  
  return;
}
```

然后，我们在每个控制器的`viewDidLoad`方法中，调用这么个方法：

```
- (void)resetBackButtonItem {
  NSArray *viewControllerArray = [self.navigationController viewControllers];
  
  long previousViewControllerIndex = [viewControllerArray indexOfObject:self] - 1;
  UIViewController *previous;
  
  if (previousViewControllerIndex >= 0) {
    previous = [viewControllerArray objectAtIndex:previousViewControllerIndex];
    previous.navigationItem.backBarButtonItem = [[UIBarButtonItem alloc]
                                                 initWithTitle:@""
                                                 style:UIBarButtonItemStylePlain
                                                 target:self
                                                 action:nil];
  }
}
```

这个方法是我们用于判断是否有上一个界面，如果有，则将上一个界面的返回按钮的标题设置为空，那么在本界面的返回按钮就不会有标题，如此一来，就解决了上个界面的标题过长，而本界面标题也很长时，导致本界面的标题不居中显示的问题。

#建议

我们可以将这`resetBackButtonItem`方法放到基类控制器中，然后在基类控制器中的`viewDidLoad`方法中调用，就不需要各个类都调用了。当然，如果现在类似笔者当前的情景，就需要手动各个界面都调用了。当前笔者的情景是，项目是由很多个团队敏捷迭代开发的，如果全局统一改，很有可能会影响到他人的版本，因此不得不只在需要处理的控制器调用。

