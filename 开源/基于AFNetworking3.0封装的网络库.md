#概述

对于开发人员来说，学习网络层知识是必备的，任何一款`App`的开发，都需要到网络请求接口。很多朋友都还在使用原生的`NSURLConnection`一行一行地写，代码到处是，这样维护起来更困难了。

对于使用`AFNetworking`的朋友来说，很多朋友都是直接调用`AFNetworking`的`API`，这样不太好，无法做到全工程统一配置。

最好的方式就是对网络层再封装一层，全工程不允许直接使用`AFNetworking`的`API`，必须调用我们自己封装的一层，如此一来，任何网络配置都可以在这一层里配置好，使用的人无须知道里面在干嘛，只管调用就可以了。

本篇为**基于AFNetworking3.0以上**的版本，支持iOS7及其以上版本。若要支持iOS6，请阅读旧版本：[基于AFNetworking2.5封装](http://www.henishuo.com/base-on-afnetworking-wrapper/)

#Version 3.2.1

* 完善缓存机制及无网或者网络异常状态下取缓存数据

#Version 3.2.0

* 增加超时设置
* 增加网络异常时是否读取本地缓存的策略

#升级为3.0版本

* 简化API，以降低使用的要求
* 增加GET/POST数据缓存、获取缓存大小、清空缓存功能
* 接口增加刷新缓存功能
* 增加取消所有请求、取消单个请求功能
* 格式化打印日志
* 增加对手动取消请求接口是否在失败时还回调的控制

#常用接口类型

应用开发过程中，所使用类型通常是`GET`、`POST`及上传图片。因此，这里只是对这几种类型提供`API`。

##GET接口

这里提供了两个`GET`请求的`API`，需要一般情况下`GET`请求都是直接写一个完整的`URL`，但是有时候为了参数可读性更强，改成传一个字典过来更容易阅读。

`HYBResponseSuccess`是响应成功的回调，返回的是字典，外部再转换成模型就可以了。<br>
`HYBResponseFail`是响应失败的回调，只有一个`NSError`对象，外部可接收处理。

```
/*!
 *  @author 黄仪标, 15-11-15 13:11:50
 *
 *  GET请求接口，若不指定baseurl，可传完整的url
 *
 *  @param url     接口路径，如/path/getArticleList
 *  @param refreshCache 是否刷新缓存。由于请求成功也可能没有数据，对于业务失败，只能通过人为手动判断
 *  @param params  接口中所需要的拼接参数，如@{"categoryid" : @(12)}
 *  @param success 接口成功请求到数据的回调
 *  @param fail    接口请求数据失败的回调
 *
 *  @return 返回的对象中有可取消请求的API
 */
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
                     refreshCache:(BOOL)refreshCache
                          success:(HYBResponseSuccess)success
                             fail:(HYBResponseFail)fail;
// 多一个params参数
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
                     refreshCache:(BOOL)refreshCache
                           params:(NSDictionary *)params
                          success:(HYBResponseSuccess)success
                             fail:(HYBResponseFail)fail;
// 多一个带进度回调
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
                     refreshCache:(BOOL)refreshCache
                           params:(NSDictionary *)params
                         progress:(HYBGetProgress)progress
                          success:(HYBResponseSuccess)success
                             fail:(HYBResponseFail)fail;
```

##POST接口


对于`POST`请求类型的接口，只有一个，看注释就可以明白如何使用了。

```
/*!
 *  @author 黄仪标, 15-11-15 13:11:50
 *
 *  POST请求接口，若不指定baseurl，可传完整的url
 *
 *  @param url     接口路径，如/path/getArticleList
 *  @param params  接口中所需的参数，如@{"categoryid" : @(12)}
 *  @param success 接口成功请求到数据的回调
 *  @param fail    接口请求数据失败的回调
 *
 *  @return 返回的对象中有可取消请求的API
 */
+ (HYBURLSessionTask *)postWithUrl:(NSString *)url
                      refreshCache:(BOOL)refreshCache
                            params:(NSDictionary *)params
                           success:(HYBResponseSuccess)success
                              fail:(HYBResponseFail)fail;
+ (HYBURLSessionTask *)postWithUrl:(NSString *)url
                      refreshCache:(BOOL)refreshCache
                            params:(NSDictionary *)params
                          progress:(HYBPostProgress)progress
                           success:(HYBResponseSuccess)success
                              fail:(HYBResponseFail)fail;
```                                

##图片上传接口

接口一次只能上传一张图片，通常也是这么处理的。这里是以文件流的形式来上传的哦。其中，`mineType`为`image/jpeg`。

```
/**
 *	@author 黄仪标, 16-01-31 00:01:40
 *
 *	图片上传接口，若不指定baseurl，可传完整的url
 *
 *	@param image			图片对象
 *	@param url				上传图片的接口路径，如/path/images/
 *	@param filename		给图片起一个名字，默认为当前日期时间,格式为"yyyyMMddHHmmss"，后缀为`jpg`
 *	@param name				与指定的图片相关联的名称，这是由后端写接口的人指定的，如imagefiles
 *	@param mimeType		默认为image/jpeg
 *	@param parameters	参数
 *	@param progress		上传进度
 *	@param success		上传成功回调
 *	@param fail				上传失败回调
 *
 *	@return
 */
+ (HYBURLSessionTask *)uploadWithImage:(UIImage *)image
                                   url:(NSString *)url
                              filename:(NSString *)filename
                                  name:(NSString *)name
                              mimeType:(NSString *)mimeType
                            parameters:(NSDictionary *)parameters
                              progress:(HYBUploadProgress)progress
                               success:(HYBResponseSuccess)success
                                  fail:(HYBResponseFail)fail;
```

##上传文件接口

```
/**
 *	@author 黄仪标, 16-01-31 00:01:59
 *
 *	上传文件操作
 *
 *	@param url						上传路径
 *	@param uploadingFile	待上传文件的路径
 *	@param progress			上传进度
 *	@param success				上传成功回调
 *	@param fail					上传失败回调
 *
 *	@return
 */
+ (HYBURLSessionTask *)uploadFileWithUrl:(NSString *)url
                           uploadingFile:(NSString *)uploadingFile
                                progress:(HYBUploadProgress)progress
                                 success:(HYBResponseSuccess)success
                                    fail:(HYBResponseFail)fail;
```

##文件下载接口

```
/*!
 *  @author 黄仪标, 16-01-08 15:01:11
 *
 *  下载文件
 *
 *  @param url           下载URL
 *  @param saveToPath    下载到哪个路径下
 *  @param progressBlock 下载进度
 *  @param success       下载成功后的回调
 *  @param failure       下载失败后的回调
 */
+ (HYBURLSessionTask *)downloadWithUrl:(NSString *)url
                            saveToPath:(NSString *)saveToPath
                              progress:(HYBDownloadProgress)progressBlock
                               success:(HYBResponseSuccess)success
                               failure:(HYBResponseFail)failure;
```

#取消请求

```
/**
 *	@author 黄仪标
 *
 *	取消所有请求
 */
+ (void)cancelAllRequest;
/**
 *	@author 黄仪标
 *
 *	取消某个请求。如果是要取消某个请求，最好是引用接口所返回来的HYBURLSessionTask对象，
 *  然后调用对象的cancel方法。如果不想引用对象，这里额外提供了一种方法来实现取消某个请求
 *
 *	@param url				URL，可以是绝对URL，也可以是path（也就是不包括baseurl）
 */
+ (void)cancelRequestWithURL:(NSString *)url;
```

在使用中，可以通过这样来调用：

```
// 取消全部请求
//  [HYBNetworking cancelAllRequest];
  
// 取消单个请求方法一
//  [HYBNetworking cancelRequestWithURL:path];
  
// 取消单个请求方法二
//  [task cancel];
```

#缓存

```
/**
 *	@author 黄仪标
 *
 *	默认只缓存GET请求的数据，对于POST请求是不缓存的。如果要缓存POST获取的数据，需要手动调用设置
 *  对JSON类型数据有效，对于PLIST、XML不确定！
 *
 *	@param isCacheGet			默认为YES
 *	@param shouldCachePost	默认为NO
 */
+ (void)cacheGetRequest:(BOOL)isCacheGet shoulCachePost:(BOOL)shouldCachePost;

/**
 *	@author 黄仪标
 *
 *	获取缓存总大小/bytes
 *
 *	@return 缓存大小
 */
+ (unsigned long long)totalCacheSize;

/**
 *	@author 黄仪标
 *
 *	清除缓存
 */
+ (void)clearCaches;
```

# BaseURL

这里还提供了两个公共接口，一个是用于设置或者更新网络接口的基础URL，一个是获取当前设置使用的网络接口基础URL。

```
/*!
 *  @author 黄仪标, 15-11-15 13:11:45
 *
 *  用于指定网络请求接口的基础url，如：
 *  http://henishuo.com或者http://101.200.209.244
 *  通常在AppDelegate中启动时就设置一次就可以了。如果接口有来源
 *  于多个服务器，可以调用更新
 *
 *  @param baseUrl 网络接口的基础url
 */
+ (void)updateBaseUrl:(NSString *)baseUrl;
+ (NSString *)baseUrl;
```

#添加公共请求头参数

通常每家公司的接口都会设置公共的请求头参数，以代表是公司的接口。默认已经配置了可接收的类型，但是如果需要额外配置，可通过调用此`api`来添加：

```
/*!
 *  @author 黄仪标, 15-11-16 13:11:41
 *
 *  配置公共的请求头，只调用一次即可，通常放在应用启动的时候配置就可以了
 *
 *  @param httpHeaders 只需要将与服务器商定的固定参数设置即可
 */
+ (void)configCommonHttpHeaders:(NSDictionary *)httpHeaders;
```

#请求、响应类型

默认responseType和requestType都是JSON格式。如果不使用JSON，可以全局配置成自己希望的格式即可。若不配置，默认就是JSON。

```
/*!
 *  @author 黄仪标, 15-12-25 15:12:45
 *
 *  配置请求格式，默认为JSON。如果要求传XML或者PLIST，请在全局配置一下
 *
 *  @param requestType 请求格式，默认为JSON
 *  @param responseType 响应格式，默认为JSO，
 *  @param shouldAutoEncode YES or NO,默认为NO，是否自动encode url
 *  @param shouldCallbackOnCancelRequest 当取消请求时，是否要回调，默认为YES
 */
+ (void)configRequestType:(HYBRequestType)requestType
             responseType:(HYBResponseType)responseType
      shouldAutoEncodeUrl:(BOOL)shouldAutoEncode
  callbackOnCancelRequest:(BOOL)shouldCallbackOnCancelRequest;
```

#URL编码问题

考虑到网络请求接口中，有时候会有中文参数，这时候就会请求失败，因此我们要对这种类型的`URL`进行编码，否则请求会失败。这里是开启或者关闭自动将`URL`编码的接口，默认为NO，表示不开启。

```
/*!
 *  @author 黄仪标, 15-11-15 15:11:16
 *
 *  开启或关闭是否自动将URL使用UTF8编码，用于处理链接中有中文时无法请求的问题
 *
 *  @param shouldAutoEncode YES or NO,默认为NO
 */
+ (void)shouldAutoEncodeUrl:(BOOL)shouldAutoEncode;
```

#格式化接口数据打印日志

```
/*!
 *  @author 黄仪标, 15-11-15 14:11:40
 *
 *  开启或关闭接口打印信息
 *
 *  @param isDebug 开发期，最好打开，默认是NO
 */
+ (void)enableInterfaceDebug:(BOOL)isDebug;
```

开启后会有非常好的打印效果，效果如下：

![image](http://www.henishuo.com/wp-content/uploads/2016/03/QQ20160325-0@2x-1-e1458921436643.png)

通常在`AppDelegate`中应用启动的代理方法中调用设置为开启就可以了。不过是否设置为开启，当应用以发布证书打包时，都不会打印日志，因为这里做了处理，可放心使用。现在已经公开在外部，项目中都可以使用哦：

```
// 项目打包上线都不会打印日志，因此可放心。
#ifdef DEBUG
#define HYBAppLog(s, ... ) NSLog( @"[%@ in line %d] ===============>%@", [[NSString stringWithUTF8String:__FILE__] lastPathComponent], __LINE__, [NSString stringWithFormat:(s), ##__VA_ARGS__] )
#else
#define HYBAppLog(s, ... )
#endif
```

#安装使用

现在已经支持`cocoapods`，引入以下命令即可：

```
pod 'HYBNetworking', '~> 3.0.0'
```

或者直接下载源代码，拖入工程使用！

#源代码

请大家到我的`github`下载源代码：[HYBNetworking](https://github.com/CoderJackyHuang/HYBNetworking)

#温馨提示

最近老有人问：编译一直报错library not found for -lAFNetworking什么问题？

**注意：**如果您是使用cocoapods来管理第三方库的，那么直接通过上面安装使用的方式来安装即可，然后pod update一下。如果您不是使用cocoapods来引入的，请手动将AFNetworking对应的版本添加到工程。

