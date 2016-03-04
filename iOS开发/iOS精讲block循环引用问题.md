#前言

本篇文章精讲iOS开发中使用Block时一定要注意内存管理问题，很容易造成循环引用。本篇文章的目标是帮助大家快速掌握使用block的技巧。

我相信大家都觉得使用block给开发带来了多大的便利，但是有很多开发者对block内存管理掌握得不够好，导致经常出现循环引用的问题。对于新手来说，出现循环引用时，是很难去查找的，因此通过Leaks不一定能检测出来，更重要的还是要靠自己的分析来推断出来。

#声景一：Controller之间block传值

现在，我们声明两个控制器类，一个叫ViewController，另一个叫HYBAController。其中，ViewController有一个按钮，点击时会push到HYBAController下。

先看HYBAController：

```
// 公开了一个方法
- (instancetype)initWithCallback:(HYBCallbackBlock)callback;

// 非公开的属性，这里放出来只是告诉大家，HYBAController会对这个属性强引用
@property (nonatomic, copy) HYBCallbackBlock callbackBlock;
```

下面分几种小场景来看看循环引用问题：

```
@interface ViewController ()

// 引用按钮只是为了测试
@property (nonatomic, strong) UIButton *button;
// 只是为了测试内存问题，引用之。在开发中，有很多时候我们是
// 需要引用另一个控制器的，因此这里模拟之。
@property (nonatomic, strong) HYBAController *vc;

@end

// 点击button时
- (void)goToNext {
  HYBAController *vc = [[HYBAController alloc] initWithCallback:^{
    [self.button setTitleColor:[UIColor greenColor] forState:UIControlStateNormal];
  }];
  self.vc = vc;
  [self.navigationController pushViewController:vc animated:YES];
}
```

现在看ViewController这里，这里在block的地方形成了循环引用，因此vc属性得不到释放。分析其形成循环引用的原因（如下图）：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-18-下午9.07.32.png)

可以简单说，这里形成了两个环：

* ViewController->强引用了属性vc->强引用了callback->强引用了ViewController
* ViewController->强引用了属性vc->强引用了callback->强引用了ViewController的属性button

对于此这问题，我们要解决内存循环引用问题，可以这么解：

不声明vc属性或者将vc属性声明为weak引用的类型，在callback回调处，将self.button改成weakSelf.button，也就是让callback这个block对viewcontroller改成弱引用。如就是改成如下，内存就可以正常释放了：

```
- (void)goToNext {
  __weak __typeof(self) weakSelf = self;
  HYBAController *vc = [[HYBAController alloc] initWithCallback:^{
    [weakSelf.button setTitleColor:[UIColor greenColor] forState:UIControlStateNormal];
  }];
//  self.vc = vc;
  [self.navigationController pushViewController:vc animated:YES];
}
```

笔者尝试过使用Leaks检测内存泄露，但是全是通过，一个绿色的勾，让你以为内存处理得很好了，实际上内存并得不到释放。

针对这种场景，给大家提点建议：

* 在控制器的生命周期viewDidAppear里打印日志：

```
- (void)viewDidAppear:(BOOL)animated {
  [super viewDidAppear:animated];
  
  NSLog(@"进入控制器：%@", [[self class] description]);
}
```

* 在控制器的生命周期dealloc里打印日志：

```
- (void)dealloc {
  NSLog(@"控制器被dealloc: %@", [[self class] description]);
}
```

这样的话，只要日志没有打印出来，说明内存得不到释放，就需要学会分析内存引用问题了。

#场景二：Controller与View之间Block传值

我们先定义一个view，用于与Controller交互。当点击view的按钮时，就会通过block回调给controller，也就反馈到控制器了，并将对应的数据传给控制器以记录：

```
typedef void(^HYBFeedbackBlock)(id model);

@interface HYBAView : UIView

- (instancetype)initWithBlock:(HYBFeedbackBlock)block;

@end

@interface HYBAView ()

@property (nonatomic, copy) HYBFeedbackBlock block;

@end

@implementation HYBAView

- (void)dealloc {
  NSLog(@"dealloc: %@", [[self class] description]);
}

- (instancetype)initWithBlock:(HYBFeedbackBlock)block {
  if (self = [super init]) {
    self.block = block;
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button setTitle:@"反馈给controller" forState:UIControlStateNormal];
    button.frame = CGRectMake(50, 200, 200, 45);
    button.backgroundColor = [UIColor redColor];
    [button setTitleColor:[UIColor yellowColor] forState:UIControlStateNormal];
    [button addTarget:self action:@selector(feedback) forControlEvents:UIControlEventTouchUpInside];
    [self addSubview:button];
  }
  
  return self;
}

- (void)feedback {
  if (self.block) {
    // 传模型回去，这里没有数据，假设传nil
    self.block(nil);
  }
}

@end
```

