---
layout: post
title:  "Mac Big Sur 升级后 JD-GUI 无法打开的问题修复"
date:   2021-01-19
categories: JAVA
---
解决方法：

在JD-GUI的APP中右键打开包，找到Contents这个文件夹，大家可以找到：Info.plist这个文件，打开它

搜索”1.8+“然后把”+“删除掉，保存，大功告成！！！

The End.