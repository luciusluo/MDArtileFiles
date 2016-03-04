#前言

本篇文章讲解XMPP中的电子名片和头像模块，只讲理论相关知识，不讲实践。本专题后续会有专门讲解如何使用电子名片和头像模块。

在Extensions中有XEP-0054扩展，提供了一种可以通过XMPP发送电子名片的机制。

* vCard，也叫Versitcard，vCard的常用文件扩展名是.vcf。在XMPP中通过XMPPvCardTemp和XMPPvCardCoreDataStorage两个类来实现。
* vCard是电子名片的文件格式标准，一般附加在电子邮件之后，但也可以用于其它场合，比如在因特网上相互交换。

#为XMPPSteam添加电子名片扩展功能

Extensions扩展里的功能都是独立的，如果需要使用到相关功能，是需要自己手动添加的。当我们需要电子名片功能时，我们需要将电子名片扩展功能添加到XMPPStream中，才能使用。

首先，我们得引入电子名片扩展功能头文件：

```
// 电子名片相关模块头文件引入
#import "XMPPvCardTempModule.h"
#import "XMPPvCardTemp.h"
#import "XMPPvCardCoreDataStorage.h"
```

其中，三个类分别对应这样的功能：

* XMPPvCardTemp 代表电子名片
* XMPPvCardCoreDataStorage 代表电子名片在core data存储
* XMPPvCardTempModule 用于提供电子名片增、删、改、查操作

#XMPPvCardTemp类

XMPPvCardTemp就相当于电子名片类了，它是继承于NSXMLElement的。这个类提供了很多的属性，每个属性代表什么意思呢？电子名片上应该有什么，就会有哪些属性。看注释吧：

```
@interface XMPPvCardTemp : XMPPvCardTempBase

// 出生日期
@property (nonatomic, strong) NSDate *bday;
// 照片
@property (nonatomic, strong) NSData *photo;
// 昵称
@property (nonatomic, strong) NSString *nickname;
// full name
@property (nonatomic, strong) NSString *formattedName;
@property (nonatomic, strong) NSString *familyName;
@property (nonatomic, strong) NSString *givenName;
@property (nonatomic, strong) NSString *middleName;
@property (nonatomic, strong) NSString *prefix;
@property (nonatomic, strong) NSString *suffix;
// 地址数组
@property (nonatomic, strong) NSArray *addresses;
// Represents the actual text that should be put on the mailing label when delivering a physical package to the person/object associated with the vCard (related to the ADR property).
@property (nonatomic, strong) NSArray *labels;

@property (nonatomic, strong) NSArray *telecomsAddresses;
@property (nonatomic, strong) NSArray *emailAddresses;

@property (nonatomic, strong) XMPPJID *jid;
// 邮件
@property (nonatomic, strong) NSString *mailer;
// 时区
@property (nonatomic, strong) NSTimeZone *timeZone;
// 地埋位置
@property (nonatomic, strong) CLLocation *location;
// 职位
@property (nonatomic, strong) NSString *title;
// 角色。标准说明：The role, occupation, or business 
// category of the vCard object within an organization.
@property (nonatomic, strong) NSString *role;
// logo
@property (nonatomic, strong) NSData *logo;
// 标准定义：Information about another person who will act 
// on behalf of the vCard object. Typically this would
// be an area administrator, assistant, or secretary 
// for the individual. Can be either a URL or an embedded vCard.
@property (nonatomic, strong) XMPPvCardTemp *agent;
// 组织
@property (nonatomic, strong) NSString *orgName;

/*
 * ORGUNITs can only be set if there is already an ORGNAME. Otherwise, changes are ignored.
 */
// 部门信息
@property (nonatomic, strong) NSArray *orgUnits;
// A list of "tags" that can be used to describe the object represented by this vCard.
// 也就是分类标签
@property (nonatomic, strong) NSArray *categories;
// 电话
@property (nonatomic, strong) NSString *note;
@property (nonatomic, strong) NSString *prodid;
@property (nonatomic, strong) NSDate *revision;
@property (nonatomic, strong) NSString *sortString;
@property (nonatomic, strong) NSString *phoneticSound;
@property (nonatomic, strong) NSData *sound;
@property (nonatomic, strong) NSString *uid;
// 个人网站URL
@property (nonatomic, strong) NSString *url;
// 电子名片版本
@property (nonatomic, strong) NSString *version;
@property (nonatomic, strong) NSString *desc;

@property (nonatomic, assign) XMPPvCardTempClass privacyClass;
@property (nonatomic, strong) NSData *key;
@property (nonatomic, strong) NSString *keyType;

+ (XMPPvCardTemp *)vCardTempFromElement:(NSXMLElement *)element;
+ (XMPPvCardTemp *)vCardTemp;
+ (XMPPvCardTemp *)vCardTempSubElementFromIQ:(XMPPIQ *)iq;
+ (XMPPvCardTemp *)vCardTempCopyFromIQ:(XMPPIQ *)iq;
+ (XMPPIQ *)iqvCardRequestForJID:(XMPPJID *)jid;


- (void)addAddress:(XMPPvCardTempAdr *)adr;
- (void)removeAddress:(XMPPvCardTempAdr *)adr;
- (void)clearAddresses;


- (void)addLabel:(XMPPvCardTempLabel *)label;
- (void)removeLabel:(XMPPvCardTempLabel *)label;
- (void)clearLabels;


- (void)addTelecomsAddress:(XMPPvCardTempTel *)tel;
- (void)removeTelecomsAddress:(XMPPvCardTempTel *)tel;
- (void)clearTelecomsAddresses;


- (void)addEmailAddress:(XMPPvCardTempEmail *)email;
- (void)removeEmailAddress:(XMPPvCardTempEmail *)email;
- (void)clearEmailAddresses;


@end
```

