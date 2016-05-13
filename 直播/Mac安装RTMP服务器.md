#概述

Mac安装RTMP服务器过程记录下来！

#一、安装Homebrew

执行命令：

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果已经安装过，而想要卸载：

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

如果已经安装过，则不用安装！

#二、安装nginx

先glone nginx项目到本地：

```
brew tap homebrew/nginx
```

执行安装：

```
brew install nginx-full --with-rtmp-module
```

通过操作以上步骤，nginx和rtmp模块就安装好了

#三、运行nginx

执行命令：

```
nginx
```

出现如下页面信息，表示nginx服务器搭建成功了，而且已经安装了RTMP模块了:

![image](http://www.henishuo.com/wp-content/uploads/2016/05/QQ20160509-0@2x.png)



#四、配置nginx和rtmp

下面开始来配置nginx的rtmp模块。首先，我们要看看nginx安装到哪里了：

```
brew info nginx-full
```

出现如下类似信息：

```
==> Caveats
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

- Tips -
Run port 80:
 $ sudo chown root:wheel /usr/local/Cellar/nginx-full/1.10.0/bin/nginx
 $ sudo chmod u+s /usr/local/Cellar/nginx-full/1.10.0/bin/nginx
Reload config:
 $ nginx -s reload
Reopen Logfile:
 $ nginx -s reopen
Stop process:
 $ nginx -s stop
Waiting on exit process
 $ nginx -s quit

To have launchd start homebrew/nginx/nginx-full now and restart at login:
  brew services start homebrew/nginx/nginx-full
Or, if you don't want/need a background service you can just run:
  nginx
```

从这些信息中，可以看到nginx.conf文件在：

```
/usr/local/etc/nginx/nginx.conf
```

nginx完整路径：

```
/usr/local/Cellar/nginx-full/1.10.0/bin/nginx
```


通过以下打开nginx.conf配置文件来配置：

```
vi /usr/local/etc/nginx/nginx.conf
```

直接滚动到最后一行，以就是在http {} 之后：

```
http {
    这里默认就有的，不用管这些
}

# 在http节点后面加上rtmp配置：
rtmp {
    server {
        listen 5920;
        application rtmplive {
            live on;
            record off;
        }
    }
}
```

重启nginx：

```
/usr/local/Cellar/nginx-full/1.10.0/bin/nginx -s reload
```


#五、安装ffmpeg

输入以下命令来安装ffmpeg：

```
brew install ffmpeg
```

安装这个需要等一段时间，然后准备一个视频文件作为来推流，我们在安装一个支持rtmp协议的视频播放器，Mac下可以用VLC。

#六、ffmpeg推流

```
ffmpeg -re -i /Users/huangyibiao/Desktop/test.mov -vcodec libx264 -acodec aac -f flv rtmp://localhost:5920/rtmplive/room
```

将视频推流到服务器后，打开VLC，然后file->open network->输入：

```
rtmp://localhost:5920/rtmplive/room
```

观看视频！

#小结

终于安装解决RTMP服务器了，后面可以自己尝试去学习相关技术了！


