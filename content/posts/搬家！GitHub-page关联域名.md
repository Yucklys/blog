---
title: 搬家！GitHub Pages 关联域名
date: 2018-11-05
categories:
- Life
- Blog
tags:
- GitHub
toc: true
---

最近有时间上 [Godaddy](https://www.godaddy.com/) 购进了一个新域名，[yucklys.com](https://yucklys.com/)，一番折腾之后总算是摆脱了 github.io ，特此做一个记录。

<!--more-->

## 服务选择

由于不想去备案，并且实际上这个博客也没多少人看，所以选择了**狗爹**的域名。狗爹的`.com`域名还是很良心的，因为我们的https通过GitHub就可以开启，所以不用额外购买服务，而且还不需要实名认证，所以相对起来还是选择了狗爹。

具体域名怎么购买我就不赘述了，点击上方Godaddy的链接即可进入主界面，注册帐号，搜索域名，找到心仪的域名就收入囊中吧！

> 喝一杯咖啡，休息一下:coffee:。

一切都准备就绪了的话，开始搬家吧！

## 搬家

### GitHub CNAME配置

首先打开博客在GitHub上的地址，比如https://github.com/Yucklys/yucklys.github.io。在setting-GitHub Pages选项下面可以看到如下的提示：

> Your site is published at <https://yourname.github.io/>

这表明你的博客现在在yourname.github.io域名下运行着呢，现在我们需要做的就是设置一个cname记录把我们的新域名，yourname.com，添加进去。

在GitHub Pages相关选项中有一个**Custom Domain**选项，填写我们的域名，然后点击旁边的**Save**。如果一切顺利的话，在刷新后GitHub Pages下方的提示应该变成了：

> Your site is published at http://yourname.com/

### Godaddy域名解析

现在让我们转移到我们的域名控制台上进行域名的解析操作。

![domain_name.png](https://i.loli.net/2018/11/06/5be10e1559b33.png)

在Godaddy的域名管理台下需要修改的只有两处：

1. 修改A记录，记录值改为GitHub Page的IP地址。

   注意，如果想给自己的博客加上小绿锁的话，这里的IP地址是**GitHu Page的HTTPS地址**。总共有四个IP地址可选，随便选择一个就好了。[官方文档](https://help.github.com/articles/setting-up-an-apex-domain/)已经给出了颇为详细的介绍。四个IP如下：

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

2. 添加一个CNAME记录，把yourname.github.io添加进去。

   这一步没什么需要注意的，如图填写即可。

好了，现在输入你的域名，应该就可以访问你放在GitHub上的博客内容了。

### 开启HTTPS

眼尖的同学就发现了，现在我们的网站只开启了http，如果是用chrome浏览的话还会被标上恼人的「不安全」。要想开启Https的话其实很简单，眼尖的同学可能也发现了，在第一步修改GitHub Page的设置的时候，有一个选项叫做**Enforce HTTPS**。

![Enforce HTTPS](https://blog.github.com/assets/img/2018-05-01-github-pages-custom-domains-enforce-https.png)

只需要简单地选中选项，页面刷新后就可以看到上方的提示变为：

> Your site is published at https://yourname.com/

但是也有可能发生一些莫名的报错，我的HTTPS无法正常使用，经过多方调查，我在GitHub文档上看到了这样一句话：![resolve http.png](https://i.loli.net/2018/11/06/5be114e1ef53e.png)

然后我先删除了我的**Custom Domain**设置，保存，然后再重新添加，保存，然后**Enforce HTTPS**就可以使用了。不是很清楚是什么原理，不过这样做确实是解决了我的问题。

等待一段时间后，就可以看到自己的Https博客了:smile: 。

### 本地配置

但是需要注意的是，GitHub Page的CNAME选项只是在项目文件夹下新建了一个CNAME(无后缀)文件，如果在本地的git项目中没有这个文件的话，下一次deploy就会把这个文件删除，导致我们需要重新配置一遍。所以在本地上的源文件内需要也新建一个CNAME文件，内容仿照GitHub中生成的CNAME文件就行。

```
domainname.com
```