下面是标准的一个小例子：

```
<?xml version="1.0" encoding="UTF-8"?>
<vcards xmlns="urn:ietf:params:xml:ns:vcard-4.0">
  <vcard>
    <tel>
      <parameters>
        <type>
          <text>work</text>
        </type>
      </parameters>
      <uri>tel:+1-111-555-1212</uri>
    </tel>
    <adr>
      <parameters>
        <type><text>work</text></type>
        <label><text>100 Waters Edge
                     Baytown, LA 30314
                     United States of America</text></label>
      </parameters>
    </adr>
    <email><text>forrestgump@example.com</text></email>
  </vcard>
</vcards>
```

#XMPPvCardCoreDataStorage

关于这个类的说明，就简单讲一讲。

```
@interface XMPPvCardCoreDataStorage : XMPPCoreDataStorage <
XMPPvCardAvatarStorage,
XMPPvCardTempModuleStorage> 

+ (instancetype)sharedInstance;

@end
```

它是一个单例类，直接与数据库有关。它遵守了XMPPvCardAvatarStorage，表示头像模块的存储代理，就可以将电子头像也写入电子名片数据库存储中。

遵守了XMPPvCardTempModuleStorage，就可以直接通过XMPPvCardTemp类对电子名片进行增、删、改、查了。

#XMPPvCardTempModule

继承于XMPPModule的类，主要是提供直接操作数据库的操作。

```
@interface XMPPvCardTempModule : XMPPModule
{
	id <XMPPvCardTempModuleStorage> __strong _xmppvCardTempModuleStorage;
    XMPPIDTracker *_myvCardTracker;
}


@property(nonatomic, strong, readonly) id <XMPPvCardTempModuleStorage> xmppvCardTempModuleStorage;
@property(nonatomic, strong, readonly) XMPPvCardTemp *myvCardTemp;

- (id)initWithvCardStorage:(id <XMPPvCardTempModuleStorage>)storage;
- (id)initWithvCardStorage:(id <XMPPvCardTempModuleStorage>)storage dispatchQueue:(dispatch_queue_t)queue;

// 若本地没有该电子名片，则从服务器提求
- (void)fetchvCardTempForJID:(XMPPJID *)jid;

// 获取某个jid的电子名片，是否忽略本地所存储的电子名片
- (void)fetchvCardTempForJID:(XMPPJID *)jid ignoreStorage:(BOOL)ignoreStorage;

// 获取某个jid的电子名片，是否自动从服务器提求
- (XMPPvCardTemp *)vCardTempForJID:(XMPPJID *)jid shouldFetch:(BOOL)shouldFetch;

// 这个API用于将电子名片存储到本地数据库，然后发送到服务器
- (void)updateMyvCardTemp:(XMPPvCardTemp *)vCardTemp;

@end


@protocol XMPPvCardTempModuleDelegate
@optional

- (void)xmppvCardTempModule:(XMPPvCardTempModule *)vCardTempModule 
        didReceivevCardTemp:(XMPPvCardTemp *)vCardTemp 
                     forJID:(XMPPJID *)jid;

- (void)xmppvCardTempModuleDidUpdateMyvCard:(XMPPvCardTempModule *)vCardTempModule;

- (void)xmppvCardTempModule:(XMPPvCardTempModule *)vCardTempModule failedToUpdateMyvCard:(NSXMLElement *)error;

@end


@protocol XMPPvCardTempModuleStorage <NSObject>

- (BOOL)configureWithParent:(XMPPvCardTempModule *)aParent queue:(dispatch_queue_t)queue;

- (XMPPvCardTemp *)vCardTempForJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;

- (void)setvCardTemp:(XMPPvCardTemp *)vCardTemp forJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;

- (XMPPvCardTemp *)myvCardTempForXMPPStream:(XMPPStream *)stream;

- (BOOL)shouldFetchvCardTempForJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;

@end
```

#如何激活电子名片功能

激活电子名片功能，步骤如下：

```
// 电子名片数据存储
XMPPvCardCoreDataStorage *vCardStorage = [XMPPvCardCoreDataStorage sharedInstance];

// 添加电子名片模块
_vCardModule = [[XMPPvCardTempModule alloc] initWithvCardStorage: vCardStorage];
// 激活
[_vCardModule activate:_xmppStream];
```

#最后

笔者能力有限，所描述之处若有不正确之处，请在评论中指出，以便快速修正。

#推荐阅读

* [Openfire+spark环境搭建（一）](http://www.henishuo.com/xmpp-spark-openfire-setup/)
* [了解XMPP协议（二）](http://www.henishuo.com/ios-xmpp-introduce/)
* [XMPPFramework核心类介绍（三）](http://www.henishuo.com/xmppframeworkd-core-introduce/)
* [iOS XMPP花名册细讲（四）](http://www.henishuo.com/ios-xmpp-roster/)

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




