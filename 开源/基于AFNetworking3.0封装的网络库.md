#前言

对于开发人员来说，学习网络层知识是必备的，任何一款`App`的开发，都需要到网络请求接口。很多朋友都还在使用原生的`NSURLConnection`一行一行地写，代码到处是，这样维护起来更困难了。

对于使用`AFNetworking`的朋友来说，很多朋友都是直接调用`AFNetworking`的`API`，这样不太好，无法做到全工程统一配置。

最好的方式就是对网络层再封装一层，全工程不允许直接使用`AFNetworking`的`API`，必须调用我们自己封装的一层，如此一来，任何网络配置都可以在这一层里配置好，使用的人无须知道里面在干嘛，只管调用就可以了。

>本篇为基于AFNetworking3.0的版本，支持iOS7及其以上版本。若要支持iOS6，请阅读旧版本：[http://www.henishuo.com/base-on-afnetworking-wrapper/](http://www.henishuo.com/base-on-afnetworking-wrapper/)

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
 *  @param url     接口路径，如/path/getArticleList?categoryid=1
 *  @param success 接口成功请求到数据的回调
 *  @param fail    接口请求数据失败的回调
 *
 *  @return 返回的对象中有可取消请求的API
 */
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
                          success:(HYBResponseSuccess)success
                             fail:(HYBResponseFail)fail;
/*!
 *  @author 黄仪标, 15-11-15 13:11:50
 *
 *  GET请求接口，若不指定baseurl，可传完整的url
 *
 *  @param url     接口路径，如/path/getArticleList
 *  @param params  接口中所需要的拼接参数，如@{"categoryid" : @(12)}
 *  @param success 接口成功请求到数据的回调
 *  @param fail    接口请求数据失败的回调
 *
 *  @return 返回的对象中有可取消请求的API
 */
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
                           params:(NSDictionary *)params
                          success:(HYBResponseSuccess)success
                             fail:(HYBResponseFail)fail;

// 支持进度
+ (HYBURLSessionTask *)getWithUrl:(NSString *)url
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
                            params:(NSDictionary *)params
                           success:(HYBResponseSuccess)success
                              fail:(HYBResponseFail)fail;

// 支持进度
+ (HYBURLSessionTask *)postWithUrl:(NSString *)url
                            params:(NSDictionary *)params
                          progress:(HYBPostProgress)progress
                           success:(HYBResponseSuccess)success
                              fail:(HYBResponseFail)fail;```
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

#设置基础URL

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

/*!
 *  @author 黄仪标, 15-11-15 13:11:06
 *
 *  对外公开可获取当前所设置的网络接口基础url
 *
 *  @return 当前基础url
 */
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

#请求与响应格式设置

默认responseType和requestType都是JSON格式。如果不使用JSON，可以全局配置成自己希望的格式即可。若不配置，默认就是JSON。

```
/*!
 *  @author 黄仪标, 15-12-25 15:12:38
 *
 *  配置返回格式，默认为JSON。若为XML或者PLIST请在全局修改一下
 *
 *  @param responseType 响应格式
 */
+ (void)configResponseType:(HYBResponseType)responseType;

/*!
 *  @author 黄仪标, 15-12-25 15:12:45
 *
 *  配置请求格式，默认为JSON。如果要求传XML或者PLIST，请在全局配置一下
 *
 *  @param requestType 请求格式
 */
+ (void)configRequestType:(HYBRequestType)requestType;
```

#URL编码问题

考虑到网络请求接口中，有时候会有中文参数，这时候就会请求失败，因此我们要对这种类型的`URL`进行编码，否则请求会失败。这里是开启或者关闭自动将`URL`编码的接口，默认为NO，表示不开启。

```
/*!
 *  @author 黄仪标, 15-11-15 14:11:40
 *
 *  开启或关闭接口打印信息
 *
 *  @param isDebug 开发期，最好打开，默认是NO
 */
+ (void)enableInterfaceDebug:(BOOL)isDebug;

/*!
 *  @author 黄仪标, 15-11-15 15:11:16
 *
 *  开启或关闭是否自动将URL使用UTF8编码，用于处理链接中有中文时无法请求的问题
 *
 *  @param shouldAutoEncode YES or NO,默认为NO
 */
+ (void)shouldAutoEncodeUrl:(BOOL)shouldAutoEncode;
```

#网络接口数据日志

对于网络请求回来的结果，如果没有一个格式化好的日志打印出来查看，就要通过断点一步步跟踪，然后打开出来看，这太麻烦。因此，这里提供了打印日志的私有`API`。默认是不开启打印日志的。

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

通常在`AppDelegate`中应用启动的代理方法中调用设置为开启就可以了。不过是否设置为开启，当应用以发布证书打包时，都不会打印日志，因为这里做了处理，可放心使用。

```
// 项目打包上线都不会打印日志，因此可放心。
#ifdef DEBUG
#define HYBAppLog(s, ... ) NSLog( @"[%@：in line: %d]-->[message: %@]", [[NSString stringWithUTF8String:__FILE__] lastPathComponent], __LINE__, [NSString stringWithFormat:(s), ##__VA_ARGS__] )
#else
#define HYBAppLog(s, ... )
#endif

```

#安装使用

现在已经支持`cocoapods`，引入以下命令即可：

```
pod 'HYBNetworking', '~> 2.0.0'
```

或者直接下载源代码，拖入工程使用！

#源代码

请大家到我的`github`下载源代码：[https://github.com/CoderJackyHuang/HYBNetworking](https://github.com/CoderJackyHuang/HYBNetworking)

#温馨提示

最近老有人问：编译一直报错library not found for -lAFNetworking什么问题？

注意：如果您是使用cocoapods来管理第三方库的，那么直接通过上面安装使用的方式来安装即可，然后pod update一下。如果您不是使用cocoapods来引入的，请手动将AFNetworking对应的版本添加到工程。

#关注我


**Swift/ObjC技术群一：[324400294(已满)]()**

**Swift/ObjC技术群二：[494669518]()**

**ObjC/Swift高级群：[461252383（注明年限，新手勿扰）]()**

关注微信公众号：[**iOSDevShares**]()

关注新浪微博账号：[标哥Jacky](http://weibo.com/u/5384637337)

标哥的GITHUB地址：[CoderJackyHuang](https://github.com/CoderJackyHuang)

**原文有更新，请阅读原文：**[http://www.henishuo.com/base-on-afnetworking3-0-wrapper/](http://www.henishuo.com/base-on-afnetworking3-0-wrapper/)

#支持并捐助


如果您觉得文章对您很有帮忙，希望得到您的支持。您的捐肋将会给予我最大的鼓励，感谢您的支持！

支付宝捐助      | 微信捐助
------------- | -------------
![image](http://www.henishuo.com/wp-content/uploads/2015/12/alipay-e1451124478416.jpg) | ![image](http://www.henishuo.com/wp-content/uploads/2015/12/weixin.jpg)
