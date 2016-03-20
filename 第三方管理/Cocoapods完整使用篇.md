
#前言


对于iOS App的开发，几乎都采用了Cocoapods来管理第三方库，那么对于我们开发人员来说，这是必备技能，必须要掌握如何使用。这篇文章就是介绍如何安装和使用CocoaPods的。

这篇文章对哪些人群参考价值？

* 对未使用过Cocoapods的人群有参考价值
* 对使用过Cocoapods，但是未深入了解过的用户有参考价值
* 对有开源精神的，希望将自己的代码贡献到Cocoapods的用户有参考价值

如果您不属于以上人群，您是可以不阅读本篇文章的，当然阅读完也会有很大的帮助。

**温馨提示**：在篇文章中所使用的Xcode版本为Xcode7.

#什么是CocoaPods？

简单来说，就是专门为iOS工程提供对第三方库的依赖的管理工具，通过CocoaPods，我们可以单独管理每个第三方库，可以更方便地管理每个第三方库的版本，而且不需要我们做太多的配置，直接交由提供支持CocoaPods项目的作者来配置了，如此便可直观、集中和自动化地管理我们项目的第三方库。

##为什么需要使用CocoaPods？

我们也许有过这样的感受：
每添加一个**第三方库、Framework或者SDK**，我们都需要手动添加相关依赖库，在工程`buildsetting`中配置路径，在build phases中添加依赖的系统库。如果所导入的第三方库还依赖其他第三方库，我们也需要手动导入且分别添加工程配置。

当我们需要更新某个第三方库的时候，我们又要手动移除该库，导入新的库，然后再配置，这是相当麻烦且没有意义的工作。当使用CocoaPods管理后，我们只需要修改为某个版本，再执行pod update即可。
       
当我们需要去掉某个第三方库时，我们是怎么做的呢？是不是将该库移除掉，然后还得把相关配置也移除掉，这样工作才干净。是不是很麻烦呢？当我们使用Cocoapods管理后，我们是怎么做的？只需要在Podfile删除该引入该库的语句，然后执行pod update即可。

当我们开始使用CocoaPods管理第三方库后，我们只需要相当少的配置，其它的一切都交由CocoaPods来管理即可，我们使用起来就更省心了。


##如何安装CocoaPods？

CocoaPods is built with Ruby and is installable with the default Ruby available on OS X. We recommend you use the default ruby.

也就是说CocoaPods是通过Ruby来安装的，MAC OSX都有一个默认的Ruby版本，推荐我们通过默认的Ruby来安装CocoaPods。

使用下面的命令安装：

```
sudo gem install cocoapods
```

事实上，这样安装未必能安装成功，因为默认的cocoapods网址是国外的，需要VPN才能访问，因此我们可以改一种方式：

* 1、先输入 `gem sources --remove https://rubygems.org/`
* 2、等待有反映后，再输入 `gem source -a https://ruby.taobao.org/`  
* 3、验证是否成功替换：`gem source -l`  
* 4、最后就可以通过`sudo gem install cocoapods`正常安装`cocoapods`了。

等待安装完成后，就可以开始使用`CocoaPods`了。

**注意：**`source`或者`sources`都可以.


#如何使用CocoaPods？

要使用`CocoaPods`，就需要一个`Podfile`文件。我们是如何为所有的工程建立`Podfile`的，下面的方式是基本的方式。

```
cd Desktop/Demos/KVODEMO
touch Podfile
vi Podfile
```

* 第一步：进入到我们所建立的工程的目录，这里是`KVODEMO`
* 第二步：通过`touch`命令新建`Podfile`
* 第三步：通过`vi Podfile`进入编辑`Podfile`
* 第四步：添加第三方库，如下图，我们添加了`AFNetworking`和`ObjectiveSugar`库，其中我们添加的`AFNetworking`版本是2.0版本，`ObjectiveSugar`版本是0.5.

