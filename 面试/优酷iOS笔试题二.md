#前言

收到某某同学将去youku的iOS笔试题的邮件，希望笔者能整理一下，并提供参考答案。笔者决定整理出来，并分享给大家。当然，与此同时，也想看看youku的笔试题到底有多难，也考考自己有多少料吧！

本篇为第二篇，第一篇已经发布出来，请大家阅读：[YouKu iOS笔试题一](http://www.henishuo.com/youku-interview-one/)

第二篇明显题目简单太多了，不知道优酷是不是真的笔试题就这么简单呢？5分钟之内差不多就可以做完了吧！

如果所提供的参考答案有任何值得置疑的地方，请一定要在评论中指出！

#前5题

![image](http://www.henishuo.com/wp-content/uploads/2016/03/ios2.jpg)

#1、这条语句的输出

```
NSLog(@"%.4f", 3.1415926);
```

A. 3.14  
B. 3.141  
C. 3.1415  
D. 3.14159  

**参考答案：应该是3.1416**

%.4f表示保留小数点后四位，因为3.1415926小数点后第5位是9，要进1，因此应该是3.1416.

但是参考答案中并没有提供这样的选项。

#2、请问下面的array对象的retainCount是多少

```
@property (nonatomic, retain) NSArray *array; 

self.array = [[NSArray alloc] init];
```

A. 1  
B. 2  
C. 3  
D. 4  

**参考答案：A**

如果没有上下文的话，只有self引用了array，其引用计数应该为1.

#3、请问下面的test对象的retainCount是多少

```
NSString *string = [[NSString alloc] initWithString:@"abc"];
NSString *test = string;
```

A. 1    
B. 2  
C. 3  
D. 4  

**参考答案：B**

从这里看，string指向了字符串@"abc"所在的内存区，而test指针指向了string指针，因此会对string所指向的内存区的引用计数加1，因此retainCount为2

#4、NSInteger count=100,以下哪条语句没有错误

```
A. NSArray *array = [NSArray arrayWithObject:count];
B. NSString *str = [NSString stringWithFormat:@"%@", count];
C. NSMutableArray *array = [NSMutableArray arrayWithCapacity:count];
D. NSString *str = [count stringValue];
```

**参考答案：C**

A中要求传对象；B中%@应该修改为%ld；D中count不是对象，没有方法！

#5、以下代码没有内存泄露的是

```
A. - (NSMutableArray *)getNameList() {
   NSMutableArray *list = [[NSMutableArray alloc] init];
   [list addObject:@"name1"];
   [list addObject:@"name2"];
   
   return list;
}

B. @property (nonatomic, retain) NSArray *list;
   
   list = [[NSArray alloc] init];
   self.list = nil;
   
C. 全局变量NSString *name;
   - (void)setName:(NSString *)string {
     name = [string retain];
   }

D. UIButton *button = [[UIButton alloc] init];
   [self.view addSubview:button];
```

**参考答案：B**

首先，这道题讲的是MRC环境下的。

A中在函数内部创建的数组，在MRC下函数返回了一个非自动释放对象，因此得不到释放；  
B中看起来题目有点问题，笔者觉得应该是self.list而不是list，不过只有B是正确的;  
C中name是全局的对象，在重新赋值之前，并没有先释放之前的内存；  
D中创建了button，但是并没有加上autorelease，所以放在self.view上后，就引用计数为2，当view释放时，使其引用计数为1，但是不会降到0，所以释放不了。

#后3题

![image](http://www.henishuo.com/wp-content/uploads/2016/03/ios3.jpg)

#6、@property(nonatomic) NSInterger age;以下代码有什么问题？

```
- (void)setAge:(int)_age {
   self.age = _age;
}
```

A. 没有问题  
B. 死循环  
C. 内存泄漏  
D. 不能用int类型做参数

**参考答案：B**

这是重写setter方法，然后在setter方法里调用setter方法，形成了死循环。

#7、下面程序的输出是

```
main() {
  int a = -1, b = 4, k;
  
  k = (a++<=0) && (!(b--<=0));
  printf("%d%d%d\n", k, a, b);
}
```

A. 003  
B. 012  
C. 103  
D. 112

**参考答案：C**

k = (a++ <= 0) && (!(b-- <= 0)); => a++表达式的值为-1，它是后增，所以表达式的值是-1；

由于a++ == -1，而-1 <= 0为真，因此继续看后面的表达式了：

但是a++是执行了的，所以a变成了0。

! (b-- <= 0) => b == 3, 这个表达式的值为真，所以k的值为1

所以，k == 1, a == 0, b == 3

#8、设有：int a=1, b=2, c=3, d=4, m=2, n=2;执行(m=a>b)&&(n=c>d)后n的值是多少

A. 1  
B. 2  
C. 3  
D. 4  

**参考答案：B**

(m = a > b) 的结果是m = 1 > 2 =>m = 0，由于&&（逻辑与）在前面的条件为假时，就会短路了，不会再继续往下执行判断，所以后面的表达式（n=c>d）是不会执行的，所以n的值不变。

#最后

我想说这真的是优酷的iOS笔试题吗？难道是应届生的？真的简单得不能再简单了！所以大家不要害怕，其实都很容易的！


#推荐阅读

* 【[iOS面试宝典](http://www.henishuo.com/ios-interview-entrance/)】