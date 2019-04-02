---
title: Hexo + github 搭建自己的个人博客
date: 2018-06-29
categories:
- Life
- Blog
tags: 
- Github
---

​	最近想搭一个个人博客，搜了一下之后找到了 Hexo 这个工具，正好借着 GitHub Pages 这个平台搭一个看看效果。

<!--more-->

​	个人博客应该是什么样子的？这一点估计每个人都有自己的想法。这里仅提供一些参考，个人博客还是需要自己去折腾为好。

​	经过小半天的折腾, 最终成品就是大家现在正在看着的博客了, 总共耗时1小时. 现在博客具备的功能如下:

- 一个优秀的界面(by next)
-  评论功能(by Valine)
- 我的联系方式
- 本地搜索系统(hexo-generator-search)
- 标签及分类
- 评论人数统计(leancloud)

## Steps

### 开始

​	首先在[github](https://github.com/)上需要一个自己的项目库, 如果没有帐号的需要自己先注册一个. 新建一个repository, 项目名为`whatever.github.io`, 选择生成默认README.md. 这样一个保存博客文件的项目库就做好了.

![新建Repo](https://i.loli.net/2019/01/05/5c30424ba3950.png)

### hexo

#### 安装

​	hexo安装前置要求是[Node.js](http://nodejs.org/)和[git](http://git-scm.com/), 需要先安装这两个才能安装hexo. 检测是否安装可以打开终端, 输入`node -v`和`git --version`, 如果没有报错则可以安装hexo了. 在终端输入`npm install -g hexo-cli  `即可安装hexo, 输入`hexo -v`检测是否安装成功, 没有报错就可以开始初始化了.

![检查安装](https://i.loli.net/2019/01/05/5c30424b85682.png)

#### 初始化

​	在任意位置新建一个文件夹用来存放博客文件, 然后在终端界面打开文件夹, 输入`hexo init`即可完成初始化 , 当看到`INFO  Start blogging with Hexo!`时表明初始化完成, 此时的项目结构如下:

```
.
├── _config.yml #项目配置文件
├── package.json
├── node_modules
├── package-lock.json
├── scaffolds
├── source
|   └── _posts #文章存放
└── themes #存放主题
```

​	运行`npm install`安装依赖, 然后就可以开始配置了.

#### 配置

​	项目的配置在_config.yml中进行, 用任意文本编辑器打开后, 需要自己根据自己的情况更新以下版块(每一部分可以通过搜索找到):

- site
- URL

`url`填写自己在github上新建的库名的github pages网址就行了(https://yourrepository), 比如我的就是(https://yucklys.github.io).

- Deployment
  `type`填写git, 然后填写repository和branch如下

```yaml
deploy:
  type: git
  repository: git@github.com:Yucklys/yucklys.github.io.git
  branch: master
```

​	repository填写项目库SSH链接, 从项目库的clone or download选项中复制SSH链接然后粘贴, branch填写master.

#### hello world

​	终端输入`hexo g`生成页面, 结束后输入`hexo s`在本地服务器上显示页面, 默认端口是4000, 显示`INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.`时打开浏览器, 输入localhost:4000并跳转即可看到自己的博客页以及初始的一个文章. 

### 上传repository

​	首先全局声明自己身份, 在终端输入:

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

​	在终端输入`cd ~/.ssh `, 如果返回”… No such file or directory ”, 则直接输入`ssh-keygen -t rsa -C "your_email@youremail.com" `(**邮箱换成你的邮箱**). 如果能进入ssh文件夹的话则输入`mkdir key_backup mv id_isa* key_backup `备份, 然后再输入`ssh-keygen -t rsa -C "your_email@youremail.com" `.

​	可以一路回车, 如果想要每次输入密码的话, 也可以设置密码.

​	然后依次输入以下命令:

```
ssh-agent -s
ssh-add ~/.ssh/id_rsa
```

​	如果出错显示`Could not open a connection to your authentication agent `, 就输入

```
eval `ssh-agent -s`
ssh-add
```

​	之后输入`clip < ~/.ssh/id_rsa.pub`复制ssh key到剪切板. 在github用户设置界面可以看到**SSH and GPG keys**选项, 在该项下选择New SSH key, 随便写个名字, 然后将key粘贴到key中并保存. 之后在终端中输入`ssh -T git@github.com`查看是否成功添加, 有警告就一路yes, 出现`Hi username! You've successfully authenticated, but Github does not provide shell access`说明添加成功.

#### 部署

​	终端切换到博客文件夹, 输入`hexo d`部署, 显示`INFO  Deploy done: git`表明部署完成. 在浏览器中输入yucklys.github.io即可进入到博客界面了.

## 结语

​	hexo的next主题最大的优点就是它的用户广泛, 所以大部分的功能都可以在`主题配置文件`中找到, 包含有完整的帮助文件以及众多next主题的大神用户自定义的方案, 可以说只需要~~cv~~简单模仿就可以做出一个不错的博客页. 当然写作的质量还是很关键的, 会用markdown是必不可少的一项技能, 这里可以去看一下hexo的[官方写作教程](https://hexo.io/zh-cn/docs/writing.html), markdown的教程网上有许多, 善用google或百度.

---

​	*2018.11.4更新*

现在使用的是[icarus](https://github.com/ppoffice/hexo-theme-icarus)主题，换主题有两方面原因，一是**icarus**的确是一款很优秀的主题，二是**next**用的人实在是太多了，每次看别人的博客都有种撞衫的不爽感……
