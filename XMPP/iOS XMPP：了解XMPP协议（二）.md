#前言

本篇文章来源于整理他人的文章，这里只整理出与iOS相关的部分！


#XMPP简介

本小节将简要介绍 XMPP，它的起源，以及为何它是一个适合实时 web 通信的协议。您将检查 XMPP 通信设置的组件，并查看展示这些组件如何使用的示例。

XMPP是Extensible Messaging and Presence Protocol的缩写！

##Web标准和XMPP

XMPP 是一组基于 XML 的技术，用于实时应用程序。最初，XMPP 作为一个框架开发，目标是支持企业环境内的即时消息传递和联机状态应用程序。当时的即时消息传递网络是私有的，非常不适合企业使用。例如，AOL Instant Messenger 不能针对公司内的安全通信进行调整。尽管存在一些商业解决方案，但它们固定的特性集通常不能进行调整，以满足组织的特殊需求。XMPP，当时名为 Jabber，允许组织构建自己的定制工具来促进实时通信，并允许安装现成的第三方解决方案。

XMPP 是一个分散型通信网络，这意味着，只要网络基础设施允许，任何 XMPP 用户都可以向其他任何 XMPP 用户传递消息。多个 XMPP 服务器也可以通过一个专门的 “服务器-服务器” 协议相互通信，提供了创建分散型社交网络和协作框架的有趣可能性，但这个主题已超出了本教程的讨论范围。

顾名思义，XMPP 可用于满足广泛的、对时间敏感的特性要求。实际上，Google Wave，一个大型多用户协作环境，将 XMPP 作为其联合协议的基础。尽管 XMPP 的出现是为了满足 “个人-个人” 即时消息传递的要求，但它完全不必局限于此任务。

#XMPP通信的结构

要促进消息传递，每个 XMPP 客户端用户必须拥有一个全局惟一标识符。基于历史原因，这些标识符称为 Jabber IDs，或称为 JIDs。鉴于这个协议的分布式特征，重要的是 JID 应包含联系用户所需的所有信息：不存在将用户链接到他们连接到的服务器的中央知识库。JID 的结构类似于电子邮件地址（但不要求 JID 同时也是有效的电子邮件收件人）。

客户端和服务器节点，我将它们统称为 XMPP 实体，都拥有 JIDs。SomeCorp 公司的员工 John Doe 可能拥有 JIDJohn.Doe@somecorp.com。这里，somecorp.com 是 SomeCorp 公司的 XMPP 服务器的地址，John.Doe 是 John Doe 的用户名。

JIDs 还拥有连接到它们的资源。这允许在一个 XMPP 实体标识符之外进一步处理细粒度；例如，尽管上面的示例总体上能够表示 John Doe，但 John.Doe@somecorp.com/Work 可以用于将数据发送到与他的工作相关的工具。

这些资源可以采用任意用户定义的名称，一个 XMPP 实体可以拥有任意数量的资源。除了可以是上下文依赖的外，它们还可以绑定到设备、工具或工作站。对于您的 Pingstream 示例，web 站点的每个访问者都将作为同一个用户登录 XMPP 服务器，但他们拥有不同的资源。

#XMPP通信类别

使用 XMPP 的实时消息传递系统包含三大通信类别：

消息传递，其中数据在有关各方之间传输；
联机状态，它允许用户广播其在线状态和可用性；
信息/查询请求，它允许 XMPP 实体发起请求并从另一个实体接收响应。
这些类别是互补的。例如，如果用户或实体离线（尽管在许多用例中，理想的状态是服务器在用户返回之前一直持有用户的消息），则没有将数据发送给用户或发起一个实体的信息/查询请求的点。这些消息中的每一条都将通过一个完整的 XML节 传递 — XML 节是以 XML 表达的独立信息项。

这三种类型的 XMPP 节都拥有以下公共属性：

* from：源 XMPP 实体的 JID；
* to：目标接收者的 JID；
* id：这次对话的可选标识符；
* type：节的可选子类型；
* xml:lang：如果内容是人们可读的，则为消息语言的描述。

基于 XMPP 的数据传输发生在一些 XML 流上，默认在端口 5222 上操作。这些 XML 流实际上是两个完整的 XML 文档，每个文档对应一个通信方向。一旦会话建立，stream 元素将打开。这个元素将封装整个通信文档。然后，一些节被注入这个文档的第二层。最后，一旦通信结束，stream 元素将关闭，形成一个完整的文档。

例如，清单1展示了一个 stream 元素，它建立了从客户端到服务器的通信。建立从客户端到服务器的通信的 stream 标记：

```					
<stream:stream from="[server]" id="[unique ID over conversation]" 
xmlns="jabber:client" xmlns:stream="http://etherx.jabber.org/streams" version="1.0">
```

##消息

一旦通信建立，客户端就能使用 message 元素将消息发送到另一个用户，message 元素包含以下任意子元素：

* subject：一个可读的字符串，表示消息主题。
* body：一个可读的字符串，表示消息体。如果每个消息体标记都拥有一个不同的 xml:lang 值，那么可以包含多个消息体标记。（xml:lang 是惟一可能的属性。）
* thread：一个惟一标识符，表示一个消息线程。客户端软件可以使用这个子元素将相关消息串联在一起。

但是，消息也可以非常简单，如清单2所示：

```
<message from="sendinguser@somedomain" to="recipient@somedomain" xml:lang='en'>
  <body>
    Body of message
  </body>
</message>
```

对于提供实时 web 界面而言，消息节是最有用的节。“发布-订阅” 模型 — 在实时 web 应用程序中使用消息来传输数据的一种替代方法 — 将稍后介绍。

##信息/查询

