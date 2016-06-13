#背景

网络连接状态检测对于我们的iOS app开发来说是一个非常通用的需求。为了更好的用户体验，我们会在无网络时展现本地或者缓存的内容，并对用户进行合适的提示。但事实上，当前iOS开发者们普遍使用的网络检测框架，实际上都无法帮助我们检测真正的网络连接状态；它们所能检测的只是本地连接状态。

#重要通知

原作者已更新，现在已支持IPV6 only！所有使用此库的朋友们，请注意升级！

#场景

本地连接状态和实际网络连接状态不一致的“伪连接”情况包括但不限于如下场景： 
 
1. 现在很流行的公用wifi，需要网页鉴权，鉴权之前无法上网，但本地连接已经建立；  
2. 存在了本地网络连接，但信号很差，实际无法连接到服务器；  
3. iOS连接的路由设备本身没有连接外网。  

很多国内外的网友关于此问题早有提问和吐槽，比如：
  
* [如何判断设备是否真正连上互联网？而不是只有网络连接](http://www.cocoachina.com/bbs/read.php?tid=283498)  
* [\[Reachability reachabilityWithHostName:\]完全没用！](http://www.cocoachina.com/bbs/read.php?tid=40989)   
* [国外网友对Reachability库缺陷在github上的提问](https://github.com/tonymillion/Reachability/issues/91)   


为了着手解决此类问题，笔者希望就此打造一个通用、简单、可靠并且能检测实际网络连接状态的框架，从而帮助提升app在上述场景下的用户体验；这就是[RealReachability](https://github.com/dustturtle/RealReachability)的由来。

#iOS下的网络检测方案

1. 从[苹果示例代码](https://developer.apple.com/library/ios/samplecode/Reachability/Introduction/Intro.html)衍生而来的各类变种，是被开发者们使用最多的；即各类reachability命名相关的框架。其中最负盛名的当属[tonymillion的Reachability](https://github.com/tonymillion/Reachability)。其基础代码来源于苹果，开发者们对其进行了扩展和语法上的支持（比如block回调等），使其更易于使用。  
2. AFNetworking框架中也提供了AFNetworkReachabilityManager，很多使用AFNetworking的小伙伴们会顺带使用该组件来进行网络检测。其代码基于苹果的[SCNetworkReachability API](https://developer.apple.com/library/mac/documentation/SystemConfiguration/Reference/SCNetworkReachabilityRef/#//apple_ref/doc/uid/TP40007260-CH101-SW4)，封装结构清晰，易于使用，且包含在AFNetworking中，也受到大家的喜爱。  
3. 一些app本身包含了例如xmpp之类的功能，此类实现通常会向连接的服务器定时发送心跳包，主要用于长连接的保活和断线处理；如果客户端没有收到server端回传的包，那么我们可以认为实际网络连接已经失效。sip协议也有类似的保活实现，定时间隔都是可配置的。基于server端连接稳定的前提，我们也可以利用这类心跳保活的机制间接达到实际网络状态检测的效果。  
4. 各种网络请求的超时。在上述伪连接的场景下，用户触发的http请求的通常结果就是超时，该时间一般不短于30秒。理论上说我们可以根据网络请求的结果来标识实际的网络状态，但这样做带来了实现上的耦合性（网络检测本应该是独立的功能模块），同时超时时间的等待也不利于打造一个很好的用户体验。

#RealReachability简单介绍

RealReachability是笔者1个月之前发布到github的开源库，项目地址如下：
https://github.com/dustturtle/RealReachability。  
短短1个月时间收获了几百个star，最近还上了github的[oc板块趋势排行榜](https://github.com/trending/objective-c)，开发者们对此框架的热情完全出乎了笔者的意料；这也正说明此框架抓住了开发者们的痛点。    

此框架开发的初衷来源于项目实际需求，离线模式对网络连接状态的要求比较苛刻，且实际场景经常会遇到“伪连接”的情况，项目中所使用的Reachability面对此场景力不从心。多方研究后引入了ping能力（此方案流量开销最小，也最简单），实现了简单的实际网络连接监测；后面将ping的能力和本地网络检测有机的结合，并且在实时性和开销上达成了一个平衡，不断提炼和优化，于是有了这个框架。  

可以告诉大家的是，这个框架在appstore上架应用中已经经受了考验，且经过了长时间的测试，可以放心使用；目前也已经有很多开发者朋友们在其app中使用了我们的框架（让笔者略感自豪的是，他们来自世界各地！）。

#RealReachability的实现原理

##RealReachability的架构

![realReachability架构概要图](http://www.henishuo.com/wp-content/uploads/2016/02/111.png)

RealReachability主要包含3大模块：connection、ping、FSM；

其中Ping模块通过对同样是苹果提供的ping样例代码进行了封装，connection模块实现了基于SCNetworkReachability的本地状态检测，FSM模块是有限状态机。

通过FSM的状态管理控制connection模块和Ping模块协同工作，剔除了重复的状态变化，并通过可配置的定时策略等业务逻辑优化，权衡了实时性和开销，最终得到了我们的实现。   
PS:其中connection模块和ping模块也可独立使用，分别提供本地网络检测和ping的能力。

感兴趣的读者也可以尝试（调用方式请参考RealReachability开源代码）。

##RealReachability的应用范围和场景

我们致力于打造一个打造一个通用、简单、可靠并且能检测实际网络连接状态的框架，从而帮助app开发者们在各种伪连接的场景下，也能够检测出网络真实的可达性，从而优化app的体验；现代架构的移动app大部分是胖客户端，即使离开了网络，也能提供很大一部分功能。RealReachability的能力也限于此。比如网速检测、个人热点信息检查等功能，不是此框架的设计目标，也超出了其能力范围。

##RealReachability的开销

localconnection是基于本地的检测，因此其带来的开销可以忽略；ping模块的单次ICMP探测包大小仅为64字节（包含包头部)，探测的频率默认为2分钟1次，对于客户端来说1小时的应用前台运行仅额外消耗2kb的流量，完全忽略不计；对于server端来说，如此低频率、低流量的分布式ICMP包，也不会带来压力：对比正常动辄几十kb甚至更多大小的请求，可以说是九牛一毛。

#RealReachability基础使用

##RealReachability集成和依赖

 - 最简便的集成方法当属pod: pod 'RealReachability'；当前的pod稳定版本版本号为1.1.1。
 - 手动集成：将RealReachability文件夹加入到工程即可。
 - 依赖：Xcode5.0+，支持ARC, iOS6+.项目需要引入SystemConfiguration.framework. 
  
##使用介绍

其接口的设计和调用方法和Reachability非常相似，大家可以无缝上手，非常方便。
开启网络监听：

```
[GLobalRealReachability startNotifier];
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(networkChanged:)
                                             name:kRealReachabilityChangedNotification
                                           object:nil];
```
关闭网络监听：

```
[GLobalRealReachability stopNotifier];
```
回调代码示例：

```
- (void)networkChanged:(NSNotification *)notification {
    RealReachability *reachability = (RealReachability *)notification.object;
    ReachabilityStatus status = [reachability currentReachabilityStatus];
    NSLog(@"currentStatus:%@",@(status));
}
```
查询当前实际网络连接状态：

```
ReachabilityStatus status = [reachability currentReachabilityStatus];
```
#RealReachability进阶使用（可选）

手动触发实时网络状态查询,可以在block回调中处理自己的业务（特别是网络请求操作），此接口的block会被异步调用。  
代码示例：

```
[GLobalRealReachability reachabilityWithBlock:^(ReachabilityStatus status) {
        switch (status)
        {
            case RealStatusNotReachable:
            {
            //  case NotReachable handler
                break;
            }
                
            case RealStatusViaWiFi:
            {
            //  case WiFi handler
                break;
            }
                
            case RealStatusViaWWAN:
            {
            //  case WWAN handler
                break;
            }
                
            default:
                break;
        }
    }];
```
手动设置目标host地址（ping探测的目标地址）：   

```
GLobalRealReachability.hostForPing = @"www.baidu.com";
```
注意：需要确保该host地址可以被ping探测到。此操作为可选，默认的目标host地址为www.baidu.com。

手动设置自动探测间隔(autoCheckInterval，单位为分钟)：
   
```
GLobalRealReachability.autoCheckInterval = 3.0f;
```

注意：此处根据应用的实时性要求来设置自动探测的间隔，实时性要求较高时可适当调低此参数，但不建议设置为1.0以下。

#结束语

希望这个框架能够帮助到大家的iOS开发！ 遇到任何疑问或者使用上的问题，都可以联系我(管振纬/游族iOS高级工程师)，期待与您交流iOS开发技术：
<openglnewbee@163.com>  
对此框架我也会持续进行维护和优化，更希望感兴趣的朋友可以到github上pull request!
开源有你更精彩！

#源代码


[RealReachability](https://github.com/dustturtle/RealReachability)

#作者介绍

本文原作者：管振纬/游族iOS高级工程师  
作者QQ群：494669518  
原作者**管振纬**授权[标哥的技术博客](www.henishuo.com)发表本篇博文。

#参考链接

* https://github.com/dustturtle/RealReachability
* https://developer.apple.com/library/ios/samplecode/Reachability/Introduction/Intro.html
* https://developer.apple.com/library/mac/documentation/SystemConfiguration/Reference/SCNetworkReachabilityRef/#//apple_ref/doc/uid/TP40007260-CH101-SW4
* https://github.com/tonymillion/Reachability
* http://www.cocoachina.com/bbs/read.php?tid=283498
* https://github.com/tonymillion/Reachability/issues/91
* http://www.cocoachina.com/bbs/read.php?tid=40989
* http://rainbird.blog.51cto.com/211214/695979



