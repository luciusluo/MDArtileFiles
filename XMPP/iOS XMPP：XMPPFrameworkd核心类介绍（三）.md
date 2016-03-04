#前言

本篇文章是整理以前在CSDN上所发表的文章，更新文章内容，更详细地描述操作，以方便大家阅读和理解。

本篇文章的内容是一年前写的，不代表到现在还可以正常使用，但是即使有版本更新，功能上还是一样的，还是具有参考价值的。当然，对于笔者而言，就是重新收拾收拾很久没有使用过的知识了！如果您正在寻找XMPP方面的资料，我相信这里可以帮助到您！

#XMPPFramework结构

在进入下一步之前，先给大家讲讲XMPPFramework的目录结构，以便新手们更容易读懂文章。我们来看看下图：

![image](http://www.henishuo.com/wp-content/uploads/2016/02/屏幕快照-2016-02-20-上午7.41.06.png)

虽然这里有很多个目录，但是我们在开发中基本只关心Core和Extensions这两个目录下的类。各个目录主要用来干嘛的？

* Authentication：这一看名字就知道与授权验证相关的。
* Categories：主要是一些扩展，尤其是NSXMLElement+XMPP扩展是必备的。
* Core：这里是XMPP的核心文件目录，我们最主要的目光还是要放在这个目录上。
* Extensions：这个目录是XMPP的扩展，用于扩展各种协议和各种独立的功能，其下每个子目录都是对应的一个单独的子功能。我们最常用到的功能有Reconnect、Roster、CoreDataStorage等。
* Utilities：都是辅助类，我们开发者不用关心这里。
* Vendor：这个目录是XMPP所引用的第三方类库，如CocoaAsyncSocket、KissXML等，我们也不用关心这里。

阅读到此，对XMPPFramework的结构有所了解了吧！

#概念知识

登录需要到账号，而所谓的账号其实就是用户唯一标识符（JID），在XMPP中使用XMPPJID类来表示。那么，用户唯一标识（JID）有什么组成？

JID一般由三部分构成：用户名，域名和资源名，格式为user@domain/resource，例如：test@example.com/Anthony。对应于XMPPJID类中的三个属性user、domain、resource。

如果没有设置主机名（HOST），则使用JID的域名（domain）作为主机名，而端口号是可选的，默认是5222，一般也没有必要改动它。

#XMPPStream类

我们要与服务器连接，就必须通过XMPPStream类了，它提供了很多的API和属性设置，通过socket来实现的。我们看到Verdor目录了吗，包含了CocoaAsyncSocket这个非常有名的socket编程库。XMPPStream类还遵守并实现了GCDAsyncSocketDelegate代理，用于客户端与服务器交互。

```
@interface XMPPStream : NSObject <GCDAsyncSocketDelegate>
```

当我们创建XMPPStream对象后，我们需要设置代理，才能回调我们的代理方法，这个是支持multicast delegate，也就是说对于一个XMPPStream对象，可以设置多个代理对象，其中协议是XMPPStreamDelegate：

```
- (void)addDelegate:(id)delegate delegateQueue:(dispatch_queue_t)delegateQueue;
```

而当我们不希望某个XMPPStream对象继续接收到代理回调时，我们通过这样的方式来移除代理：

```
- (void)removeDelegate:(id)delegate delegateQueue:(dispatch_queue_t)delegateQueue;
- (void)removeDelegate:(id)delegate;
```

接下来，我们要设置主机和端口，通过设置这两个属性：

```
/**
 * The server's hostname that should be used to make the TCP connection.
 * 注释太长，简单说就是主机。这个属性是可选设置的，如果没有设置主机，默认会使用domain
 */
@property (readwrite, copy) NSString *hostName;

/**
 * The port the xmpp server is running on.
 * If you do not explicitly set the port, the default port will be used.
 * If you set the port to zero, the default port will be used.
 * 
 * The default port is 5222.
**/
@property (readwrite, assign) UInt16 hostPort;
```

XMPPStream有XMPPJID类对象作为属性，标识用户，因为我们后续很多操作都需要到myJID：

```
@property (readwrite, copy) XMPPJID *myJID;
```

而管理用户在线状态的就交由XMPPPresence类了，它同样被作为XMPPStream的属性，组合到XMPPStream中，后续很多关于用户的操作是需要到处理用户状态的：

```
/**
 * Represents the last sent presence element concerning the presence of myJID on the server.
 * In other words, it represents the presence as others see us.
 * 
 * This excludes presence elements sent concerning subscriptions, MUC rooms, etc.
 * 
 * @see resendMyPresence
**/
@property (strong, readonly) XMPPPresence *myPresence;
```

###XMPPStreamDelegate

这个协议是非常关键的，我们的很多主要操作都集中在这个协议的代理回调上。它分为好几种类型的代理API，比如授权的、注册的、安全的等：

```
@protocol XMPPStreamDelegate
@optional
// 将要与服务器连接是回调
- (void)xmppStreamWillConnect:(XMPPStream *)sender;

// 当tcp socket已经与远程主机连接上时会回调此代理方法
// 若App要求在后台运行，需要设置XMPPStream's enableBackgroundingOnSocket属性
- (void)xmppStream:(XMPPStream *)sender socketDidConnect:(GCDAsyncSocket *)socket;

// 当TCP与服务器建立连接后会回调此代理方法
- (void)xmppStreamDidStartNegotiation:(XMPPStream *)sender;

// TLS传输层协议在将要验证安全设置时会回调
// 参数settings会被传到startTLS
// 此方法可以不实现的，若选择实现它，可以可以在
// 若服务端使用自签名的证书，需要在settings中添加GCDAsyncSocketManuallyEvaluateTrust=YES
// 
- (void)xmppStream:(XMPPStream *)sender willSecureWithSettings:(NSMutableDictionary *)settings;

// 上面的方法执行后，下一步就会执行这个代理回调
// 用于在TCP握手时手动验证是否受信任
- (void)xmppStream:(XMPPStream *)sender didReceiveTrust:(SecTrustRef)trust
                                      completionHandler:(void (^)(BOOL shouldTrustPeer))completionHandler;

// 当stream通过了SSL/TLS的安全验证时，会回调此代理方法
- (void)xmppStreamDidSecure:(XMPPStream *)sender;

// 当XML流已经完全打开时（也就是与服务器的连接完成时）会回调此代理方法。此时可以安全地与服务器通信了。
- (void)xmppStreamDidConnect:(XMPPStream *)sender;

// 注册新用户成功时的回调
- (void)xmppStreamDidRegister:(XMPPStream *)sender;

// 注册新用户失败时的回调
- (void)xmppStream:(XMPPStream *)sender didNotRegister:(NSXMLElement *)error;

// 授权通过时的回调，也就是登录成功的回调
- (void)xmppStreamDidAuthenticate:(XMPPStream *)sender;

// 授权失败时的回调，也就是登录失败时的回调
- (void)xmppStream:(XMPPStream *)sender didNotAuthenticate:(NSXMLElement *)error;

// 将要绑定JID resource时的回调，这是授权程序的标准部分，当验证JID用户名通过时，下一步就验证resource。若使用标准绑定处理，return nil或者不要实现此方法
- (id <XMPPCustomBinding>)xmppStreamWillBind:(XMPPStream *)sender;

// 如果服务器出现resouce冲突而导致不允许resource选择时，会回调此代理方法。返回指定的resource或者返回nil让服务器自动帮助我们来选择。一般不用实现它。
- (NSString *)xmppStream:(XMPPStream *)sender alternativeResourceForConflictingResource:(NSString *)conflictingResource;

// 将要发送IQ（消息查询）时的回调
- (XMPPIQ *)xmppStream:(XMPPStream *)sender willReceiveIQ:(XMPPIQ *)iq;
// 将要接收到消息时的回调
- (XMPPMessage *)xmppStream:(XMPPStream *)sender willReceiveMessage:(XMPPMessage *)message;
// 将要接收到用户在线状态时的回调
- (XMPPPresence *)xmppStream:(XMPPStream *)sender willReceivePresence:(XMPPPresence *)presence;

/**
 * This method is called if any of the xmppStream:willReceiveX: methods filter the incoming stanza.
 * 
 * It may be useful for some extensions to know that something was received,
 * even if it was filtered for some reason.
**/
// 当xmppStream:willReceiveX:(也就是前面这三个API回调后)，过滤了stanza，会回调此代理方法。
// 通过实现此代理方法，可以知道被过滤的原因，有一定的帮助。
- (void)xmppStreamDidFilterStanza:(XMPPStream *)sender;

// 在接收了IQ（消息查询后）会回调此代理方法
- (BOOL)xmppStream:(XMPPStream *)sender didReceiveIQ:(XMPPIQ *)iq;
// 在接收了消息后会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didReceiveMessage:(XMPPMessage *)message;
// 在接收了用户在线状态消息后会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didReceivePresence:(XMPPPresence *)presence;

// 在接收IQ/messag、presence出错时，会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didReceiveError:(NSXMLElement *)error;

// 将要发送IQ（消息查询时）时会回调此代理方法
- (XMPPIQ *)xmppStream:(XMPPStream *)sender willSendIQ:(XMPPIQ *)iq;
// 在将要发送消息时，会回调此代理方法
- (XMPPMessage *)xmppStream:(XMPPStream *)sender willSendMessage:(XMPPMessage *)message;
// 在将要发送用户在线状态信息时，会回调此方法
- (XMPPPresence *)xmppStream:(XMPPStream *)sender willSendPresence:(XMPPPresence *)presence;

// 在发送IQ（消息查询）成功后会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didSendIQ:(XMPPIQ *)iq;
// 在发送消息成功后，会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didSendMessage:(XMPPMessage *)message;
// 在发送用户在线状态信息成功后，会回调此方法
- (void)xmppStream:(XMPPStream *)sender didSendPresence:(XMPPPresence *)presence;

// 在发送IQ（消息查询）失败后会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didFailToSendIQ:(XMPPIQ *)iq error:(NSError *)error;
// 在发送消息失败后，会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didFailToSendMessage:(XMPPMessage *)message error:(NSError *)error;
// 在发送用户在线状态失败信息后，会回调此方法
- (void)xmppStream:(XMPPStream *)sender didFailToSendPresence:(XMPPPresence *)presence error:(NSError *)error;

// 当修改了JID信息时，会回调此代理方法
- (void)xmppStreamDidChangeMyJID:(XMPPStream *)xmppStream;

// 当Stream被告知与服务器断开连接时会回调此代理方法
- (void)xmppStreamWasToldToDisconnect:(XMPPStream *)sender;

// 当发送了</stream:stream>节点时，会回调此代理方法
- (void)xmppStreamDidSendClosingStreamStanza:(XMPPStream *)sender;

// 连接超时时会回调此代理方法
- (void)xmppStreamConnectDidTimeout:(XMPPStream *)sender;

// 当与服务器断开连接后，会回调此代理方法
- (void)xmppStreamDidDisconnect:(XMPPStream *)sender withError:(NSError *)error;

// p2p类型相关的
- (void)xmppStream:(XMPPStream *)sender didReceiveP2PFeatures:(NSXMLElement *)streamFeatures;
- (void)xmppStream:(XMPPStream *)sender willSendP2PFeatures:(NSXMLElement *)streamFeatures;


- (void)xmppStream:(XMPPStream *)sender didRegisterModule:(id)module;
- (void)xmppStream:(XMPPStream *)sender willUnregisterModule:(id)module;

// 当发送非XMPP元素节点时，会回调此代理方法。也就是说，如果发送的element不是
// <iq>, <message> 或者 <presence>，那么就会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didSendCustomElement:(NSXMLElement *)element;
// 当接收到非XMPP元素节点时，会回调此代理方法。也就是说，如果接收的element不是
// <iq>, <message> 或者 <presence>，那么就会回调此代理方法
- (void)xmppStream:(XMPPStream *)sender didReceiveCustomElement:(NSXMLElement *)element;
```

到此，也就理解了XMPPStream五五六六了吧！！！

#XMPPIQ

消息查询（IQ）就是通过此类来处理的了。XMPP给我们提供了IQ方便创建的类，用于快速生成XML数据。若头文件声明如下：


```
@interface XMPPIQ : XMPPElement

// 生成iq
+ (XMPPIQ *)iq;
+ (XMPPIQ *)iqWithType:(NSString *)type;
+ (XMPPIQ *)iqWithType:(NSString *)type to:(XMPPJID *)jid;
+ (XMPPIQ *)iqWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid;
+ (XMPPIQ *)iqWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid child:(NSXMLElement *)childElement;
+ (XMPPIQ *)iqWithType:(NSString *)type elementID:(NSString *)eid;
+ (XMPPIQ *)iqWithType:(NSString *)type elementID:(NSString *)eid child:(NSXMLElement *)childElement;
+ (XMPPIQ *)iqWithType:(NSString *)type child:(NSXMLElement *)childElement;

- (id)init;
- (id)initWithType:(NSString *)type;
- (id)initWithType:(NSString *)type to:(XMPPJID *)jid;
- (id)initWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid;
- (id)initWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid child:(NSXMLElement *)childElement;
- (id)initWithType:(NSString *)type elementID:(NSString *)eid;
- (id)initWithType:(NSString *)type elementID:(NSString *)eid child:(NSXMLElement *)childElement;
- (id)initWithType:(NSString *)type child:(NSXMLElement *)childElement;

// IQ类型，看下面的说明
- (NSString *)type;

// 判断type类型
- (BOOL)isGetIQ;
- (BOOL)isSetIQ;
- (BOOL)isResultIQ;
- (BOOL)isErrorIQ;

// 当type为get或者set时，这个API是很有用的，用于指定是否要求有响应
- (BOOL)requiresResponse;

- (NSXMLElement *)childElement;
- (NSXMLElement *)childErrorElement;

@end
```

IQ是一种请求／响应机制，从一个实体从发送请求，另外一个实体接受请求并进行响应。例如，client在stream的上下文中插入一个元素，向Server请求得到自己的好友列表，Server返回一个，里面是请求的结果。 

###\<type>\</type>有以下类别（可选设置如：\<type>get\</type>）：

* get :获取当前域值。类似于http get方法。
* set :设置或替换get查询的值。类似于http put方法。
* result :说明成功的响应了先前的查询。类似于http状态码200。
* error: 查询和响应中出现的错误。


下面是一个IQ例子：

```
<iq from="huangyibiao@welcome.com/ios"  
    id="xxxxxxx" 
    to="biaoge@welcome.com/ios"  
    type="get"> 
  <query xmlns="jabber:iq:roster"/> 
</iq> 
```

#XMPPPresence

这个类代表<presence></presence>节点，我们通过此类提供的方法来生成XML数据。它代表用户在线状态，它的头文件内容很少的：

```
@interface XMPPPresence : XMPPElement

// Converts an NSXMLElement to an XMPPPresence element in place (no memory allocations or copying)
+ (XMPPPresence *)presenceFromElement:(NSXMLElement *)element;

+ (XMPPPresence *)presence;
+ (XMPPPresence *)presenceWithType:(NSString *)type;
// type：用户在线状态，看下面的讲解
// to：接收方的JID
+ (XMPPPresence *)presenceWithType:(NSString *)type to:(XMPPJID *)to;

- (id)init;
- (id)initWithType:(NSString *)type;

// type：用户在线状态，看下面的讲解
// to：接收方的JID
- (id)initWithType:(NSString *)type to:(XMPPJID *)to;

- (NSString *)type;

- (NSString *)show;
- (NSString *)status;

- (int)priority;

- (int)intShow;

- (BOOL)isErrorPresence;

@end
```

presence用来表明用户的状态，如：online、away、dnd(请勿打扰)等。当改变自己的状态时，就会在stream的上下文中插入一个Presence元素，来表明自身的状态。要想接受presence消息，必须经过一个叫做presence subscription的授权过程。 

###\<type>\</type>有以下类别（可选设置如：\<type>subscribe\</type>）：

* subscribe：订阅其他用户的状态
* probe：请求获取其他用户的状态
* unavailable：不可用，离线（offline）状态


###\<show>\</show>节点有以下类别，如\<show>dnd\</show>：

* chat：聊天中
* away：暂时离开
* xa：eXtend Away，长时间离开
* dnd：勿打扰

###\<status>\</status>节点

这个节点表示状态信息，内容比较自由，几乎可以是所有类型的内容。常用来表示用户当前心情，活动，听的歌曲，看的视频，所在的聊天室，访问的网页，玩的游戏等等。

###\<priority>\</priority>节点

范围-128~127。高优先级的resource能接受发送到bare JID的消息，低优先级的resource不能。优先级为负数的resource不能收到发送到bare JID的消息。

发送一个用户在线状态的例子：

```
<presence from="alice@wonderland.lit/pda"> 
  <show>dnd</show> 
  <status>浏览器搜索：标哥的技术博客，或者直接访问www.henishuo.com</status> 
</presence> 
```

#XMPPMessage

XMPPMessage是XMPP框架给我们提供的，方便用于生成XML消息的数据，其头文件如下：

```
@interface XMPPMessage : XMPPElement

+ (XMPPMessage *)messageFromElement:(NSXMLElement *)element;

+ (XMPPMessage *)message;
+ (XMPPMessage *)messageWithType:(NSString *)type;
+ (XMPPMessage *)messageWithType:(NSString *)type to:(XMPPJID *)to;
+ (XMPPMessage *)messageWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid;
+ (XMPPMessage *)messageWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid child:(NSXMLElement *)childElement;
+ (XMPPMessage *)messageWithType:(NSString *)type elementID:(NSString *)eid;
+ (XMPPMessage *)messageWithType:(NSString *)type elementID:(NSString *)eid child:(NSXMLElement *)childElement;
+ (XMPPMessage *)messageWithType:(NSString *)type child:(NSXMLElement *)childElement;

- (id)init;
- (id)initWithType:(NSString *)type;
- (id)initWithType:(NSString *)type to:(XMPPJID *)to;
- (id)initWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid;
- (id)initWithType:(NSString *)type to:(XMPPJID *)jid elementID:(NSString *)eid child:(NSXMLElement *)childElement;
- (id)initWithType:(NSString *)type elementID:(NSString *)eid;
- (id)initWithType:(NSString *)type elementID:(NSString *)eid child:(NSXMLElement *)childElement;
- (id)initWithType:(NSString *)type child:(NSXMLElement *)childElement;

- (NSString *)type;
- (NSString *)subject;
- (NSString *)body;
- (NSString *)bodyForLanguage:(NSString *)language;
- (NSString *)thread;

- (void)addSubject:(NSString *)subject;
- (void)addBody:(NSString *)body;
- (void)addBody:(NSString *)body withLanguage:(NSString *)language;
- (void)addThread:(NSString *)thread;

- (BOOL)isChatMessage;
- (BOOL)isChatMessageWithBody;
- (BOOL)isErrorMessage;
- (BOOL)isMessageWithBody;

- (NSError *)errorMessage;

@end
```

message是一种基本**推送**消息方法，它不要求响应。主要用于IM、groupChat、alert和notification之类的应用中。

###\<type>\</type>有以下类别（可选设置如：\<type> chat\</type>）：

* normal：类似于email，主要特点是不要求响应；
* chat：类似于qq里的好友即时聊天，主要特点是实时通讯；
* groupchat：类似于聊天室里的群聊；
* headline：用于发送alert和notification；
* error：如果发送message出错，发现错误的实体会用这个类别来通知发送者出错了；

###\<body>\</body>节点

所要发送的内容就放在body节点下

消息节点的例子：

```
<message to="lily@jabber.org/contact" type="chat"> 
    <body>您好？您的博客名是叫标哥的技术博客吗？地址是http://www.henishuo.com吗？</body> 
</message> 
```

#最后

本篇文章就到此为止吧，讲了那么多，再讲下去可能连笔者都晕了~好好只收吧！如果文章中有错误的地方，请大家在评论中提出或者直接在QQ上联系笔者！谢谢~

期待下一篇吧~

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



