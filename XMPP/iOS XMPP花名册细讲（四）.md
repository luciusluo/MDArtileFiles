#前言

本篇文章是整理以前在CSDN上所发表的文章，更新文章内容，更详细地描述操作，以方便大家阅读和理解。

本篇文章的内容是一年前写的，不代表到现在还可以正常使用，但是即使有版本更新，功能上还是一样的，还是具有参考价值的。当然，对于笔者而言，就是重新收拾收拾很久没有使用过的知识了！如果您正在寻找XMPP方面的资料，我相信这里可以帮助到您！

本节是笔者回忆一年前所学习的资源而整理成本篇文章。本节只讲花名册及其对应于XMPP中的类。

#XMPPModule

在讲解花名册类之前，先讲讲其基类，XMPPModule是提供给所有extensions/modules下的类继承于此基类：

```
@interface XMPPModule : NSObject {
	XMPPStream *xmppStream;

	dispatch_queue_t moduleQueue;
	void *moduleQueueTag;
	
	id multicastDelegate;
}

// 调度队列
@property (readonly) dispatch_queue_t moduleQueue;
@property (readonly) void *moduleQueueTag;

@property (strong, readonly) XMPPStream *xmppStream;

- (id)init;
- (id)initWithDispatchQueue:(dispatch_queue_t)queue;

// 注册花名册模块的功能
- (BOOL)activate:(XMPPStream *)aXmppStream;
// 取消花名册模块的功能
- (void)deactivate;

// 设置代理，并指定模块队列，支持多代理
- (void)addDelegate:(id)delegate delegateQueue:(dispatch_queue_t)delegateQueue;

// 从队列移除代理
- (void)removeDelegate:(id)delegate delegateQueue:(dispatch_queue_t)delegateQueue;
// 移除代理
- (void)removeDelegate:(id)delegate;

- (NSString *)moduleName;

@end
```

这XMPPModule作为扩展模块的主要基类，提供了通用的API。下面可以开始讲讲花名册了！

#XMPPRoster

```
@interface XMPPRoster : XMPPModule {
	__strong id <XMPPRosterStorage> xmppRosterStorage;
    
    XMPPIDTracker *xmppIDTracker;
	
	Byte config;
	Byte flags;
	
	NSMutableArray *earlyPresenceElements;
	
	DDList *mucModules;
}

- (id)initWithRosterStorage:(id <XMPPRosterStorage>)storage;
- (id)initWithRosterStorage:(id <XMPPRosterStorage>)storage dispatchQueue:(dispatch_queue_t)queue;

// 花名册存储代理
@property (strong, readonly) id <XMPPRosterStorage> xmppRosterStorage;

// 自动获取花名册，默认为YES
@property (assign) BOOL autoFetchRoster;

// 当与服务器断点连接时，是否自动清除user和resource，默认为YES
@property (assign) BOOL autoClearAllUsersAndResources;

// 默认为YES，表示自动接受已知在线状态订阅请求
@property (assign) BOOL autoAcceptKnownPresenceSubscriptionRequests;

// 默认为NO。若设置为YES，且autoFetchRoster为NO，则永远不会自动获取花名册
@property (assign) BOOL allowRosterlessOperation;

// 花名册是否已经手动获取（手动调用过fetchRoster）或者
// 自动获取（调用autoFetchRoster）但是还没有populated
@property (assign, getter = hasRequestedRoster, readonly) BOOL requestedRoster;

// 获取花名册是否正在populating
@property (assign, getter = isPopulating, readonly) BOOL populating;

// 获取是否有花名册。
@property (assign, readonly) BOOL hasRoster;

// 手动获取花名册
- (void)fetchRoster;

// 将用户添加到花名册
- (void)addUser:(XMPPJID *)jid withNickname:(NSString *)optionalName;
// 将用户添加到花名册，可设置昵称、分组
- (void)addUser:(XMPPJID *)jid withNickname:(NSString *)optionalName groups:(NSArray *)groups;
- (void)addUser:(XMPPJID *)jid withNickname:(NSString *)optionalName groups:(NSArray *)groups subscribeToPresence:(BOOL)subscribe;

// 设置或修改用户的昵称
- (void)setNickname:(NSString *)nickname forUser:(XMPPJID *)jid;

// 移除用户
- (void)removeUser:(XMPPJID *)jid;

// 若当前接收不到指定user的presence，可以通过调用此API来发送要接收指定user的presence
- (void)subscribePresenceToUser:(XMPPJID *)jid;

// 取消订阅指定用户的presence
- (void)unsubscribePresenceFromUser:(XMPPJID *)jid;

// 若之前已经接收过user的presence且已经授权，则调用此API可直接获取授权
- (void)revokePresencePermissionFromUser:(XMPPJID *)jid;

// 接收来自指定user的presence
- (void)acceptPresenceSubscriptionRequestFrom:(XMPPJID *)jid andAddToRoster:(BOOL)flag;

// 拒绝接收来自指定user的presence
- (void)rejectPresenceSubscriptionRequestFrom:(XMPPJID *)jid;

@end
```

###XMPPRosterDelegate

要操作花名册，就得需要XMPPRosterDelegate协议了，有了它就可以任意操作花名册了~有些英文翻译过来感觉不太合适，因此就干脆不翻译了~因此阅读完之后，我只知道英文，并不知道中文该怎么表达~哈哈

