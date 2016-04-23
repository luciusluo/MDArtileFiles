#前言


相信很多朋友都使用了友盟统计这个SDK吧，能够统计出我们崩溃的日志，但是反馈的日志是无法确定到底是哪里发生崩溃的，那么我们如何去查呢？

dYSM是打包的时候生成的，位于/Users/<用户名>/Library/Developer/Xcode/Archives下，找到它就可以拿友盟统计上的错误日志来查找崩溃在程序的哪个类哪行代码了。不过，这不是绝对的，有的日志是查不到崩溃在何处的。

#查找dYSM文件


在友盟统计上，错误日志这里会有应用的版本号，我们要根据这个版本号，找到我们对应的`ipa`包，然后找到`dYSM`文件。在错误日志的下位，有出错的版本号，出错的次数，出错的首次日期，最后一次出现的日期。

如下：

```
[huangyibiao@huangyibiaodeMacBook-Pro ~] $ cd /Users/huangyibiao/Library/Developer/Xcode/Archives/
[huangyibiao@huangyibiaodeMacBook-Pro ~/Library/Developer/Xcode/Archives] $ ls
2015-11-09	2015-11-27
[huangyibiao@huangyibiaodeMacBook-Pro ~/Library/Developer/Xcode/Archives] $ cd 2015-11-27/
[huangyibiao@huangyibiaodeMacBook-Pro ~/Library/Developer/Xcode/Archives/2015-11-27] $ ls
appName 15-11-27 上午11.02.xcarchive
```

这个是路径，改成自己电脑的用户名：/Users/huangyibiao/Library/Developer/Xcode/Archives/。最后，我们就是需要找到这个：appName 15-11-27 上午11.02.xcarchive

得到了这个东西，接下来就好办了！

#学习分析日志

如下为友盟统计的一个错误日志：

```
*** -[__NSPlaceholderDictionary initWithObjects:forKeys:count:]: attempt to insert nil object from objects[8]
(null)
(
	0   CoreFoundation                      0x000000018268654c  + 160
	1   libobjc.A.dylib                     0x000000019365c0e4 objc_exception_throw + 60
	2   CoreFoundation                      0x00000001825748e8  + 420
	3   CoreFoundation                      0x0000000182574718  + 72
	4   appName                             0x10062d8f8 appName + 6478072
	5   appName                             0x10062ab34 appName + 6466356
	6   appName                             0x100621344 appName + 6427460
	7   appName                             0x1001d2bd4 appName + 1911764
	8   UIKit                               0x0000000186ec8a14  + 96
	9   UIKit                               0x0000000186eb1d08  + 612
	10  UIKit                               0x0000000186ec83b0  + 592
	11  UIKit                               0x0000000186ec803c  + 700
	12  UIKit                               0x0000000186ec1590  + 684
	13  UIKit                               0x0000000186e94e60  + 264
	14  UIKit                               0x000000018713446c  + 15220
	15  UIKit                               0x0000000186e933d0  + 1716
	16  CoreFoundation                      0x000000018263ed34  + 24
	17  CoreFoundation                      0x000000018263dfd8  + 264
	18  CoreFoundation                      0x000000018263c088  + 712
	19  CoreFoundation                      0x00000001825691f4 CFRunLoopRunSpecific + 396
	20  GraphicsServices                    0x000000018b98b6fc GSEventRunModal + 168
	21  UIKit                               0x0000000186efa10c UIApplicationMain + 1488
	22  appName                             0x1003dcb3c appName + 4049724
	23  libdyld.dylib                       0x0000000193cdaa08  + 4
)

dSYM UUID: 67BB446B-A9F3-35A0-9A3D-9BCC3C55F9B9
CPU Type: arm64
Slide Address: 0x0000000100000000
Binary Image: appName
Base Address: 0x00000001000e4000
```

单看友盟日志，是看不出来到底在哪里崩溃的。不过日志中有了崩溃的地址，我们可以通过命令查出来到底是哪个类哪一行哪一列出现的崩溃。

我们点击上面的错误日志中第一个不是颜色比较特殊的一个地址，这里是`0x10062d8f8`，点击它就出弹出类似下面这样的：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-17-下午5.41.29.png)

看到这一行：`dwarfdump --arch=arm64 --lookup 0x10062d8f8`了吗，这就是用于查找地址`0x10062d8f8`错误信息的地址，然后就是指定我们对应的`dYSM`文件的路径：

加上`dYSM`路径后，形成这样的：

```
dwarfdump --arch=arm64 --lookup 0x1003dcb3c /Users/huangyibiao/Desktop/dSYMs/3.2.2\(10482\) appName\ 15-11-27\ 19.16.xcarchive/dSYMs/appName/Contents/Resources/DWARF/appName
```

**注意**：`appName`为自己应用的名称

