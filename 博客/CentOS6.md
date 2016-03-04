>#阿里云CentOS6.5安装PHP环境

#安装PHP

```
yum -y install php
```

出现`Complete!`时，就说明安装成功了。

使用`rpm -ql php`命令查看已经安装好的`php`组件：

```
rpm -ql php
```

这此就会提示：

```
/etc/httpd/conf.d/php.conf
/usr/lib64/httpd/modules/libphp5.so
/var/lib/php/session
/var/www/icons/php.gif
```

上传文件到服务器:

```
scp /Users/huangyibiao/Downloads/sh-1.3.0.zip  root@101.101.101.101:/home
```

服务器这边解压：

```
unzip sh-1.3.0.zip
```

通过命令安装：

```
./install.sh
```
出现了无权限的问题，在此之前，我们先授权：

```
chmod -R 777 sh-1.3.0/
```

然后再安装。

选择安装的版本：

```
You select the version :
web    : nginx
nginx : 1.4.4
php    : 5.5.7
mysql  : 5.6.15
```

查看FTP和MYSQL的账号和密码：

```
[root@iZ25v5sdfdsf sh-1.3.0]# cat account.log
##########################################################################
#
# thank you for using aliyun virtual machine
#
##########################################################################

FTP:
account:xxx
password:xxxxxx

MySQL:
account:root
password:xxxxxxxx
```