>#iOS中的通知(NSNotificationCenter)

---
通知中心是一个单例。通知在`iOS`中是一种设计模式。每一个应用程序都有一个通知中心`NSNotificationCenter`实例, 专门负责协助不同对象之间的消息通信.

任何一个对象都可以向通知中心发布`NSNotification`, 描述自己在做什么，而任何注册了该通知的对象该特定通知发布的时候会收到这个通知。

#获取通知中心对象

---
通过下面的方式来获取通知中心对象：

```
NSNotificationCenter *center = [NSNotificationCenter  defaultCenter];
```

#通知对象主要属性

---

一个完整通知，也就是`NSNotification`对象，有下面这三个主要的属性：

```
@property (readonly, copy) NSString *name;
@property (nullable, readonly, retain) id object;
@property (nullable, readonly, copy) NSDictionary *userInfo;
```

其中，第一个就是通知的名称，第二个是通知发布者，而第三个就是一些额外的信息。比如第三个可以存储一些数据，这样接收者就可以接收到这些数据了。

#通知对象初始化方法

---

初始化一个通知对象，有这几个方法：

```
- (instancetype)initWithName:(NSString *)name
                      object:(nullable id)object
                       userInfo:(nullable NSDictionary *)userInfo;

+ (instancetype)notificationWithName:(NSString *)aName 
                              object:(nullable id)anObject;
+ (instancetype)notificationWithName:(NSString *)aName 
                              object:(nullable id)anObject 
                            userInfo:(nullable NSDictionary *)aUserInfo;
```

苹果还提供了一个初始化方法，但是苹果要求不能使用：

```
/* do not invoke; not a valid initializer for this class */
- (instancetype)init /*NS_UNAVAILABLE*/;
```

#发布通知

---
通知的发布，需要通过通知中心`NSNotificationCenter`来发布。发布通知中几个三个方法：

```

- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSString *)aName 
                      object:(nullable id)anObject;
- (void)postNotificationName:(NSString *)aName
                      object:(nullable id)anObject
                       userInfo:(nullable NSDictionary *)aUserInfo;
```

* 第一个方法直接将一个通知对象发布出去，这种方法使用场景极少，几乎没有见到使用的
* 第二个方法是根据通知的名称，发布通知的对象，将通知发布出去。当不需要传递参数时，这种方法使用场景比较多
* 第三个方法与第二个方法就差一个参数，而第三方方法主要是可以发布通知的同时传递一些额外的信息

#注册通知

---
要想能够接收到通知，就得提前注册通知到通知中心，否则不会接收到。苹果提供了两个注册通知的方法：

```
- (void)addObserver:(id)observer
           selector:(SEL)aSelector 
           name:(nullable NSString *)aName
            object:(nullable id)anObject;

- (id <NSObject>)addObserverForName:(nullable NSString *)name 
                             object:(nullable id)obj
                              queue:(nullable NSOperationQueue *)queue 
                              usingBlock:(void (^)(NSNotification *note))block
```

第一个方法使用的方式是最多的，第二个方法很少见到使用。第二个方法是支持`block`回调的，而`queue`参数就是在哪个队列。

#取消注册通知

---
通知注册了，在不需要时还需要移除注册。通知中心不会保留监听器对象, 在通知中心注册过的对象,必须在该对象释放前取消注册. 否则,当相应的通知再次出现时, 通知中心仍然会向该监听器发送消息. 因为, 相应的监听器对象已经被释放了, 所以可能会导致应用崩溃，这种崩溃很常见。

苹果提供了两个方法，可以移除注册：

```
- (void)removeObserver:(id)observer;

- (void)removeObserver:(id)observer
                  name:(nullable NSString *)aName
                object:(nullable id)anObject;
```

通常，我们会在控制器的`dealloc`之前，先取消注册通知：

```
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

如果我们只是想取消某一个通知的注册，而不是全部，那么可以使用第二个方法，将通知名称传过去，就会只取消注册该通知。

#键盘通知

---

对于键盘，我们经常需要注册与移除注册通知：

```
UIKeyboardWillShowNotification // 键盘即将显示 
UIKeyboardDidShowNotification // 键盘显示完毕 
UIKeyboardWillHideNotification // 键盘即将隐藏 
UIKeyboardDidHideNotification // 键盘隐藏完毕 
UIKeyboardWillChangeFrameNotification // 键盘的位置尺寸即将发生改变 
UIKeyboardDidChangeFrameNotification // 键盘的位置尺寸改变完毕
```

#关注我

---
**微信公众号：[iOSDevShares]()**<br>
**有问必答QQ群：324400294**