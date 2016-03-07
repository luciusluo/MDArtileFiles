>#CentOS上安装git服务器

#引言

本人非后台开发人员，但是也学了一些后台开发知识，一直很好奇什么是`git`服务器。本人毕业以来一直从事`ios`开发，但是在业余时间也学习了`ThinkPHP`框架、`HTML5`、`CSS3`、`JavaScript`，做过网站，因此可以说前后端都有所了解了，可从来没有搞过服务器。

本文就是首次学习如何在阿里云上搭建`git`服务器时所写下的记录，包括遇到的一些问题。这个过程也折腾了我不少时间，通过百度搜索出来的文章中都是只讲大致过程，没有具体到所有步骤，导致按照其文章说明去搭建时出现了不少问题，最终也没有折腾出来。因此，本人在此写下最简单的搭建`git`的方式，不过每次访问`git`仓库时，都需要输入密码。虽然如此，但是这是最简单的搭建方式，对于某些朋友、同学来说已经可以了，因为对于那些不是专业搞服务器的人员，真的很难搞明白。

我们不专业？一样可以搭建出来！！！

>注意：本文中只记录如何安装`git`，以及创建`git`仓库，并可以通过正常的`ssh`访问到。对于`git`权限分配，这里不说明，因为这篇文章的写作目的是帮助像我这样的非专业人士简单地搭建出来。下一篇文章，我们再介绍如何通过`gitosis`来管理`git`权限。

#环境

公网IP地址：`101.101.101.101`

操作系统： `CentOS 6.5 64位`

>提示：不同的操作系统，其操作命令不都是一样的，因此本文记录的只是在`CentOS 6.5 64位`操作系统上搭建`git`服务器的过程。对于其它操作系统，请参考他人的博文。

#前提条件

在进行下一步之前，我们必须要准备好相关条件。

* 首先，我们需要懂得如何连接远程终端：在阿里云管理控制台中，找到云服务器ECS，然后找到实例，看到实例的右边有一个**更多**，点击它就可以弹出相关操作选项，然后点击连接远程终端。
* 其次，在首次进入连接远程终端时，会提示一个密码，这时候一定要记住。当然，如果没记住，也没有关系，我也没有记录下来，所以就查如何修改密码。修改密码：在阿里云管理控制台中，找到云服务器ECS，然后找到实例，下边有一个`重置密码`，点击它就可以修改密码了，这个是登录操作系统的密码。
* 记住连接管理终端的密码，如果忘记了，没有关系，弹出的窗口中下方有一个修改终端密码。修改好就可以了。
* 进入到管理终端后，登录的用户为`root`，密码就是第二步中所记录下来的密码。这个密码要求必须有大写字母、小写字母和数字组成，要求8个长度以上。一定要记住了，以后每次登录都需要到此密码。

#第一步：安装openssh

登录阿里云远程终端，登录成功后，在终端中输入以下命令来安装openssh服务器与客户端工具：

```
sudo yum install openssh-server openssh-client 
```

>提示：可以直接复制命令，然后在远程终端右上角有一个***复制命令输入***，我们点击它，然后将我们复制的命令粘贴，点击就可以执行。

#第二步：安装git

通过下面的命令来安装`git`：

```
sudo yum install git-core
```

#第二步：安装python工具

```
sudo yum install python-setuptools 
```

#第三步：添加用户

在安装完`git`后，我们需要添加一个用户：

```
sudo useradd git
sudo passwd git
```
其中第一条命令是添加用户`git`，注意这里的`git`是用户，不是文件夹。第二条命令是给`git`用户设置密码，这个一定要记住后。

可以不使用`sudo`，若提示`Permission denied`，再使用`sudo`也是可以的。

>##提示：
>如果操作过程感觉不对，想要删除该用户重新操作，看下面的方案：

Centos删除用户`git`时候提示：

```
userdel: user git is currently logged in.
```

当提示当前正处于登录状态时，先执行下面的命令，再重启服务器：

```
mv /var/run/utmp /var/run/utmp_touch > /var/run/utmp
```

重启服务器后，再输入以下命令就可以删除了，其中`git`就是刚才我们创建的用户：

```
sudo userdel -r git
```

#第四步：配置git仓库

这里，我们切换到用户`git`：

```
su git
```

然后就可以创建仓库，我们创建`repository`作为仓库，以后所有的git项目都放到这个仓库中：

```
cd /home
mkdir git
cd git
mkdir repository  
```
这里共有四条命令，一步步执行。从此以后所有源码的`root`都在`/home/root/repository`目录下了。

下面需要给它权限。如果在用户`git`下不能操作，就切换到`root`用户下用`sudo`操作。切换用户为：`su root`，然后输入密码就可以切换了。

```
chown git:git /home/git/repository 
chmod 755 /home/git/repository 
```

接下来，全局配置git用户信息，设置值时，修改成自己的值：

```
git config --global user.name "huangyibiao(需替换)"   
git config --global user.email "huangyibiao@163.com(需替换)" 
```


