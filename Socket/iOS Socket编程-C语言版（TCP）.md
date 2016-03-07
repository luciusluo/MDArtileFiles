#序言

本篇文章为总结使用`C`语言的`api`来完成`TCP`通信的基本功能，如果您对`Socket`不了解，请先阅读上一篇理论知识：

* [iOS Socket理论知识](http://www.henishuo.com/ios-socket-theory/)

如果您还想学习`UDP`编程，请阅读：

* [iOS Socket编程-C语言版(UDP)](http://www.henishuo.com/ios-socket-udp-c-version/)

如果文章中有任何您认为不正确的或者有疑问的，请联系笔者！

#1. TCP Socket编程


`TCP`是面向连接的，安全可靠的传输层协议。`TCP`的程序基本框架设计图：

![image](http://www.coderyi.com//qiniu/429/image/5269612b25e6df1b3ee5ab8352b2c3b6.jpg)

**注意：**`Socket`通信一定有要服务端和客户端。

##1.1 TCP Socket客户端

客户端的工作流程：首先调用`socket`函数创建一个`Socket`，然后指定服务端的`IP`地址和端口号，就可以调用`sendto`将字符串传送给服务器端，并可以调用`recvfrom`接收服务器端返回的字符串，最后关闭该socket。

笔者这里分成了六步：

*   第一步：创建`socket`并配置`socket`
*   第二步：调用`bind`绑定监听ip和端口号
*   第三步：调用`connect`连接服务器
*   第四步：调用`getsockname`获取套接字信息
*   第五步：调用`send`发送消息到服务器端
*   第六步：调用`close`关闭`socket`

这里没有写接收来自服务器端的消息，大家可以自行添加。

###1.1.1 客户端的代码实现：

```
- (void)tcpClient {
  // 第一步：创建soket
  // TCP是基于数据流的，因此参数二使用SOCK_STREAM
  int error = -1;
  int clientSocketId = socket(AF_INET, SOCK_STREAM, 0);
  BOOL success = (clientSocketId != -1);
  struct sockaddr_in addr;
  
  // 第二步：绑定端口号
  if (success) {
    NSLog(@"client socket create success");
    // 初始化
    memset(&addr, 0, sizeof(addr));
    addr.sin_len = sizeof(addr);
    
    // 指定协议簇为AF_INET，比如TCP/UDP等
    addr.sin_family = AF_INET;
    
    // 监听任何ip地址
    addr.sin_addr.s_addr = INADDR_ANY;
    error = bind(clientSocketId, (const struct sockaddr *)&addr, sizeof(addr));
    success = (error == 0);
  }
  
  if (success) {
    // p2p
    struct sockaddr_in peerAddr;
    memset(&peerAddr, 0, sizeof(peerAddr));
    peerAddr.sin_len = sizeof(peerAddr);
    peerAddr.sin_family = AF_INET;
    peerAddr.sin_port = htons(1024);
    
    // 指定服务端的ip地址，测试时，修改成对应自己服务器的ip
    peerAddr.sin_addr.s_addr = inet_addr("192.168.1.107");
    
    socklen_t addrLen;
    addrLen = sizeof(peerAddr);
    NSLog(@"will be connecting");
    
    // 第三步：连接服务器
    error = connect(clientSocketId, (struct sockaddr *)&peerAddr, addrLen);
    success = (error == 0);
    
    if (success) {
      // 第四步：获取套接字信息
      error = getsockname(clientSocketId, (struct sockaddr *)&addr, &addrLen);
      success = (error == 0);
      
      if (success) {
        NSLog(@"client connect success, local address:%s,port:%d",
              inet_ntoa(addr.sin_addr),
              ntohs(addr.sin_port));
        
        // 这里只发送10次
        int count = 10;
        do {
          // 第五步：发送消息到服务端
          send(clientSocketId, "哈哈，server您好！", 1024, 0);
          count--;
          
          // 告诉server，客户端退出了
          if (count == 0) {
            send(clientSocketId, "exit", 1024, 0);
          }
        } while (count >= 1);
        
        // 第六步：关闭套接字
        close(clientSocketId);
      }
    } else {
      NSLog(@"connect failed");
      
      // 第六步：关闭套接字
      close(clientSocketId);
    }
  }
}
```

###1.1.2 客户端的打印日志

```
2015-12-06 18:35:00.385 iOS-Socket-C-Version-Client[9726:4256295] client socket create success
2015-12-06 18:35:00.386 iOS-Socket-C-Version-Client[9726:4256295] will be connecting
2015-12-06 18:35:00.507 iOS-Socket-C-Version-Client[9726:4256295] client connect success, local address:192.168.1.100,port:50311
```

说明连接服务器成功，然后发送了消息到服务器端。

##1.2 TCP Socket服务器端

服务器端的工作流程：首先调用`socket`函数创建一个套接字，然后调用`bind`函数将其与本机地址以及一个本地端口号绑定，接收到一个客户端时，服务器显示该客户端的IP地址，并将字串返回给客户端。

笔者这里分成了五步：

* 第一步：创建`socket`并配置`socket`
* 第二步：调用`bind`绑定服务器本机ip及端口号
* 第三步：调用`listen`监听客户端的连接，并指定同时最多可让`accept`的数量 
* 第四步：调用`accept`等待客户端的连接
* 第五步：调用`recvfrom`接收来自客户端的消息
* 第六步：调用`close`关闭`socket`

###1.2.1 服务器端代码实现

```
- (void)tcpServer {
  // 第一步：创建socket
  int error = -1;
  
  // 创建socket套接字
  int serverSocketId = socket(AF_INET, SOCK_STREAM, 0);
  // 判断创建socket是否成功
  BOOL success = (serverSocketId != -1);
  
  // 第二步：绑定端口号
  if (success) {
    NSLog(@"server socket create success");
    // Socket address
    struct sockaddr_in addr;
    
    // 初始化全置为0
    memset(&addr, 0, sizeof(addr));
    
    // 指定socket地址长度
    addr.sin_len = sizeof(addr);
    
    // 指定网络协议，比如这里使用的是TCP/UDP则指定为AF_INET
    addr.sin_family = AF_INET;
    
    // 指定端口号
    addr.sin_port = htons(1024);
    
    // 指定监听的ip，指定为INADDR_ANY时，表示监听所有的ip
    addr.sin_addr.s_addr = INADDR_ANY;
    
    // 绑定套接字
    error = bind(serverSocketId, (const struct sockaddr *)&addr, sizeof(addr));
    success = (error == 0);
  }
  
  // 第三步：监听
  if (success) {
    NSLog(@"bind server socket success");
    error = listen(serverSocketId, 5);
    success = (error == 0);
  }
  
  if (success) {
    NSLog(@"listen server socket success");
    
    while (true) {
      // p2p
      struct sockaddr_in peerAddr;
      int peerSocketId;
      socklen_t addrLen = sizeof(peerAddr);
      
      // 第四步：等待客户端连接
      // 服务器端等待从编号为serverSocketId的Socket上接收客户连接请求
      peerSocketId = accept(serverSocketId, (struct sockaddr *)&peerAddr, &addrLen);
      success = (peerSocketId != -1);
      
      if (success) {
        NSLog(@"accept server socket success,remote address:%s,port:%d",
              inet_ntoa(peerAddr.sin_addr),
              ntohs(peerAddr.sin_port));
        char buf[1024];
        size_t len = sizeof(buf);
        
        // 第五步：接收来自客户端的信息
        // 当客户端输入exit时才退出
        do {
          // 接收来自客户端的信息
          recv(peerSocketId, buf, len, 0);
          if (strlen(buf) != 0) {
            NSString *str = [NSString stringWithCString:buf encoding:NSUTF8StringEncoding];
            if (str.length >= 1) {
              NSLog(@"received message from client：%@",str);
            }
          }
        } while (strcmp(buf, "exit") != 0);
        
        NSLog(@"收到exit信号，本次socket通信完毕");
        
        // 第六步：关闭socket
        close(peerSocketId);
      }
    }
  }
}
```

###1.2.2 服务器端的打印日志

```
2015-12-06 18:34:31.258 iOS-Socket-C-Version-Server[39929:2622200] server socket create success
2015-12-06 18:34:31.258 iOS-Socket-C-Version-Server[39929:2622200] bind server socket success
2015-12-06 18:34:31.259 iOS-Socket-C-Version-Server[39929:2622200] listen server socket success
2015-12-06 18:35:00.743 iOS-Socket-C-Version-Server[39929:2622200] accept server socket success,remote address:192.168.1.100,port:50311
2015-12-06 18:35:00.743 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.743 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.743 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.744 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.745 iOS-Socket-C-Version-Server[39929:2622200] received message from client：哈哈，server您好！
2015-12-06 18:35:00.745 iOS-Socket-C-Version-Server[39929:2622200] received message from client：exit
2015-12-06 18:35:00.745 iOS-Socket-C-Version-Server[39929:2622200] 收到exit信号，本次socket通信完毕
```

我们这里打印出了客户端发来的消息，由于上面实现的代码中，只发10次，所以这里只有10条。

#源代码

小伙伴们，可以到github下载了：[https://github.com/CoderJackyHuang/iOS-Socket-C-Version](https://github.com/CoderJackyHuang/iOS-Socket-C-Version)

**注意：**这里面有两个工程，一个是客户端，一个是服务器端。运行时，先运行服务器端，然后再选择客户端。另外，客户端所指定的服务器端的ip地址一定要修改成您本机对应的ip，不然使用笔者这里的ip就会失败。

