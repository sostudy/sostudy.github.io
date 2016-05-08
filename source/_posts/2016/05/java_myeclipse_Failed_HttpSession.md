---
title: MyEclipse2015启动Tomcat中项目时报错HttpSession出错
date: 2016-05-05 21:30:37
tags:
 - Failed
 - JDK
categories:
 - MyEclipse
#toc: true
---
## MyEclipse2015启动Tomcat中项目时报错HttpSession出错
<!--more-->
Tomcat中项目时报错HttpSession出错，如下
``` java
org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[localhost].StandardContext[/mj_oto_edus_auto_new]]
	************************************
Caused by: java.lang.NoClassDefFoundError: HttpSession
	************************************
Caused by: java.lang.NoClassDefFoundError: HttpSessionBindingEvent
	************************************
```

百度搜索HttpSession的信息,百科对它的简单介绍是Java的一个接口。
> HttpSession是Java平台对session机制的实现规范，因为它仅仅是个接口，具体到每个web应用服务器的提供商，除了对规范支持之外，仍然会有一些规范里没有规定的细微差异。

#### [_HttpSession_ (Java EE 5 SDK)](http://www.baidu.com/link?url=qVAKkg9VvrY8jxVRdnzq9pMkL4ohgErwtvWY9PNUTF0K7fbY2rjd3uErWygKyqjYXB7tmNMMVy-wCQyO9u67yTK5Dc0LjR9aGaHlbaKOTgC0iEGz_WJaemVRAg-rYfsC&wd=&eqid=835b47ba000119cc00000003572af1b5)
发现原来他是在Java EE 5中，于是在MyEclipse中Libraries中加上“JavaEE 5.0 Generic Library”，继续执行项目，项目继续报错，缺少jstl。
查看“JavaEE 5.0 Generic Library”发现其中只有一个“javaee.jar”,于是在MyEclipse的目录中找到“javaee.jar”所在的目录，发现其中有个文件README.txt
``` java
This folder contains libraries obtained from the Sun JEE5 Referene SDK.  
http://java.sun.com/javaee/downloads/index.jsp
The license for these 3 jar libraries is also contained in this folder.

The JSF 1.2 implementation files (jsf-impl.jar, jsf-api.jar)
were obtained from: https://javaserverfaces.dev.java.net/files/documents/1866/52042/jsf-1.2_04-b07-FCS.zip

The JSTL 1.2 implementation files: (jstl-1.2.jar)
were obtained from: https://maven-repository.dev.java.net/repository/jstl/jars/jstl-1.2.jar
```

发现原来还少jsf-impl.jar, jsf-api.jar，jstl-1.2.jar这三个文件
于是继续在Library中添加JSF Mojarra 1.2_04 Libraries,JSTL 1.2 Library两项。
> JSF Mojarra 1.2_04 Libraries 包含jsf-impl.jar, jsf-api.jar
JSTL 1.2 Library 包含jstl-1.2.jar

然后运行Tomcat，问题解决


我是在MyEclipse 2015使用的，其他版本的不清楚，也可以直接下载到本地，加载到项目中。
> 百度网盘链接: http://pan.baidu.com/s/1gfNhWcz 密码: r39f