```
pod 'AFNetworking', '~> 2.0'  
pod 'ObjectiveSugar', '~> 0.5'
```

* 按下`esc`键，然后输入`:wq`，就可以保存了。然后在终端输入`pod install`，就可以安装第三方库了。

在安装完成后，我们不再是打开后缀为`.xcodeproj`的工程，而是打开后缀为`.xcworkspace`的工作空间了。

关于Podfile更高级的使用，请参考[**官方文档**](https://guides.cocoapods.org/using/the-podfile.html)

或者关注后续文章！

#在Objective-C工程中的使用

在工程中，我们只需要通过引入改文件就可以直接使用了，比如我们引入了第三方库`Masonry`（纯代码自动布局），我们在`Objective-C`工程中就可以通过`import`头文件即可。

```
#import <Masonry.h>
```

注意，如果这么做提示找不到头文件，那么我们可以尝试这么引入：
`#import "Masonry.h"`或者通过`#import "Masonry/Masonry.h"`

如果仍然没有效果，那么需要在工程配置一下.在工程的`Build Settings`搜索`Search Paths`，然后在`User header search paths`中添加`$(SRCROOT)`并选择`recursive`（也就是递归查找）


#在Swift工程中的使用


我相信大家在`Swift`工程中使用`CocoaPods`也遇到了不少问题，尤其是如何`import`模块问题。
当初我遇到这种问题时，也在网上搜索了很多的资料，但是都不是我希望的方案。在网上有两种方式：
通过Swift工程可以桥接`Objectice-C`的方式，建立一个`Bridge-head.h`（名字随便起），然后进入到`Build Settings`，在搜索框中输入`bridg`，找到`Objective-C Bridging Header`，选项，把头文件的路径赋值给该选项。如下所示：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-09-上午9.50.08.png)

也就是：工程名/桥接文件名.h。在刚才所建立的桥接文件中，通过`#import "头文件.h"`就可以了。


