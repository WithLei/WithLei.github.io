---
layout: post  
title:  "Flutter开发中遇到的坑 - Waiting for another flutter command to release the startup lock"  
date: 2018-12-07  
description: "Flutter开发中遇到的坑 - Waiting for another flutter command to release the startup lock"
tag: Flutter
---

## 本机环境
 - 系统：Windows 10 X64
 - IDE：Android Studio
 
 这个错误发生于在编译器里get package，等了很久没有获取到package之后，我果断关掉了AS，打开终端输入了flutter doctor但是出现了这个错误。
 ![error](https://img-blog.csdnimg.cn/20181207163710675.png)
查了一下github的flutter issue 找到了解决方法，如下： 
1. 打开flutter的安装目录/bin/cache/ 
2. 删除lockfile文件 
3. 重启AndroidStudio