#第五步：创建仓库
下面我们创建一个空的仓库，并初始化，然后测试是否成功。

```
mkdir /home/git/repository/testproject
git init --bare testproject
```
这里的`testproject`是一个目录，也就是我们的仓库。先创建目录，然后初始化仓库。执行该命令成功后`/home/git/repository/`下生成了一个`testproject`目录，该目录里面只有一个`.git`文件夹。

外部测试一下是否能正确访问，在我的用于开发的电脑上，通过终端`terminal`访问：

```
git clone git@101.101.101.101:/home/git/repository/testproject
```
然后在终端打印出下面的信息，就说明成功地`clone`下来了。注意，这里项目的路径在服务器这边是`/home/git/repository/testproject`，一定要写全了。

```
Cloning into 'testproject'...
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), 55.35 KiB | 0 bytes/s, done.
Checking connectivity... done.
```

然后我们随便放一个文件到这个目录下，然后提交：

```
cd testproject/

// 查看添加文件是否有状态变化
git status

git add .
git commit -m "提交一个测试文件"
git push origin master
```

当我们`push`后，出下面的信息就说明成功了：

```
git push origin master
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 21.97 KiB | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@101.101.101.101:/home/git/repository/testproject
   b84af6c..4867d46  master -> master
```

#终端连接阿里云
每次都到阿里云实例连接远程终端是不是很麻烦？我们可以不用到网站操作，直接在我们本机的终端就可以直接连接远程，并操作。

首先，其端口是`22`，因此我们在本机需要先修改一下本机的端口：

编辑ssh_config文件，找到`Port`，将`Port 5680`改成`Port 22`。注意，可能公司所使用的git的端口号为`5680`，在操作公司用的项目时，再执行这里的步骤，然后修改端口为`5680`

```
// 然后输入本机的密码，这里是使用管理员权限
sudo -s

vi /etc/ssh/ssh_config

// 然后在终端保存、退出
:wq

```

下面我们连接远程服务器：

```
ssh git@101.101.101.101
```

其中，`101.101.101.101`是本人的服务器的公网`ip`，这里只是15免费云，所以拿来练手的。`git`是用户，我们在前面的步骤中添加过`git`用户。这一步骤，会先寻找公钥，如果没有，则提示输入密码，输入密码即可登录。不过登录的是`git`用户，并非是`root`用户。要切换到`root`用户，通过`su root`然后输入密码即可。

#生成rsa密/公钥
每次连接都需要输入密码，是否觉得很麻烦？我们可以通过添加公钥、密钥来处理。

```
ssh-keygen -t rsa
```
上面是生成rsa密码，这时会提示保存到哪里。如果您还没有在本机添加过任何的rsa，那么使用默认路径即可。如果原来已经有了`id_rsa`和`id_rsa.pub`文件怎么办？我们在保存时，输入别的路径：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/huangyibiao/.ssh/id_rsa):
```

默认路径为：`/Users/用户名/.ssh/id_rsa`。由于本机已经存在了，我们换一下名称：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/huangyibiao/.ssh/id_rsa):

/Users/huangyibiao/.ssh/id_rsa_test
```

此时会提示输入密码，这个密码一定要记住，这里输入的是：`123456`。
那么就生成了两个文件，一个是`id_rsa_test`私钥，另一个是`id_rsa_test.pub`公钥。

接下来，我们还需要将这个公钥上传到服务器这边：

```
scp /Users/huangyibiao/.ssh/id_rsa_test.pub root@101.101.101.101:/home/git/
```

上传成功后，在本机操作:

```
vi /Users/huangyibiao/.ssh/config
```

然后添加以下内容，以便在校验时，能找到我们密钥：

```
host 101.101.101.101
IdentityFile /Users/huangyibiao/.ssh/id_rsa_test
```

如果服务端这边需要配置多个公钥，我们可以这么做：将所有的公钥都放到authorized_keys文件中。

```
cat /home/git/id_dsa.pub >> /home/git/.ssh/authorized_keys 
```
如果没有`.ssh`目录，就先创建`mkdir .ssh`。如果没有`authorized_keys`，就先创建。

现在我们在别的机器上访问：

```
ssh git@101.101.101.101
```

在`mac`电脑上就出现了访问钥匙串的提示，要求将我们生成这个私钥/公钥时所指定的`phrase`输入，这个就是其密码。以后每次执行上面的命令连接服务器，就会自动去读取钥匙串，然后自动连接了。

如果我们使用`root`权限登录，还是要求输入`root`用户的密码的。

```
ssh 101.101.101.101

root@101.101.101.101's password:
Last login: Fri Nov  6 23:57:45 2015 from 106.39.222.47

Welcome to aliyun Elastic Compute Service!

[root@iZ25v54cz28Z ~]#
```

如果我们指定为`git`用户登录：

```
 $ ssh git@101.101.101.101
 
Last login: Sat Nov  7 10:04:27 2015 from 106.39.222.47

Welcome to aliyun Elastic Compute Service!
```

不再需要输入密码了，直接就可以登录了。