虽然是`Objective-C`第三方库，事实上我们也可以使用`Swift`的方式引入的，也就是通过 `import 模块名` 的方式来引入。所以对于上面的方式，我是不喜欢的。那么再看看网上的另一种方式：[Swift第三方管理](http://blog.shiqichan.com/How-To-Import-3rd-Lib-Into-Swift-Project/)

当然，现在swift出了一个[Package Manager](https://swift.org/package-manager/)，专门管理第三方引用的。

这是通过`submodule`的方式来管理的。
创建`submodule`,在当前项目的同级目录下执行类似这样的命令，如下：

```
git submodule add https://github.com/Masonry.git
```

然后将生成的`Masonry.xcodeproj`拖入到工程中。
在`xcode`工程的`general`中，点击`embeded libraries`中的+号，然后改我们的第三方库`framework`，类似下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-09-上午9.54.40.png)

最后就可以直接在工程中需要使用的地方，通过`import`模块名来使用了。
如果是多人团队开发，我们就需要共享了，那么其他成员就需要通过下面的命令来安装：

```
git submodule update --init --recursive
```

这是通过递归来安装或者更新`submodule`。这是挺麻烦的一件事。然后根据使用的经验，我们下载别人的工程下来后，执行了上面的命令，安装好相关模块了，运行工程经常出现报错的问题，也就是配置问题。因此，个人很不喜欢这种方式。

事实上，在Xcode7是不再需要这么做了，对于其他Xcode版本是否需要，未验证。
我们通过cocoapods安装的第三方库会自动生成为framework，然后我们只需要在使用的地方直接通过import 模块名使用即可。但是有时候可能会出现某个第三方库直接通过import 模块名时，提示找不到，也就是没有智能提示。这时候我们可以通过在xcode工程的general中的embeded libraries点击+，然后导入该framework，就可以正常import了。另外如果导入的第三方库在运行时，报错了，类似于：

```
dyld: Library not loaded: @rpath/ReactiveCocoa.framework/ReactiveCocoa 
Referenced from: /private/var/mobile/Containers/Bundle/Application/31ABC86A-C1BD-40DD-A117-D2C8F79A98FE/SwiftGithubClient.app/SwiftGithubClient Reason: image not found
```

**那么我们可以这么解决：**

在`Build Phases->Link Binary With Libraries`->找到出错的库的名称->修改`required`为`optional`即可。

#如何升级CocoaPods版本？

升级CocoaPods是非常简单的，只需要一个命令即可。
正常情况下，只需要一个命令就可以升级了：

```
sudo gem install cocoapods
```

但是有可能需要更新`gem`才能升级`cocoapods`，因此我们可能需要这么做：

```
$ sudo gem update --system // 先更新gem，国内需要切换源
$ gem sources --remove https://rubygems.org/
$ gem sources -a http://ruby.taobao.org/
$ gem sources -l
    CURRENT SOURCES
    http://ruby.taobao.org/
$ sudo gem install cocoapods // 安装cocoapods
$ pod setup
```

然后查看版本号：

```
$ pod --version
0.39.0
```

#如何让自己的开源项目支持CocoaPods？

这里就介绍我写的一个三方库`HYBMasonryAutoCellHeight`让其支持`Cocoapods`的步骤。

* 1、第一步：打开终端并进入到工程的目录。这里是我所开源的一个`HYBMasonryAutoCellHeight`开源库，用于自动计算`cell`的高度。

```
cd HYBMasonryAutoCellHeight
```

* 2、第二步：创建一个`Podspec`文件。通过命令`pod spec create`创建,

```
pod spec create HYBMasonryAutoCellHeight
```

* 3、第三步：编辑该`podspec`文件:

```
VI HYBMasonryAutoCellHeight.podspec
```

大家会看到，这自动生成的已经是一个模板了，我们需要做的就是修改相关内容，补充内容就可以满足我们的需求了。对于更高级的使用，就需要参考官方文档，一步步地学习使用了。
看看我所提供的第三方库的`podspec`文件的配置，去掉注释后看一看最关键的配置：

```
Pod::Spec.new do |s|

s.name         = "HYBMasonryAutoCellHeight"
s.version      = "0.0.1"
s.summary      = "基于Masonry的自动计算cell的行高的扩展库"
s.homepage     = "http://hybios.lianliankeji.com/cocoapods-support/"
s.license      = "MIT"
s.author       = { "huangyibiao" => "huangyibiao520@163.com" }
s.platform     = :ios, "6.0"
s.source       = { :git => "https://github.com/632840804、HYBMasonryAutoCellHeight.git", :tag => "0.0.1" }
s.source_files  = "HYBMasonryAutoCellHeight", "*.{h,m}"
s.requires_arc = true
s.dependency "Masonry", "~> 0.6"
```

这一步很关键，需要配置的东西比较多。主要是`s.source`这里必须提供我们的`git`路径、指定`tag`号和`s.source_files`这里必须指定库的源代码文件。通常我们会把我们的库放到与`Demo`工程的`.xcodeproj`同级，比较这里的`HYBMasonryAutoCellHeight`目录是与`.xcodeproj`是同级的父级目录，然后我们就可以更简单的设置了。如下图：

![image](http://www.henishuo.com/wp-content/uploads/2015/12/屏幕快照-2015-12-09-上午10.09.47.png)

然后我们在设置`s.source_files`时，第一个就设置为`HYBMansonryAutoCellHeight`，第二个就是设置为所有的`.{h,m}`类型的文件。对于比较复杂的设置，可以参考`AFNetworking`中的`podspec`文件。

**如果我们的开源库依赖系统库怎么办？**

```
# s.framework = 'SomeFramework'// 去掉#，设置依赖的系统库名称
# s.frameworks = 'SomeFramework', 'AnotherFramework'//设置多个系统库名称

# s.library = 'iconv'// 设置只依赖一个系统的library
# s.libraries = 'iconv', 'xml2' // 设置依赖多个系统的library

# s.xcconfig = { 'HEADER_SEARCH_PATHS' => '$(SDKROOT)/usr/include/libxml2' }// 这里是工程配置，这样使用者就不需要手动处理，由pod自动处理了。
```

**如果的开源库依赖其他第三方库，怎么办：**

```
s.dependency 'JSONKit', '~> 1.4'//设置我们的开源库依赖哪些第三方库和依赖的版本号。
```

* 4、第四步：这里呢，我们设置的版本号为`0.0.1`，那么`tag`号为`0.0.1`，因此我们还需要新建一个`tag`,名为`0.0.1`，然后推到`git`：

```
$ git commit -m "如果当前有变化，先提交到git上，再创建tag"
$ git tag 0.0.1
$ git push --tags
$ git push origin master
```
第四步可以不用。

接下来，我们需要验证我们的配置是否正确：

* 5、当然我们还可以直接使用:`pod lib lint`来验证所有，最后一个是设置允许警告:

```
pod lib lint HYBMasonryAutoCellHeight.podspec --allow-warnings
```

如果验证通过，会出现这样的提示：

```
-> HYBMasonryAutoCellHeight (0.0.1)

HYBMasonryAutoCellHeight passed validation.
```

如果刚才我们的配置出错，或者需要修改一些内容，重新再推，则可以先删除远程的`tag 0.0.1`，然后在修改后，重复上面的第四步。下面是删除远程`tag`命令：

```
git push origin :refs/tags/0.0.1
```

6、第五步：在验证通过后，提交到`cocoapods`。也就是通过命令`pod trunk push` 库名`.podspec`来推送到远程的`cocoapods`：

```
pod trunk push HYBMasonryAutoCellHeight.podspec --allow-warnings
```

如果出现警告，需要修改相关内容以去掉警告。当操作成功后，我们就可以通过`pod search`命令来搜索我们的库了。

```
$ pod search HYBMasonryAutoCellHeight

-> HYBMasonryAutoCellHeight (0.0.1)
   基于Masonry的自动计算cell的行高的扩展库
   pod 'HYBMasonryAutoCellHeight', '~> 0.0.1'
   - Homepage: http://hybios.lianliankeji.com/cocoapods-support/
   - Source:   https://github.com/632840804/HYBMasonryAutoCellHeight.git
   - Versions: 0.0.1 [master repo]
```

更详细地使用，请参考[官方文档](https://guides.cocoapods.org/making/making-a-cocoapod.html)

这里有一篇文章，写得相当不错：[支持pod](http://www.exiatian.com/cocoapods%E5%AE%89%E8%A3%85%E4%BD%BF%E7%94%A8%E5%8F%8A%E9%85%8D%E7%BD%AE%E7%A7%81%E6%9C%89%E5%BA%93/)

#如何生成Cocoapods私有库？

要让自己的库变成私有库，那么我们的代码也是需要通过`git`来管理的。
这里假设我们新建了一个工程，名叫`BookEffect`，然后已经`push`到`git`远程服务器。

* 第一步：按照第六节一样，创建好`podspec`文件，并将整个工程推送到`git`服务器这边。
* 第二步：这才是引入私有库的方式。`pod 'DemoLib',:git=>"http://xxxxx.git"`(替换为真实的git地址)

具体不细说，请参考：[生成私有pod](http://www.cnblogs.com/superhappy/p/3468377.html)

#使用Cocoapods打包静态库

这里就不细说，不过推荐一篇文章：[打包表态库](http://www.cnblogs.com/brycezhang/p/4117180.html)，写得很不错，如果需要，可以参考。

最后，感谢各位认真阅读本篇文章，感谢您的支持。

#参考

* [官方文档](https://guides.cocoapods.org/making/index.html)
