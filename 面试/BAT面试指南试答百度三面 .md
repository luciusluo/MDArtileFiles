#概述

本篇文章是继[BAT面试指南试答百度一面](http://101.200.209.244/bat-interview-first/)和[BAT面试指南试答百度二面](http://101.200.209.244/bat-interview-baidu-second/)部分而试答，一面中只写了算法部分，将一面中的iOS部分移到二面中写，本篇完全是手写算法的三面题。

**郑重声明：**题目来源于[BAT 面试指南](https://bestswifter.com/bat-interview/)百度三面手写算法题。

#三面手写算法题

下面是随手想而写下的算法，效率可能不高，也可能会出错，大家可以根据自己的想法评论！

##3.1 给一个字符串，如何判断它是否是合法的IP地址，比如 "192.168.1.1" 就是合法的。

首先，我们需要明确一个合法的IP格式应该是怎么样的，每个值是0-255，因此，我们可以通过字符串分割后，分别判断值是否在0-255即可。

###3.1.1 Objective-C实现

```
/**
 *	Judge whether the specified string is a valid form of IP or not.
 *
 *	@param ipString	The specified string to be checked.
 *
 *	@return YES if is a valid form of IP, otherwise NO.
 */
- (BOOL)isValidIP:(NSString *)ipString {
  if (ipString.length <= 0) {
    return NO;
  }
  
  NSArray *tempArray = [ipString componentsSeparatedByString:@"."];
  if (tempArray.count != 4) {
    return NO;
  }
  
  __block BOOL isValid = YES;
  [tempArray enumerateObjectsUsingBlock:^(NSString *_Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    if (obj.integerValue < 0 || obj.integerValue > 255) {
      isValid = NO;
      *stop = YES;
    }
  }];
  
  return isValid;
}
```

###3.1.2 C语言实现

```
/**
 *	Input：a string，judge whether it is a valid form of IP string
 *
 *	@param ipString
 *
 *	@return 1 if it is a valid form of IP string, otherwise 0
 */
int isValidIP(char *ipString) {
  int len = (int)strlen(ipString);
  
  if (len < 7 || len > 15) {
    return 0;
  }
  
  int itemValue = 0;
  int index = 1;
  for (int i = len - 1; i >= 0; --i) {
    char ch = ipString[i];
    
    if (ch >= '0' && ch <= '9') {
      itemValue += (ch - '0') * pow(10, index - 1);
      index++;
    } else if (ch == '.') {
      index = 1;
      if (itemValue < 0 || itemValue > 255) {
        return 0;
      }
      
      itemValue = 0;
    } else {
      return 0;
    }
  }
  
  if (itemValue < 0 || itemValue > 255) {
    return 0;
  }

  return 1;
}
```

C语言实现的话，就需要遍历了。我们是从后往前遍历的，就可以一将遍历判断完成。

##3.2 说说大数相加的思路，动手写代码实现。

大学的时候搞ACM就已经实现过大数相加了，也写过算法。大数相加的关键点是通过字符串来实现相加，以串最长的作为基准，将串短的高位补0，然后对位相加，并做好进位处理。

大学的时候已经研究过这个算法了，因此现在手写起来还是挺顺的。记得最后还可能有进位哦，需要将进位处理一下。我们这里是先低位相加，不断往高位方向移动做加法，借助临时数组，将计算结果逆序存放。大数相加和相减都是挺简单的，大数相乘就难度大多一点点。

```
/**
 *	做两个超大数相加算法，采用0补高位的方式再做加法运算
 *
 *	@param lhsSource	大数1
 *	@param rhsSource	大数2
 *	@param result		接收结果，确保长度足够
 */
void addBigNumbers(char *lhsSource, char *rhsSource, char *result) {
  int lhsLen = (int)strlen(lhsSource);
  int rhsLen = (int)strlen(rhsSource);
  int len = lhsLen > rhsLen ? lhsLen : rhsLen;
  
  char *temp = malloc(sizeof(char *) * (len + 2));
  int i = lhsLen - 1;
  int j = rhsLen - 1;
  int k = 0;
  char lhsChar = '0';
  char rhsChar = '0';
  
  // 进位
  int carryBit = 0;
  int z = 0;
  
  while (i >= 0 || j >= 0) {
    // 串短的就以'0'补位，用于做加法运算
    if (i < 0) {
      lhsChar = '0';
    } else {
      lhsChar = lhsSource[i];
    }
    
    // 串短的就以'0'补位，用于做加法运算
    if (j < 0) {
      rhsChar = '0';
    } else {
      rhsChar = rhsSource[j];
    }
    
    // 对位相加，再加上进位值
    z = lhsChar - '0' + rhsChar - '0' + carryBit;
    // 有可能>=10，需要取做进位处理
    temp[k++] = z % 10 + '0';
    // 更新进位
    carryBit = z / 10;
    
    i--;
    j--;
  }
  
  // 全部相加完之后，有可能还有进位，需要将进位顶到高位
  while (carryBit > 0) {
    temp[k++] = carryBit % 10 + '0';
    carryBit /= 10;
  }
  
  // 我们借助了临时字符数组来存储计算结果，但是计算结果是倒序的，
  // 我们需要将计算结果变成正序
  k--;
  i = 0;
  while (k >= 0) {
    result[i++] = temp[k--];
  }
  
  // 别忘了添加上字符串结束标记符
  result[i] = '\0';
  
  // temp是自己在堆上申请的内存，记得释放
  free(temp);
}
```

随手写两个字符串相加：

```
char *lhsSource = "12368102369126318236218391231231232132132";
char *rhsSource = "9999999999999991232399999999999999999999999";
char result[100];
addBigNumbers(lhsSource, rhsSource, result);
NSLog(@"result = %s", result);

// print
// result = 10012368102369117550636218391231231232132131
```

##3.3 简述TCP建立和关闭连接时，握手的过程。为什么前者是三次握手，后者需要四次？

以下是百度了一下相关文章，在阅读完之后才明白为什么后者需要4次。其实，我想大家跟笔者一样，若没有搞过TCP相关的项目，怎么会记得那么多，只是记得大学教科书里说过的三次握手。

TCP建立连接时，握手的过程大概如下：

* 客户端发送SYN到服务端
* 服务端发布SYN/ACK到客户端，此时开始建立连接
* 客户端发布ACK到服务端，此时正式建立好连接

客户端发送SYN到服务端，而服务端返回了客户端发过来的SYN，同时也返回ACK，那么客户端接收到之后，就可以确定服务端收到了SYN信号，而客户端接收到服务端返回来的ACK信号后，再将ACK信号发送到服务端，服务端就明确客户端收到了服务端发过去的信号。因此，这三次握手就可以确定了双方的身份。

TCP关闭连接时，握手的过程大致如下：

* 客户端发送FIN包到服务端：此时客户端进入FIN_WAIT_1等待对方确认状态
* 服务端返回ACK包到客户端：此时客户端结束FIN_WAIT_1状态，并进入FIN_WAIT_2状态，等待服务端的发过来的关闭请求
* 服务端发送FIN包到客户端：此时服务端进入CLOSE_WAIT状态，等待客户端确认关闭请求
* 客户端返回ACK包到服务端：此时服务端正式关闭，结束CLOSE_WAIT状夶

TCP关闭连接之所以需要四次握手，是因为TCP连接是全双工，是双向的。

更详细地部分，大家可以阅读[TCP连接建立、关闭的握手过程](http://blog.chinaunix.net/uid-25018796-id-94900.html)这篇文章。

##3.4 假设有10W条电话号码，如何通过输入电话号码的某一段内容，快速搜索出来。比如输入234，以下两个号码都会显示在搜索结果中：

```
123456789000
188888823400
```

其实最简单的解决方案是遍历所有字符串，然后用KMP算法。但是这样的问题是需要遍历 10W 个元素，效率比较低。我想到的是办法是使用索引。建立100个索引（00 到 99），比如输入 234 时只需要在索引23对应的区域查找即可，可以加快100倍速度。但是缺点是插入数据时，需要更新多个索引，数据量会是原来的10倍。

我能想到的也是建立索引，几乎所有大批量数据中查找信息，应该都需要建立索引的。索引如何建立才能查找快，这是也需要分析的。我想公司里面的数据，也是建立好索引来检索的，至于更新、插入操作，应该不会马上去更新索引吧，可以通过定时脚本来更新索引。比如夜黑风高的时候再跑一下脚本更新一下索引库~

##3.5 OC的数组中，添加nil对象会有什么问题。

这个问题自己问过自己，平时开发中就遇到过不少崩溃是由于插入nil对象引起的。其实字典也是一样的，插入nil对象都会引起崩溃。

比如，我们下面这么写就会引起崩溃：

```
NSMutableArray *array = [[NSMutableArray alloc] init];

//  -[__NSArrayM insertObject:atIndex:]: object cannot be nil'
[array addObject:nil];
```

这是很常见的崩溃打印信息吧？但是，如果我们在初始化时，通过下面的API来添加nil，是不会有事的，只是表示结束而已。

```
NSArray *array = [[NSArray alloc] initWithObjects:@"sss", nil, @"sfsdf"];
// 结果只有sss，后面的因为中间有nil而被过滤了
NSLog(@"%@", array);
```

#小结

百度三面题目到此结束，所答不代表一定正确，所写的算法也不确定足够健壮。好好准备一番，但愿能有机会去百度面一面！

对于3.4题，大家有什么好的算法解决吗？可以在评论中指出哦！