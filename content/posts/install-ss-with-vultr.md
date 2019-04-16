---
title: Vultr 与小飞机
date: 2018-08-17
categories:
- Life
tags:
- Shadowsocks
---

多的不说了，标题已经不言而喻。

<!--more-->

## 什么是shadowsocks?

> **Shadowsocks** 可以指：一种基于 [Socks5](https://zh.wikipedia.org/wiki/SOCKS#SOCK5) 代理方式的加密传输协议，也可以指实现这个协议的各种传输包。 Shadowsocks 分为服务器端和客户端，在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。 
>
> 摘自 [维基百科](https://zh.wikipedia.org/wiki/Shadowsocks)

需要注意的一点是shadowsocks**不是**vpn, 它只是一种基于**SOCKS5协议**的对于网络请求的加密/解密方法, 以达到科学上网的目的.

### shadowsocks原理

一般情况下, 如果在本机上想要访问**外网**(Google/Twitter/Youtube)是直接与远程服务建立连接并传输数据. 但在受限的网络环境下传输的数据会先经过**防火墙(GFW)**的检查, 如果检查出传输内容包含受限内容的话, 就会阻止此次传输, 导致无法获取远程服务数据.

而shadowsocks则是根据**SOCKS5**协议封装的一种数据传输的方法. 

1. 从主机上发出的请求会先经过本地的**ss客户端(ss-local)**, ss客户端会根据配置的加密方法与密码对原数据进行加密, 再将加密过的数据发送给GFW. 
2. 由于数据经过了加密, GFW无法识别出该请求是否受限, 于是通过此次请求. 
3. 按照用户配置的内容, 该被加密的数据会先发送到**境外的ss服务端(ss-server)**, 通过同样的算法解密后得到真正的请求数据, 然后从指定的服务器获取返回数据. 
4. 最后通过类似的过程, 受限网络的返回数据就会被主机接收到了.

### 为什么选择shadowsocks?

> 现在网上这么多vpn服务, 为什么要使用shadowsocks呢?

现在一个能够使用的VPN也比较难找, 毕竟因为众所周知的原因, VPN很容易就被封. 就性价比来说, 买来的VPN也不如自己租的服务器.

最关键的是, 用自己的能力科学上网是非常爽的.

### 准备步骤

现在知道原理了之后, 就明白实现shadowsocks有三个关键点:

- 一台在防火墙之外的服务器
- 本地安装的 shadowsocks 客户端, 用于加密以及传输数据
- 安装在服务器的 shadowsocks 服务端, 用于解密以及数据的中转

就像开头所说的, 这篇教程使用的是 **Vultr** 的服务器, 最便宜的一种是*2.5美元/月*, 500G流量/月, 实际上一般人连100G都用不到. Vultr 的服务器是按**小时计费**的, 只要账户里有钱, 就会每小时自动扣除, **即使关机也会计费**. 另外, Vultr 支持**支付宝**（UPDATE 2019: 现在也支持微信了 ）.

> 点击这个[Vultr链接](https://www.vultr.com/)注册帐号并部署服务器

对了, 本机的系统是 Windows/Mac/Linux 甚至手机都没关系。使用 SSH 连接服务器的话 Windows 推荐安装 [Xshell](https://pc.qq.com/detail/4/detail_2644.html)，Mac 和 Linux 上直接使用**Terminal** 就可以了。

好了, 这就开始吧!

# 部署

## Vultr服务器部署

注册完Vultr帐号了并且向账户内存入足够的钱后就可以开始服务器的部署了. 点击**Deploy now**或者[部署服务器](https://my.vultr.com/deploy/)进入服务器部署页面. 一次对以下内容修改:

1. 选择服务器位置. 这里的位置可以随意, 速度差距不大, 不过建议**不要选择日本**的服务器, 因为虽然日本的服务器速度最快, 但是ip常常被封, 需要经常换ip. 如果想要[测试服务器速度](https://www.vultrvps.com/test-server)的话可以从这个网站测试, 或者ping一下也可以.

2. 选择服务器系统. **CentOS 6+，Debian 7+，Ubuntu 12+**都可以支持的, 这里建议选择 **Ubuntu 18.04 64bits** 的版本, 因为这可以方便之后用**锐速**或者 **BBR** 加速.

   ![选择服务器系统](https://i.loli.net/2018/08/18/5b7829bc76e37.jpg)

3. 选择价位. 值得高兴的是最近(2018.8.19)Vultr推出了全部服务器**$2.5/mo**的价位, 以前的话只有一些特定服务器才会有这价位, 所以机会难得. 这里选择随意, 虽然高价位提升了带宽和流量, 但说实话如果只是单人使用的话**$2.5/mo**就足够了.

   ![优惠](https://i.loli.net/2018/08/18/5b782bcfe904a.jpg)

4. 额外选项. 这里选择第一个**Enable IPv6**就可以了.

   ![Vultr额外选项](https://i.loli.net/2018/08/18/5b782b53ca373.jpg)

5. 其他选项保持默认, 点击右下**Deploy Now**就开始自动部署了.

新建的服务器需要几分钟的时间部署, 这段时间可以用来下载**Xshell**(windows)或者吃瓜, 部署结束后在个人页面就可以看到**Status**显示部署完成. 

点击新建的服务器, 在**Overview**标签下可以看到**IP Address**, **Username**, 以及**Password**,这三行就是之后连接服务器的关键.

![Overview](https://i.loli.net/2018/08/18/5b78320edfa18.png)



## 服务端ss服务搭建

### Windows

#### 连接到服务器

首先需要安装有[Xshell](https://pc.qq.com/detail/4/detail_2644.html), 可以通过链接或百度搜索下载.

1. 打开*File-New(Alt+N)*

![Xshell新建服务器设置](https://i.loli.net/2018/08/19/5b797bce861fc.jpg)

2. 如图设置服务器的信息, **Name**是服务器的名字, 可以随便填. **Host**填写在Vultr注册的服务器的**IP Address**, 其他选项保持默认就行. 

   ![Xshell服务器IP](https://i.loli.net/2018/08/19/5b797cc506ef6.jpg)

3. 接着设置用户名和密码, 用户名和密码分别是在 Vultr 上的 **Username** 和 **Password**, 填写完后勾上记住用户和密码就行了.

4. 如果如下图所示, 则表明已经连接成功了. 注意连接的服务器必须是已经部署好的. 如果没有连接成功的话就多试几次, 还不行的话就ping一下ip, 看一下是不是被墙了. 如果ping不上的话就重新再部署一个服务器吧.

   ![Xshell连接成功](https://i.loli.net/2018/08/19/5b79819f3cb5c.jpg)

#### 搭建ss服务

1. 下载一键搭建ss服务脚本, 直接复制粘贴就行了.

   ```shell
   git clone https://github.com/flyzy2005/ss-fly
   ```

2. 运行搭建ss代码

   ```shell
   ss-fly/ss-fly.sh -i yourpassword 1024
   ```

   这里把**yourpassword**替换为你想要设置的密码就行了, 随意设定, 以后使用ss客户端就用这个密码. 后面的**1024**是服务的端口号, 默认是1024.

3. 等待一会后出现成功提示就可以了.

4. 启动服务:

   ```shell
   /etc/init.d/ss-fly start
   ```

   其他相关操作:

   ```shell
   启动：/etc/init.d/ss-fly start
   停止：/etc/init.d/ss-fly stop
   重启：/etc/init.d/ss-fly restart
   状态：/etc/init.d/ss-fly status
    
   修改配置文件：vim /etc/shadowsocks.json
   ```

ss服务启动之后一般情况下不需要调整服务端了, 服务器常开就行.

#### ss客户端

shadowsocks的官方client地址是[shadowsocks clients](https://shadowsocks.org/en/download/clients.html), 但似乎是被墙了. windows的GUI Client可以从Github上下载.

- [shadowsocks-win](https://github.com/shadowsocks/shadowsocks-windows/releases)
- [shadowsocks-Qt5](https://github.com/shadowsocks/shadowsocks-windows/releases)

下载后解压启动shadowsocks, 会在右下角显示一个![](https://i.loli.net/2018/08/19/5b7989881e4b4.jpg)的图标,右击即可打开选项.  服务器添加窗口打开后填写服务器相关数据即可.

![客户端设置](https://i.loli.net/2018/08/19/5b798aa7c597f.jpg)

服务器添加成功后, 右键图标选择启动服务即可启动shadowsocks, 其他选项例如开机自启动,自动更新之类的看自己需求.

# 参考文章

[[一键脚本搭建SS/搭建SSR服务并开启BBR加速 | flyzy小站](https://www.flyzy2005.win/fan-qiang/shadowsocks/install-shadowsocks-in-one-command/)]