```
// 花名册操作相关代理
@protocol XMPPRosterDelegate
@optional

// 当接收到presence订阅请求时会回调
// 通完[presence from]获取发送请求的user
- (void)xmppRoster:(XMPPRoster *)sender didReceivePresenceSubscriptionRequest:(XMPPPresence *)presence;

// 当接收到花名册Push（消息查询）时会回调
- (void)xmppRoster:(XMPPRoster *)sender didReceiveRosterPush:(XMPPIQ *)iq;

// 当接收到初始化花名册时，会回调
- (void)xmppRosterDidBeginPopulating:(XMPPRoster *)sender;

// 当接收到初始化花名册且已经存储到coredata时，会回调
- (void)xmppRosterDidEndPopulating:(XMPPRoster *)sender;

/**
 * Sent when the roster receives a roster item.
 *
 * Example:
 *
 * <item jid='romeo@example.net' name='Romeo' subscription='both'>
 *   <group>Friends</group>
 * </item>
**/
- (void)xmppRoster:(XMPPRoster *)sender didReceiveRosterItem:(NSXMLElement *)item;

@end
```

#XMPPRosterCoreDataStorage

XMPPRosterCoreDataStorage类是与花名册的数据存储直接相关的类，用于方便操作core data。它的基类是XMPPCoreDataStorage，提供了很多与数据库操作的API和属性。

这个类是个单例，与数据库操作都集中在此类中了。

```
@interface XMPPRosterCoreDataStorage : XMPPCoreDataStorage <XMPPRosterStorage> {
	// Inherits protected variables from XMPPCoreDataStorage
	
#if __has_feature(objc_arc_weak)
	__weak XMPPRoster *parent;
#else
	__unsafe_unretained XMPPRoster *parent;
#endif
	dispatch_queue_t parentQueue;
	void *parentQueueTag;
    
	NSMutableSet *rosterPopulationSet;
}


+ (instancetype)sharedInstance;


/* Inherited from XMPPCoreDataStorage
 * Please see the XMPPCoreDataStorage header file for extensive documentation.
 
- (id)initWithDatabaseFilename:(NSString *)databaseFileName storeOptions:(NSDictionary *)storeOptions;
- (id)initWithInMemoryStore;

@property (readonly) NSString *databaseFileName;
 
@property (readwrite) NSUInteger saveThreshold;

@property (readonly) NSManagedObjectModel *managedObjectModel;
@property (readonly) NSPersistentStoreCoordinator *persistentStoreCoordinator;

@property (readonly) NSManagedObjectContext *mainThreadManagedObjectContext;
 
*/

// 获取User
- (XMPPUserCoreDataStorageObject *)myUserForXMPPStream:(XMPPStream *)stream
                            managedObjectContext:(NSManagedObjectContext *)moc;

// 获取Resource
- (XMPPResourceCoreDataStorageObject *)myResourceForXMPPStream:(XMPPStream *)stream
                                          managedObjectContext:(NSManagedObjectContext *)moc;

// 获取User
- (XMPPUserCoreDataStorageObject *)userForJID:(XMPPJID *)jid
                                   xmppStream:(XMPPStream *)stream
                         managedObjectContext:(NSManagedObjectContext *)moc;

// 获取Resource
- (XMPPResourceCoreDataStorageObject *)resourceForJID:(XMPPJID *)jid
										   xmppStream:(XMPPStream *)stream
                                 managedObjectContext:(NSManagedObjectContext *)moc;

@end
```

###XMPPRosterStorage

要使用XMPPRoster，我们通常会使用XMPPFramework框架给我们提供的XMPPCoreDataStorage数据库模块功能。而要使用此功能，就需要通过XMPPRosterStorage代理来实现。

```
// 花名册数据存在代理
@protocol XMPPRosterStorage <NSObject>
@required

// 配置父花名册
- (BOOL)configureWithParent:(XMPPRoster *)aParent queue:(dispatch_queue_t)queue;

- (void)beginRosterPopulationForXMPPStream:(XMPPStream *)stream;
- (void)endRosterPopulationForXMPPStream:(XMPPStream *)stream;

- (void)handleRosterItem:(NSXMLElement *)item xmppStream:(XMPPStream *)stream;
- (void)handlePresence:(XMPPPresence *)presence xmppStream:(XMPPStream *)stream;

- (BOOL)userExistsWithJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;

- (void)clearAllResourcesForXMPPStream:(XMPPStream *)stream;
- (void)clearAllUsersAndResourcesForXMPPStream:(XMPPStream *)stream;

- (NSArray *)jidsForXMPPStream:(XMPPStream *)stream;

- (void)getSubscription:(NSString **)subscription
                    ask:(NSString **)ask
               nickname:(NSString **)nickname
                 groups:(NSArray **)groups
                 forJID:(XMPPJID *)jid
             xmppStream:(XMPPStream *)stream;

@optional

// 与头像模块（XMPPvCardAvatarModule）集成时，会使用到此API。
#if TARGET_OS_IPHONE
- (void)setPhoto:(UIImage *)image forUserWithJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;
#else
- (void)setPhoto:(NSImage *)image forUserWithJID:(XMPPJID *)jid xmppStream:(XMPPStream *)stream;
#endif

@end
```

#最后

到此，就讲了以下几个主要类和代理：

* XMPPModule（扩展模块的基类）
* XMPPRoster（花名册）
* XMPPRosterCoreDataStorage（花名册存储类）
* XMPPRosterStorage（花名册存储代理）
* XMPPRosterDelegate（花名册操作类）

#推荐阅读

* [Openfire+spark环境搭建](http://www.henishuo.com/xmpp-spark-openfire-setup/)
* [了解XMPP协议](http://www.henishuo.com/ios-xmpp-introduce/)
* [XMPPFramework核心类介绍](http://www.henishuo.com/xmppframeworkd-core-introduce/)


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



