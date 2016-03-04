>#Wordpress网站添加百度联盟验证

---
初次使用`wordpress`，不知道怎么添加百度联盟验证。在这里，写下如何通过百度联盟验证的过程：

#第一步：注册账号

---

首先，我们需要注册百度联盟账号：[注册百度联盟账号](https://www.baidu.com/s?ie=UTF-8&wd=%E7%99%BE%E5%BA%A6%E8%81%94%E7%9B%9F)

首次注册成功后，必须先验证我们的网站才行。

#第二步：域名已备案

---
如果您的域名没有备案，那是无法难过验证的，因此，我们首先要有一个已备案的域名。如果您的域名没有备案，可先备案。如果您不想自己去备案，可以通过爱亿科技购买已备案的域名。本人就是购买的已备案的域名。

然后通过[www.chaicp.com](www.chaicp.com)查询我们的域名的备案信息。

#第三步：添加验证代码

---
我们需要在网站后台找到`header.php`添加验证的代码。也就是在`外观=>编辑`找到`header.php`，然后编辑这个文件。

在`header.php`文件中，找到这行代码：

```
<meta name="viewport" content="width=device-width, initial-scale=1">
```

然后在其下面添加百度联盟验证代码：

```
<meta name="baidu_union_verify" content="03e1a039854add5c1def826ff054869f">
```

添加后，文件中变成这样：

```
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="baidu_union_verify" content="03e1a039854add5c1def826ff054869f">
```

然后保存文件，在保存成功后，等待一分钟左右，就可以在百度联盟网站点击[完成验证]按钮来验证，第一次可能会不通过，再点就通过了！！！

>###注意：不要直接使用我这里的`03e1a039854add5c1def826ff054869f`，一定要使用对应生成给你的代码！！！