信息/查询节拥有广泛的功能。一个例子就是 “发布-订阅” 模型，在该模型中，发布者通知服务器某个特定资源进行了更新，服务器则通知已选择订阅这些通知并拥有适当授权的所有 XMPP 用户。

来自发布者的一系列项目被编码为一些节，格式为基于 XML 的 Atom 发布格式。每个项目都包含在一个 item 元素内，然后合并到一个 pubsub 元素中，最后成为一个信息/查询节。在 清单 3（选自 XMPP 发布-订阅规范）中，Shakespeare's Hamlet（JID 为 hamlet@denmark.lit/blogbot）用他著名的独白发布一个更新到 pubsub.shakespeare.lit pubsub 更新节点：


清单 3. 对 pubsub.shakespeare.lit pubsub 更新节点的更新

```					
<iq type="set"
    from="hamlet@denmark.lit/blogbot"
    to="pubsub.shakespeare.lit"
    id="pub1">
  <pubsub xmlns="http://jabber.org/protocol/pubsub">
    <publish node="princely_musings">
      <item>
        <entry xmlns="http://www.w3.org/2005/Atom">
          <title>Soliloquy</title>
          <summary>
To be, or not to be: that is the question:
Whether 'tis nobler in the mind to suffer
The slings and arrows of outrageous fortune,
Or to take arms against a sea of troubles,
And by opposing end them?
          </summary>
          <link rel="alternate" type="text/html"
                href="http://denmark.lit/2003/12/13/atom03"/>
          <id>tag:denmark.lit,2003:entry-32397</id>
          <published>2003-12-13T18:30:02Z</published>
          <updated>2003-12-13T18:30:02Z</updated>
        </entry>
      </item>
    </publish>
  </pubsub>
</iq>
``` 

信息/查询节也用于请求一个特定 XMPP 实体的有关信息。例如，在 清单 4 中的节中，boreduser@somewhere 正在查找friendlyuser@somewhereelse 拥有的公共项目。


清单 4. 用户查找由 friendlyuser@somewhereelse 拥有的公共项目

```			
<iq type="get"
    from="boreduser@somewhere"
    to="friendlyuser@somewhereelse"
    id="publicStuff">
  <query xmlns="http://jabber.org/protocol/disco#items"/>
</iq>
```

反过来，friendlyuser@somewhereelse 使用一列可被订阅到使用 “发布-订阅” 的项目进行响应，如 清单 5 所示：

```					
<iq type="result"
    from="friendlyuser@somewhereelse"
    to="boreduser@somewhere"
    id="publicStuff">
  <query xmlns="http://jabber.org/protocol/disco#items">
    <item jid="stuff.to.do"
          name="Things to do"/>
    <item jid="stuff.to.not.do"
          name="Things to avoid doing"/>
  </query>
</iq>
```

在 清单 5 中的信息/查询节中的每个返回项目都拥有一个可以订阅到的 JID。信息/查询还允许超出本教程范围的广泛的服务器信息请求。它们中的许多在针对多服务器环境的 web 应用程序上下文中有用，或者作为复杂的分散型协作框架的基础。

##联机状态

联机状态信息包含在一个联机状态（presence）节中。如果 type 属性省略，那么 XMPP 客户端应用程序假定用户在线且可用。否则，type 可设置为 unavailable，或者特定于 pubsub 的值：subscribe、subscribed、unsubscribe 和unsubscribed。它也可以是针对另一个用户的联机状态信息的一个错误或探针。

一个联机状态节可以包含以下子元素：

* show：一个机器可读的值，表示要显示的在线状态的总体类别。这可以是 away（暂时离开）
* chat（可用且有兴趣交流）、dnd（请勿打扰）、或 xa（长时间离开）。
* status：一个可读的 show 值。该值为用户可定义的字符串。
* priority：一个位于 -128 到 127 之间的值，定义消息路由到用户的优先顺序。如果值为负数，用户的消息将被扣留。

例如，清单 6 中的 boreduser@somewhere 可以用这个节来表明聊天意愿：

```
<presence xml:lang="en">
  <show>chat</show>
  <status>Bored out of my mind</status>
  <priority>1</priority>
</presence>
```

注意from属性此处省略了。另一个用户 friendlyuser@somewhereelse 可以通过发送 清单 7 中的节来探测 boreduser@somewhere 的状态：

```
<presence type="probe" from="friendlyuser@somewhereelse" to="boreduser@somewhere"/>
Boreduser@somewhere's server would then respond with a tailored presence response:
<presence xml:lang="en" from="boreduser@somewhere" to="friendlyuser@somewhereelse">
  <show>chat</show>
  <status>Bored out of my mind</status>
  <priority>1</priority>
</presence>
```

这些联机状态值源自 “个人-个人” 消息传递软件。show 元素的值 — 通常用于确定将向其他用户显示的状态图标 — 在聊天应用程序之外如何使用现在还不清楚。状态值可能会在微博工具中找到用武之地；例如，Google Talk（一个 XMPP 聊天服务）中的用户状态字段的更改可以被导入为 Google Buzz 中的微博条目。

另一种可能性就是将状态值用作每用户应用程序状态数据的携带者。尽管此规范将状态定义为可读，但没有什么能够阻止您在那里存储任意字符串来满足您的要求。对于某些应用程序而言，它可以不是可读的，或者，它可以携带微格式形态的数据负载。

您可以为一个 XMPP 实体拥有的每个资源独立设置联机状态信息，以便访问和接收连接到一个应用程序中的单个用户的所有工具和上下文的数据只需一个用户帐户。每个资源都可以被分配一个独立的优先级；XMPP 服务器将首先尝试将消息传递给优先级较高的资源。

 
#最后

阅读到此，应该对XMPP协议有所了解了。我们需要掌握文中所提到的节点名称，每个节点所对应的功能。


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




