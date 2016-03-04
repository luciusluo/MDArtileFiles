#前言

不知道大家有没有遇到这样的问题：原来我的`xcode7`是可以正常使用插件的，可以当我升级为`xcode7.1`以后，所有的插件都失效了，而且`Alcatraz`插件管理器也失败了，在`xcode`的菜单栏`window`上也没有显示`Package Manager`了。因此找了很多办法想要将插件安装回来，不然开发效率明显要降低！！！

#第一步


首先，我们先到这个目录，查看`Alcatraz.xcplugin`是否存在：

```
~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins
```

查看方式：`Finder->前往->填写上面的路径`

#第二步


如果已经存在，执行以下命令：

```
rm -rf ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/Alcatraz.xcplugin
rm -rf ~/Library/Application\ Support/Alcatraz
```

这个操作是先将之前的清空掉，以便一会重新安装！当然也可以直接在目录这里右键选择删除！

#第三步

		
执行下面的命令安装插件：

```
curl -fsSL https://raw.githubusercontent.com/supermarin/Alcatraz/deploy/Scripts/install.sh | sh
```

安装时可能会出现提示安装成功，但是前面还有一句话，大概意思就是说连接失败。

如果出现这种类似的情况，就需要重新执行上面的安装命令，直到安装成功。

安装成功的验证：

```
cd ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/
ls 
```

如果有了`Alcatraz.xcplugin`，那就说明成功了。前提是安装过程没有提示失败，这才表示成功。如果安装过程有提示失败信息，而这个插件包又存在，那么安装是不成功的，就需要重新移除，再次执行命令安装。

#无效？

**Xcode升级后插件失效解决方案：**

这里有一个脚本可以刷新所有的插件，下载[插件脚本](https://github.com/cikelengfeng/RPAXU)，按照文档说明运行脚本即可。亲测可用！！！

