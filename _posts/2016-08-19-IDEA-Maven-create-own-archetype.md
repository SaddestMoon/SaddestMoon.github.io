---
layout: post
title: IDEA中maven项目创建自己的archetype并使用
description: "论偷懒的办法"
category: 成长の足迹
tags: [笔记 , 代码狗 ,Java]
imagefeature: 
comments: true
share: true
---
# IDEA中maven项目创建自己的archetype并使用 #

## 1.前言 ##

1. 本文假设用的是自己本机的maven repository.
2. 用的是windows,其它操作系统本文中需要调整的地方请自行脑补...

## 2.正文 ##

### 2.1创建并安装archetype ###

首先,我们得有一个比较完善的可以用来构建archetype的maven工程.

我这里就随手找一个已有的项目来作为示例.

在项目的根目录(即项目的pom文件所在目录)下执行maven命令:

		mvn archetype:create-from-project

![](http://i.imgur.com/ey1LxPV.png)

这样本项目的archetype就创建好了.注意生成的archetype的目录.

	我的项目是在D:\IDEAProjects\demo下,生成的archetype是D:\IDEAProjects\demo\target\generated-sources\archetype

在archetype的根目录下再执行以下maven命令:

	mvn install

![](http://i.imgur.com/Ljgotc8.png)

这样就把该archetype安装到了你自己的maven repository中了.


**注意:**记下你的archetype安装到的目录,在里面找到这个archetype的pom文件,2.3中要用到.

### 2.2将本机的maven库部署为web服务 ###

已在本机搭建maven私服的请绕过此步.

目的就是为了让自己maven repository能通过http的方式访问到.

根据个人喜好吧,采用tomcat、jetty、IIS什么的都可以,我这里使用的IIS.

比如我本机的maven repository是在`D:\repository\maven`

我是发布在了本机的9999端口上:

![](http://i.imgur.com/lBU4Xf6.png)

那么我通过http://localhost:9999/就可以访问到我本机的maven repository.

### 2.3使用 ###

IDEA中,新建maven project,选择add archetype.

![](http://i.imgur.com/nevb93O.png)

还记得在2.1最后让去找的archetype的pom文件吗？groupId、ArtifactId、Version就按照这个pom文件中的来填.

Repository 填2.2自己发布的url,我的就是 http://localhost:9999/

点OK完成添加,然后剩下的就跟平时创建maven项目没什么区别了.

## 3.注意事项及可能会遇到的问题 ##

如果你是和我一样,在IDEA中执行maven命令,那么一定要注意去看下你的maven的配置,执行命令时用的是IDEA自带的maven插件,还是你自己安装的.

### 用的是IDEA自带的maven插件 ###

那么你在用install安装archetype时,使用的maven resposity就是默认的目录,默认目录一般在C盘的用户目录下,比如:

	C:\Users\Administrator\.m2\repository

### 用的是自己安装的maven插件 ###

那么你在用install安装archetype时,使用的maven resposity就是你maven安装目录下的conf/settings.xml中所配置的目录(如果settings.xml中没配置,那就使用的也是默认的目录,同上).

如果执行maven命令时,报错提示mvn.bat找不到,最简单粗暴的方式就是,找到maven安装目录,把bin目录下的mvn.cmd复制一份,重命名为mvn.bat就行了.

## 其它 ##

在2.3步骤中添加的archetype,IDEA中界面上是没提供修改及删除按钮的,如果想修改或删除,可以打开如下文件手动操作:

	C:\Users\Administrator\.IntelliJIdea2016.2\system\Maven\Indices\UserArchetypes.xml

其中 IntelliJIdea2016.2 是我安装的IDEA版本,各位依自己具体情况而定.