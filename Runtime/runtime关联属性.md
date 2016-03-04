>#[原文出自：标哥的技术博客](http://www.henishuo.com/runtime-association/)


#前言

---
在开发中经常需要给已有的类扩展添加方法和属性，但是`Objective-C`是不允许给已有类扩展属性的，因为类扩展是不会自动生成成员变量的。但是，苹果提供了`runtime`，我们可以通过`runtime`使用关联`API`就可以做到了。

#关联API介绍

---
我们先看看关联API，只有这三个API，使用也是非常简单的：

```
/** 
 * Sets an associated value for a given object using a given key and association policy.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * @param value The value to associate with the key key for object. Pass nil to clear an existing association.
 * @param policy The policy for the association. For possible values, see “Associative Object Behaviors.”
 * 
 * @see objc_setAssociatedObject
 * @see objc_removeAssociatedObjects
 */
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)

/** 
 * Returns the value associated with a given object for a given key.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * 
 * @return The value associated with the key \e key for \e object.
 * 
 * @see objc_setAssociatedObject
 */
id objc_getAssociatedObject(id object, const void *key)

/** 
 * Removes all associations for a given object.
 * 
 * @param object An object that maintains associated objects.
 * 
 * @note The main purpose of this function is to make it easy to return an object 
 *  to a "pristine state”. You should not use this function for general removal of
 *  associations from objects, since it also removes associations that other clients
 *  may have added to the object. Typically you should use \c objc_setAssociatedObject 
 *  with a nil value to clear an association.
 * 
 * @see objc_setAssociatedObject
 * @see objc_getAssociatedObject
 */
void objc_removeAssociatedObjects(id object)
```

实际上，我们几乎不会使用到`objc_removeAssociatedObjects`函数，这个函数的功能是移除指定的对象上所有的关联。

##设置关联值


对于设置关联，我们需要使用下面的API关联起来：

```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
```

参数说明：

* `object`：与谁关联，通常是传`self`
* `key`：唯一键，在获取值时通过该键获取，通常是使用`static const void *`来声明
* `value`：关联所设置的值
* `policy`：内存管理策略，比如使用`copy`

##获取关联值

如果我们要获取所关联的值，需要通过`key`来获取，调用如下函数：

```
id objc_getAssociatedObject(id object, const void *key)
```

参数说明：

* `object`：与谁关联，通常是传`self`，在设置关联时所指定的与哪个对象关联的那个对象
* `key`：唯一键，在设置关联时所指定的键

##关联策略

我们先看看设置关联时所指定的`policy`，它是一个枚举类型，看官方说明：

```
/**
 * Policies related to associative references.
 * These are options to objc_setAssociatedObject()
 */
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```
我们说明一下各个值的作用：

* OBJC_ASSOCIATION_ASSIGN：表示弱引用关联，通常是基本数据类型，如`int`、`float`
* OBJC_ASSOCIATION_RETAIN_NONATOMIC：表示强（strong）引用关联对象
* OBJC_ASSOCIATION_COPY_NONATOMIC：表示关联对象copy
* OBJC_ASSOCIATION_RETAIN：表示强（strong）引用关联对象，但不是线程安全的
* OBJC_ASSOCIATION_COPY：表示关联对象copy，但不是线程安全的

#扩展属性

---
我们来写一个例子，扩展`UIControl`添加Block版本的`TouchUpInside`事件。

扩展头文件声明：

```
#import <UIKit/UIKit.h>

typedef void (^HYBTouchUpBlock)(id sender);

@interface UIControl (HYBBlock)

@property (nonatomic, copy) HYBTouchUpBlock hyb_touchUpBlock;

@end
```

扩展实现文件：

```
#import "UIControl+HYBBlock.h"
#import <objc/runtime.h>

static const void *sHYBUIControlTouchUpEventBlockKey = "sHYBUIControlTouchUpEventBlockKey";

@implementation UIControl (HYBBlock)

- (void)setHyb_touchUpBlock:(HYBTouchUpBlock)hyb_touchUpBlock {
  objc_setAssociatedObject(self,
                           sHYBUIControlTouchUpEventBlockKey,
                           hyb_touchUpBlock,
                           OBJC_ASSOCIATION_COPY);
  
  [self removeTarget:self
              action:@selector(hybOnTouchUp:)
    forControlEvents:UIControlEventTouchUpInside];
  
  if (hyb_touchUpBlock) {
    [self addTarget:self
             action:@selector(hybOnTouchUp:)
   forControlEvents:UIControlEventTouchUpInside];
  }
}

- (HYBTouchUpBlock)hyb_touchUpBlock {
  return objc_getAssociatedObject(self, sHYBUIControlTouchUpEventBlockKey);
}

- (void)hybOnTouchUp:(UIButton *)sender {
  HYBTouchUpBlock touchUp = self.hyb_touchUpBlock;
  
  if (touchUp) {
    touchUp(sender);
  }
}

@end
```

使用起来很简单吧！！！

#写在最后

---
本文章是专门介绍通过runtime如何给已有类添加扩展属性，如果文章中出现有疑问的地方，请在评论中评论，笔者会在第一时间回复您的！

且看且珍惜！！！

#关注我

---
如果在使用过程中遇到问题，或者想要与我交流，可加入有问必答**QQ群：[324400294]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

#支持并捐助

---
如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
