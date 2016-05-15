---
title: Loadrunner Error问题处理
date: 2016-05-13 12:06:37
tags: 
 - Loadrunner
categories: Performance Test
#toc: true
---
# Loadrunner报错，处理方法记录
<!--more-->
## Error -26488: Could not obtain information about submitted file
脚本中的上传操作回访时报错Error -26488是因为Loadrunner默认文件上传位置为当前脚本所在的目录。
> 解决方法：将上传的文件放到脚本的目录下

