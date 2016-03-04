#前言

Mac OSX 内置了Apache和PHP，这样使用起来非常方便。本文以笔者Mac OSX 10.11.2为例，主要内容包括：

* 配置环境
* 启动Apache
* 运行PHP
* 重启Apache

#配置环境

1. 打开“系统设置偏好（System Preferences）” 
2. “共享（Sharing）” 
3. “互联网共享（Web Sharing）勾上

在终端中运行：

```
sudo vi /etc/apache2/httpd.conf
```

打开Apache的配置文件，找到

```
#LoadModule php5_module libexec/apache2/libphp5.so
```

把前面的#号去掉，保存（在命令行输入:w）并退出vi（在命令行输入:q）。

运行以下命令来配置PHP功能：

```
// 复制一份
sudo cp /etc/php.ini.default /etc/php.ini

// 然后编辑
vi /etc/php.ini
```

我们可以设置文件上传大小限制等，如下是其中的一小部分：

```
81 ; Whether to allow HTTP file uploads.
782 ; http://php.net/file-uploads
783 file_uploads = On
784
785 ; Temporary directory for HTTP uploaded files (will use system default if not
786 ; specified).
787 ; http://php.net/upload-tmp-dir
788 ;upload_tmp_dir =
789
790 ; Maximum allowed size for uploaded files.
791 ; http://php.net/upload-max-filesize
792 upload_max_filesize = 2M
793
794 ; Maximum number of files that can be uploaded via a single request
795 max_file_uploads = 20
```

#启动Apache

我们不说Apache是什么东西，笔者也不清楚，笔者写这篇文章时，也是刚准备学习，写文章的目的不只是分享，更重要的是给自己留下学习的笔记。

其实，笔者只知道Apache是服务器，与nginx都是服务器。

启动服务器的命令：

```
sudo apachectl start
```

然后输入电脑的密码授予权限，就可以启动了。

查看Apache的版本信息：

```
apachectl －v
```

如果要授权，请先加sudo。笔者当前的版本信息为：

```
Server version: Apache/2.4.16 (Unix)
Server built:   Jul 31 2015 15:53:26
```

#重启Apache

重新启动服务器的命令：

```
sudo apachectl restart
```

#运行PHP

Apache的默认根目录是在/Library/WebServer/Documents/，通过以下命令进入查看：

```
cd /Library/WebServer/Documents/
ls
```

以后，我们就可以直接通过：http://localhost/ 来访问根目录的文件了。当然，如果配置了HOST，也可以通过访问：http://127.0.0.1/ 来访问根目录的文件。

现在，我们直接在浏览器访问：http://localhost，是否出现了"It Works!"？是的话，那么就可以成功地访问了。

#写在最后

本文只是笔者学习PHP的笔记！当然也欢迎高手留言，求指导速成方法。倘若大家觉得对自己也有用，欢迎大家一起来学习PHP，一起讨论PHP语法知识，一起练习PHP。

#关注我

**PHP学习交流群：536643087**

**阅读原文，原文有更新：**[http://www.henishuo.com/mac-php-config/](http://www.henishuo.com/mac-php-config/)




