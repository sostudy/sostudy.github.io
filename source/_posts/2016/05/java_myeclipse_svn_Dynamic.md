---
title: MyEclipse2015项目报错Failed while installing Dynamic Web Module 2.5
date: 2016-05-05 21:30:37
tags:
 - Failed
 - Svn
categories:
 - MyEclipse
#toc: true
---
## 错误信息Failed while installing Dynamic Web Module 2.5解决方法
<!--more-->
使用MyEclipse 2015从SVN检出代码后，项目无法放到Tomcat中，提示错误信息
> Failed while installing Dynamic Web Module 2.5

解决方法是在MyEclipse中 ，右键项目选择“Properties”，找到“Project Facets”项
> 找到 Dynamic Web Module ，把他后面的3.0改成 2.5

再执行添加项目操作就可以将项目导入Tomcat中了。