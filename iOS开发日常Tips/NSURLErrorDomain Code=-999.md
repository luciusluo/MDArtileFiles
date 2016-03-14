#前言

今天有一个线上bug，是分配给提供H5的团队的，但是后台查不出来原因。于是让前端iOS帮忙查一查原因。

今天，交给我来帮忙查原因，但是问题在网络好的状态下并不必现，很难去定位问题的根本原因。最后只能到测试旁边连接测试专用的慢网环境，然后才能必现。

刚进入界面时是加载一个H5页面，当点击H5页面上的某个按钮的时候，会通过webview拦截到scheme，然后去走接口请求数据，得到数据后再加载H5页面。此时，走了webview加载失败的回调。

加断点，打印出来的error信息是:

```
Error Domain=NSURLErrorDomain Code=-999 “The operation couldn’t be completed.
```

然后，笔者进入NSURLError.h中查看-999代表什么对应的key是什么:

```
NS_ENUM(NSInteger)
{
    NSURLErrorUnknown = 			-1,
    NSURLErrorCancelled = 			-999,
    NSURLErrorBadURL = 				-1000,
    NSURLErrorTimedOut = 			-1001,
    NSURLErrorUnsupportedURL = 			-1002,
    NSURLErrorCannotFindHost = 			-1003,
    NSURLErrorCannotConnectToHost = 		-1004,
    NSURLErrorNetworkConnectionLost = 		-1005,
    NSURLErrorDNSLookupFailed = 		-1006,
    NSURLErrorHTTPTooManyRedirects = 		-1007,
    NSURLErrorResourceUnavailable = 		-1008,
    NSURLErrorNotConnectedToInternet = 		-1009,
    NSURLErrorRedirectToNonExistentLocation = 	-1010,
    NSURLErrorBadServerResponse = 		-1011,
    NSURLErrorUserCancelledAuthentication = 	-1012,
    NSURLErrorUserAuthenticationRequired = 	-1013,
    NSURLErrorZeroByteResource = 		-1014,
    NSURLErrorCannotDecodeRawData =             -1015,
    NSURLErrorCannotDecodeContentData =         -1016,
    NSURLErrorCannotParseResponse =             -1017,
    NSURLErrorAppTransportSecurityRequiresSecureConnection NS_ENUM_AVAILABLE(10_11, 9_0) = -1022,
    NSURLErrorFileDoesNotExist = 		-1100,
    NSURLErrorFileIsDirectory = 		-1101,
    NSURLErrorNoPermissionsToReadFile = 	-1102,
    NSURLErrorDataLengthExceedsMaximum NS_ENUM_AVAILABLE(10_5, 2_0) =	-1103,
    
    // SSL errors
    NSURLErrorSecureConnectionFailed = 		-1200,
    NSURLErrorServerCertificateHasBadDate = 	-1201,
    NSURLErrorServerCertificateUntrusted = 	-1202,
    NSURLErrorServerCertificateHasUnknownRoot = -1203,
    NSURLErrorServerCertificateNotYetValid = 	-1204,
    NSURLErrorClientCertificateRejected = 	-1205,
    NSURLErrorClientCertificateRequired =	-1206,
    NSURLErrorCannotLoadFromNetwork = 		-2000,
    
    // Download and file I/O errors
    NSURLErrorCannotCreateFile = 		-3000,
    NSURLErrorCannotOpenFile = 			-3001,
    NSURLErrorCannotCloseFile = 		-3002,
    NSURLErrorCannotWriteToFile = 		-3003,
    NSURLErrorCannotRemoveFile = 		-3004,
    NSURLErrorCannotMoveFile = 			-3005,
    NSURLErrorDownloadDecodingFailedMidStream = -3006,
    NSURLErrorDownloadDecodingFailedToComplete =-3007,

    NSURLErrorInternationalRoamingOff NS_ENUM_AVAILABLE(10_7, 3_0) =         -1018,
    NSURLErrorCallIsActive NS_ENUM_AVAILABLE(10_7, 3_0) =                    -1019,
    NSURLErrorDataNotAllowed NS_ENUM_AVAILABLE(10_7, 3_0) =                  -1020,
    NSURLErrorRequestBodyStreamExhausted NS_ENUM_AVAILABLE(10_7, 3_0) =      -1021,
    
    NSURLErrorBackgroundSessionRequiresSharedContainer NS_ENUM_AVAILABLE(10_10, 8_0) = -995,
    NSURLErrorBackgroundSessionInUseByAnotherProcess NS_ENUM_AVAILABLE(10_10, 8_0) = -996,
    NSURLErrorBackgroundSessionWasDisconnected NS_ENUM_AVAILABLE(10_10, 8_0)= -997,
};
```
找到了吧！！！NSURLErrorCancelled就是-999，它代表请求被取消的意思。

#根本原因

出现NSURLErrorDomain Code=-999的根本原因是什么呢？其实就是因为webview在之前的请求还没有加载完成，下一个请求发起了，此时webview会取消掉之前的请求，因此会回调到失败这里。

因此，在处理Webview的加载失败的回调时，要注意拦截掉被取消的请求。


#解决方案

在webview加载失败时，添加如下代码来判断：

```
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error {
  [self stopAnimating];
  
  // 如果是被取消，什么也不干
  if([error code] == NSURLErrorCancelled)  {
    return;
  }
  
  // 后续失败处理
}
```

#最后

如果大家遇到同样的问题，请不要着急，这个bug不一定是后台的，也不一定是前端的，因此彼此应该要互相配合，共同找到问题的根本原因。

由于安卓也有同样的问题，但是复现率没有iOS的高，在笔者找出根本原因后，H5人员与安卓端描述，希望安卓端也能统一改，但是描述不清楚，导致安卓这边有意见。于是叫笔者过来帮忙解释原因，讲了半天，安卓的leader说不可能~不会的~不应该~不影响~

发现跟安卓沟通要是不懂一点安卓，真心容易被人忽悠。当然，最后还是要改的，事实都摆在面前了，还能有什么借口可以逃避！

希望大家在遇到同样的问题时，淡定！沟通协作共同解决问题。