接下来看HYBAController，增加了两个属性，在viewDidLoad时，创建了aView属性：

```
@interface HYBAController()

@property (nonatomic, copy) HYBCallbackBlock callbackBlock;

@property (nonatomic, strong) HYBAView *aView;
@property (nonatomic, strong) id currentModel;

@end

@implementation HYBAController

- (instancetype)initWithCallback:(HYBCallbackBlock)callback {
  if (self = [super init]) {
    self.callbackBlock = callback;
  }
  
  return self;
}

- (void)viewDidLoad {
  [super viewDidLoad];
  
  self.title = @"HYBAController";
  self.view.backgroundColor = [UIColor whiteColor];
  
  self.aView = [[HYBAView alloc] initWithBlock:^(id model) {
    // 假设要更新model
    self.currentModel = model;
  }];
  // 假设占满全屏
  self.aView.frame = self.view.bounds;
  [self.view addSubview:self.aView];
  self.aView.backgroundColor = [UIColor whiteColor];
}

- (void)viewDidAppear:(BOOL)animated {
  [super viewDidAppear:animated];
  
  NSLog(@"进入控制器：%@", [[self class] description]);
}

- (void)dealloc {
  NSLog(@"控制器被dealloc: %@", [[self class] description]);
}

@end
```

关于上一场景所讲的循环引用已经解决了，因此我们这一小节的重点就放在controller与view的引用问题上就可以了。

**分析：**如下图所示：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-18-下午9.48.41.png)

**所形成的环有：**

* vc->aView->block->vc（self）
* vc->aView->block->vc.currentModel

**解决的办法可以是：**在创建aView时，block内对currentModel的引用改成弱引用：

```
__weak __typeof(self) weakSelf = self;
self.aView = [[HYBAView alloc] initWithBlock:^(id model) {
	// 假设要更新model
	weakSelf.currentModel = model;
}];
```

我见过很多类似这样的代码，直接使用成员变量，而不是属性，然后他们以为这样就不会引用self，也就是控制器，从而不形成环：

```
self.aView = [[HYBAView alloc] initWithBlock:^(id model) {
	// 假设要更新model
	_currentModel = model;
}];
```

这是错误的理解，当我们引用了\_currentModel时，它是控制器的成员变量，因此也就引用了控制器。要解决此问题，也是要改成弱引用：

```
__block __weak __typeof(_currentModel) weakModel = _currentModel;
self.aView = [[HYBAView alloc] initWithBlock:^(id model) {
  // 假设要更新model
  weakModel = model;
}];
```

这里还要加上\_\_block哦！

#模拟循环引用

假设下面如此写代码，是否出现内存得不到释放问题？（其中，controller属性都是强引用声明的）

```
@autoreleasepool {
  A *aVC = [[A alloc] init];
  B *bVC = [[B allcok] init];
  aVC.controller = bVC;
  bVC.controller = aVC;
}
```

**分析：**

* aVC->强引用了bVC
* bVC->强引用了aVC

如果是这样引用，就形成环了。aVC->bVC->aVC，这就形成了环。

#看看是否形成环

```
NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"a",@"b",@"abc",nil];
HYBAController *vc = [[HYBAController alloc] initWithCallback:^{
	[mArray removeObjectAtIndex:0];
}];
[self.navigationController pushViewController:vc animated:YES];
```

* vc ->强引用了block
* block->强引用了myArray

因此，并没有形成环。

如果上面将myArray给当前控制器强引用，也不会形成环：

* vc ->强引用了block
* block->强引用了self（当前控制器）

如果将vc设置成self.vc属性且是强引用，才形成了循环引用。

 

#写在最后

本篇文章就讲这么多吧，写本篇文章的目的是教大家如何分析内存是否形成环，只要懂得了如何去分析内存是否循环引用了，那么在开发时一定会特别注意内存管理问题，而且查找内存相关的问题的bug时，也比较轻松。

#源代码

本篇写了个小demo来测试的，如果只看文章不是很明白的话，如果下载demo来运行看一看，可以帮助您加深对block内存引用问题的理解。

下载地址：[标哥的GITHUB下载地址](https://github.com/CoderJackyHuang/iOSBlockUseDemo)

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


版权声明：本篇文章由[标哥的技术博客](http://www.henishuo.com/)原创出品，转载请注明出处。阅读原文：http://www.henishuo.com/ios-block-memory-cycle/