然后一回车，我们得到类似如下的信息：

```
Looking up address: 0x00000001005234e4 in .debug_info... found!

0x0082c2d5: Compile Unit: length = 0x00002416  version = 0x0002  abbr_offset = 0x00000000  addr_size = 0x08  (next CU at 0x0082e6ef)

0x0082c2e0: TAG_compile_unit [100] *
             AT_producer( "Apple LLVM version 7.0.0 (clang-700.0.72)" )
             AT_language( DW_LANG_ObjC )
             AT_name( "../../HYBTestDemo/HYBTest/HYBTestController.m" )
             AT_stmt_list( 0x0028dd53 )
             AT_comp_dir( "/Users/huangyibiao/HYBDemo/HYBTest/" )
             AT_APPLE_optimized( 0x01 )
             AT_APPLE_major_runtime_vers( 0x02 )
             AT_low_pc( 0x00000001005211b4 )
             AT_high_pc( 0x0000000100524114 )

0x0082dada:     TAG_subprogram [126] *
                 AT_low_pc( 0x00000001005231b8 )
                 AT_high_pc( 0x0000000100523570 )
                 AT_frame_base( reg29 )
                 AT_object_pointer( {0x0082dafa} )
                 AT_name( "-[HDFDemo post:parameters:success:failure:]" )
                 AT_decl_file( "/Users/huangyibiao/HYBTestDemo/HYBTestController.m" )
                 AT_decl_line( 330 )
                 AT_prototyped( 0x01 )
                 AT_APPLE_optimized( 0x01 )
Line table dir : '/Users/huangyibiao/HYBTestDemo/HYBTest/HYBTestController.m'
Line table file: 'HYBTestController.m' line 369, column 1 with start address 0x00000001005234e4
```

上面的类名、方法名、项目名，笔者随便改了一下，只是不想让大家知道而已。不过这生成的就是类似这样子的。

然后，我们可以分析得到是在类`HYBTestController`中的第`369`行第`1`列崩溃的，就可以直接定位到错误的地方，然后分析出错的代码，`fix`掉就可以了。

#通常无法定位错误的日志

像这种日志：

```
Application received signal SIGSEGV
(null)
(
	0   CoreFoundation                      0x0000000186e8a5b8  + 160
	1   libobjc.A.dylib                     0x00000001975e40e4 objc_exception_throw + 60
	2   CoreFoundation                      0x0000000186e8a4dc  + 0
	3   appName                             0x100869714 _ZN15CTXAppidConvert17IsConnectionAppIdEPKc + 1712168
	4   libsystem_platform.dylib            0x0000000197e0094c _sigtramp + 52
	5   CoreFoundation                      0x0000000186e30ae4  + 20
	6   CoreFoundation                      0x0000000186d6f220 _CFXNotificationPost + 2060
	7   Foundation                          0x0000000187c6ecc0  + 72
	8   UIKit                               0x000000018b8ca7e4  + 1132
	9   UIKit                               0x000000018b8d2cbc  + 92
	10  UIKit                               0x000000018b8d2c44  + 380
	11  UIKit                               0x000000018b8c6578  + 512
	12  FrontBoardServices                  0x000000018f0f962c  + 28
	13  CoreFoundation                      0x0000000186e42a28  + 20
	14  CoreFoundation                      0x0000000186e41b30  + 312
	15  CoreFoundation                      0x0000000186e40154  + 1756
	16  CoreFoundation                      0x0000000186d6d0a4 CFRunLoopRunSpecific + 396
	17  GraphicsServices                    0x000000018ff0f5a4 GSEventRunModal + 168
	18  UIKit                               0x000000018b6a23c0 UIApplicationMain + 1488
	19  appName                             0x1003dcb3c newPatient + 4049724
	20  libdyld.dylib                       0x0000000197c52a08  + 4
)

dSYM UUID: 67BB446B-A9F3-35A0-9A3D-9BCC3C55F9B9
CPU Type: arm64
Slide Address: 0x0000000100000000
Binary Image: appName
Base Address: 0x00000001000b0000
```

通常是分析不出来的，通过地址是无法查找出来的。像这种类型的`bug`目前还没有很好的解决方案。像这种错误，分析出来是这样的：

```
Looking up address: 0x0000000100869714 in .debug_info... not found.
Looking up address: 0x0000000100869714 in .debug_frame... not found.
```

查找地址失败，因此无法定位具体是哪里有问题，也就无法解决了。

#结尾

其实通过友盟所收集到的bug不都是可解的。有的时候可能是友盟自身所产生的，但是那不是bug，而是友盟的处理事件。

当然，有的时候可能是我们所使用的第三方SDK所引起的，比如百度地图等，对于类似这样的问题，通过的解决办法就是尝试升级SDK。如果SDK已经是最新，那么只能通过反馈到对应的平台了！




