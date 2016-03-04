#前言

iOS中常用的持久化存储方式有好几种：

* 偏好设置(NSUserDefaults)
* plist文件存储
* 归档
* SQLite3
* Core Data

这里不细讲数据库，只针对性地讲讲文件存储、归档/解档、偏好设置等。

在此之前，我们需要先讲讲沙盒（Sandbox）才能继续讲解。

#沙盒

每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒。沙盒下的目录如下：

* Application：存放程序源文件，上架前经过数字签名，上架后不可修改
* Documents: 保存应⽤运行时生成的需要持久化的数据,iTunes同步设备时会备份该目录。例如,游戏应用可将游戏存档保存在该目录
* tmp: 保存应⽤运行时所需的临时数据,使⽤完毕后再将相应的文件从该目录删除。应用 没有运行时,系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录。
* Library/Caches: 保存应用运行时⽣成的需要持久化的数据,iTunes同步设备时不会备份 该目录。⼀一般存储体积大、不需要备份的非重要数据，比如网络数据缓存存储到Caches下
* Library/Preference: 保存应用的所有偏好设置，如iOS的Settings(设置) 应⽤会在该目录中查找应⽤的设置信息。iTunes同步设备时会备份该目录

#NSUserDefaults

NSUserDefaults是个单例类，用于存储少量数据。NSUserDefaults实际上对plist文件操作的封装，更方便我们直接操作，一般用于存储系统级别的偏好设置。比如我们经常将登录后的用户的一些设置通过NSUserDefaults存储到plist文件中。

有很多App，他们也是将用户的账号和密码存储在偏好设置中。我们不讲安全性问题，因此不讨论存储在偏好设置下是否安全。

使用起来非常简单，如下：

```
// 写入文件
- (void)saveUserName:(NSString *)userName password:(NSString *)password {
  [[NSUserDefaults standardUserDefaults] setObject:userName forKey:@"username"];
  [[NSUserDefaults standardUserDefaults] setObject:password forKey:@"password"];
  [[NSUserDefaults standardUserDefaults] synchronize];
}

// 在用的时候，就可以读取出来使用
NSString *userName = [[NSUserDefaults standardUserDefaults] objectForKey:@"username"];
NSString *password = [[NSUserDefaults standardUserDefaults] objectForKey:@"password"];
```

存储到偏好设置的只有系统已经提供好的类型，比如基本类型、NSNumber、NSDictionary、NSArray等。对于NSObject及继承于NSObject的类型，是不支持的。如下：

```
NSObject *obj = [[NSObject alloc] init];
[[NSUserDefaults standardUserDefaults] setObject:obj forKey:@"obj"];

// 就会崩溃
Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 'Attempt to insert non-property list object <NSObject: 0x7fb502680cb0> for key obj'
```

#plist存储

有的时候，我们需要将下载的数据存储到文件中存储起来，比如，有时候我们将下载起来的城市的数据存储到本地，当更新城市的顺序时，下次也能够按照最后一次操作的顺序来显示出来。

```
// 数据存储,是保存到手机里面,
// Plist存储，就是把某些数据写到plist文件中
// plist存储一般用来存储数组和字典
// Plist存储是苹果特有,只有苹果才能生成plist
// plist不能存储自定义对象，如NSObject、model等
NSDictionary *dict = @{@"age":@"18",@"name":@"USER"};
  
// 保存应用沙盒(app安装到手机上的文件夹)
// Caches文件夹
// 在某个范围内容搜索文件夹的路径
// directory:获取哪个文件夹
// domainMask:在哪个范围下获取 NSUserDomainMask:在用户的范围内搜索
// expandTilde是否展开全路径,YES:展开
NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
NSLog(@"%@",cachePath);
  
// 拼接文件路径
NSString *filePath = [cachePath stringByAppendingPathComponent:@"data.plist"];
  
// 获取应用沙盒
NSString *homePath = NSHomeDirectory();
NSLog(@"%@",homePath);
  
// File:文件全路径 => 所有文件夹路径 + 文件路径
[dict writeToFile:filePath atomically:YES];
// 将数据取出来
NSLog(@"%@", [NSDictionary dictionaryWithContentsOfFile:filePath]);
```

