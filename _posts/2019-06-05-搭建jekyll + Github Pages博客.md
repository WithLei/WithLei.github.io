---
layout: post  
title:  "搭建jekyll + Github Pages博客"  
date: 2019-06-05  
description: "讲解搭建jekyll + Github Pages博客过程"
tag: 博客搭建
---

个人博客：https://withlei.github.io/

> 操作系统：Windows10

# Jekyll是什么
> 引用自官网：
Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

简单来说，Jekyll就是将纯文本转化为静态博客网站，不需要数据库支持，也没有评论功能，想要评论功能的话可以借助第三方的评论服务。
Jekyll + Github Pages可以让你更加专注于博客内容，而不是如何搭建一个博客平台。

# Github Pages是什么
> GitHub Pages 是一个静态网站托管服务，直接从github仓库托管你个人、公司或者项目页面 ，并且不需要你写任何后端语言来支持。

Github Pages的服务是免费的，但是也有一些限制：

仓库空间不大于1G
每个月的流量不超过100G
每小时更新不超过 10 次

但是这些限制对我们普通人来说肯定没影响的，所以可以忽略。
这里只是将Github Pages当做一个平台，其他详细信息可以在[Github Pages 文档](https://help.github.com/en/categories/github-pages-basics)查看

# 开始安装Jekyll
安装Jekyll前，确认已经按下步骤进行安装
1. 安装Ruby+DevKit
2. 安装Jekyll
## 1.安装Ruby+DevKit
下载地址：https://rubyinstaller.org/downloads/![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605193903523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
直接选择带DevKit的版本

安装完成后，检查ruby是否安装成功
```
ruby -v
gem -v //查看gem 是否正常安装
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605194141500.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605194445405.png)
## 2.安装Jekyll
```
gem install jekyll
```
**快速启动**
安装了 Jekyll 的 Gem 包之后，就可以在命令行中使用 Jekyll 命令了。官网提供了一个快速启动的例子：
```cmd
# 安装bundler，bundler通过gemfile文件来管理gem包
gem install  bundler

# 创建一个新的Jekyll项目，并命名为myblog
jekyll new myblog

# 进入myblog目录
cd myblog

# 在Jekyll自带的服务器上预览你的项目，默认的运行地址为http://localhost:4000
# bundle exec 表示在当前项目依赖的上下文环境中执行命令 jekyll serve
bundle exec jekyll serve
```
Jekyll 自带了一个开发用的服务器，可以让你使用浏览器在本地进行预览。
```
jekyll serve
# 开发服务器将会运行在 http://localhost:4000/
```
serve 指令将会自动监测变化，生成新的文件。想关闭这功能，你可以使用 jekyll serve --no-watch，这里还有其他几个参数：
- jekyll serve --livereload相当于前端开发中自动刷新浏览器。
- jekyll serve --incremental相当于模块热替换，只刷新更改的模块。
- jekyll serve --detach  和jekyll serve命令相同，但是会脱离终端在后台运行，如果你想关闭服务器，可以使用kill -9 1234命令，"1234" 是进程号（PID）。如果你找不到进程号，那么就用ps aux | grep jekyll命令来查看，然后关闭服务器。

访问测试链接：http://127.0.0.1:4000/
出现该页面说明创建部署成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605205005984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5,size_16,color_FFFFFF,t_70)
# 开始搭建Github Pages
## 1.使用Jekyll写博文
你可能喜欢markdown或html来写博文，都可以，但是博文文件的命名规则要服从下面的规则：
```
 year-month-title.markup //markup为你的文件格式的后缀名
 ```
 在你的文章头部添加yaml头信息
```
---
layout: post
title:  "Jekyll+Github搭建个人博客"
date:   2019-06-05
categories: original
---
```
写上自己的博文内容，将这个文件保存在blog里面的_posts目录里面即可。再重启jekyll内置服务器，刷新页面：http://localhost:4000，如果没有，可以先输入：
```
jekyll build 
```
重新生成页面，在启动服务器，这样就可以在页面看到自己添加的博文的标题了。
这就是在本地搭建jekyll和写博文的大致过程了，相信还有其他的搭建方法，但是估计都是大同小异吧。
## 2.部署到Github Pages
这里有两种方法：
1. 使用命令完成
2. 使用GitHub desktop应用完成
### 1）使用命令操作
接下来的操作都是用GIT命令完成的，不再是cmd了。首先，大家应该都拥有了github账号，没有的注册一个就好了。

- 创建个人仓库
就是建立一个新的仓库，但是这个仓库的名字必须为你的github的名字+github+io，即yourname.github.io
- 将目录切换到你想要放github博客的文件目录下，在这个目录git bash 将刚才建的仓库克隆下来：
```
  git clone git@github.com:yourname/yourname.github.io.git
  ```
这时，你会发现你的文件夹下会多出一个yourname的文件，我们把之前的blog下的所有文件复制到里面。
- 然后把里面的所有文件push到刚刚建的远程仓库，步骤我就不写了。
这时，在浏览器里面输入网址：http://yourname.github.io 就可以看你的个人博客网站了，这就是你的博客网站的地址了。
**前面所说的yourname指的是你的github账号名字。**
- 嗯，接下来你就可以查看你的博客网站了。其中还可以在github的settings中选择你的博客主题。

### 2）使用GitHub desktop应用
将本地博客使用desktop上传仓库，仓库名为 yourname.github.io ，接着同步后就可以访问了

参考博客：
[windows下Jekyll安装步骤及问题 - 彭世瑜](https://blog.csdn.net/mouday/article/details/79300135)
[Jekyll中文官网](https://www.jekyll.com.cn/)
[Jekll+Github Pages搭建静态博客](https://www.jianshu.com/p/9f198d5779e6)
[Jekyll搭建个人博客-潘柏信](http://baixin.io:8000/2016/10/jekyll_tutorials1/)