我们看看打印的结果：

```
2016-02-17 22:14:43.055 iOSPersistentStorageDemo[25471:809758] /Users/huangyibiao/Library/Developer/CoreSimulator/Devices/CF3A5A4C-486F-4A72-957B-2AD94BD90EC1/data/Containers/Data/Application/65E8F814-45E5-420C-A174-822A7830748E/Library/Caches
2016-02-17 22:14:43.055 iOSPersistentStorageDemo[25471:809758] /Users/huangyibiao/Library/Developer/CoreSimulator/Devices/CF3A5A4C-486F-4A72-957B-2AD94BD90EC1/data/Containers/Data/Application/65E8F814-45E5-420C-A174-822A7830748E
2016-02-17 22:14:43.056 iOSPersistentStorageDemo[25471:809758] {
    age = 18;
    name = USER;
}
```

注意：操作plist文件时，文件路径一定要是全路径。

#归档(NSKeyedArchiver)

自定义对象应用范围很广，因为它对应着MVC中的Model层，即实体类。在程序中，我们会在Model层定义很多的entity，例如User、Teacher、Person等。

那么对自定义对象的归档显得重要的多，因为很多情况下我们需要在Home键之后保存数据，在程序恢复时重新加载，那么，归档便是一个好的选择。

下面我们自定义一个Person类：

```
// 要使对象可以归档，必须遵守NSCoding协议
@interface Person : NSObject<NSCoding>

@property (nonatomic, assign) int age;
@property (nonatomic, strong) NSString *name;

@end

@implementation Person

// 什么时候调用:只要一个自定义对象归档的时候就会调用
- (void)encodeWithCoder:(NSCoder *)aCoder {
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeInt:self.age forKey:@"age"];
}

- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
       self.name = [aDecoder decodeObjectForKey:@"name"];
       self.age = [aDecoder decodeIntForKey:@"age"];
    }
    return self;
}
@end
```

如何将自定义对象归档和解档：

```
- (void)savePerson { 
	// 归档:plist存储不能存储自定义对象，此时可以使用归档来完成
	Person *person = [[Person alloc] init];
	person.age = 18;
	person.name = @"USER";
	    
	// 获取tmp目录路径
	NSString *tempPath = NSTemporaryDirectory();
	    
	// 拼接文件名
	NSString *filePath = [tempPath stringByAppendingPathComponent:@"person.data"];
	    
	// 归档
	[NSKeyedArchiver archiveRootObject:person toFile:filePath];
}

- (void)readPerson {
	// 获取tmp  
	NSString *tempPath = NSTemporaryDirectory();
	    
	// 拼接文件名
	NSString *filePath = [tempPath stringByAppendingPathComponent:@"person.data"];
	    
	// 解档
	Person *p = [NSKeyedUnarchiver unarchiveObjectWithFile:filePath];
	NSLog(@"%@ %d",p.name,p.age);
}
``` 

假设我们定义了一个自定义的view，这个view是用xib或者storybard来生成的，那么我们我一定如下方法时，就需要如下实现：

```
@implementation CustomView

// 解析xib,storyboard文件时会调用
- (id)initWithCoder:(NSCoder *)aDecoder {
    // 什么时候调用[super initWithCoder:aDecoder]？
    // 只要父类遵守了NSCoding协议,就调用[super initWithCoder:aDecoder]
    if (self = [super initWithCoder:aDecoder]) {
        NSLog(@"%s",__func__);
    }
    
    return  self;
}

@end
```

如果想要学习如何通过runtime来实现自动归档、解档，请阅读文章：[通过runtime自动归档/解档](http://www.henishuo.com/runtime-archive-unarchive-automaticly/)

#最后

关于数据库，这里不讲解了，因为数据库是一个很大方面的知识，不是三言两语所能讲通的。

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


