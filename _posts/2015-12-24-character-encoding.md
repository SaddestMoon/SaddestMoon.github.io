---
layout: post
title: 字符编码相关(简略版)
description: "笨就要多读书"
category: 成长の足迹
tags: [笔记 , 代码狗 ,Java]
imagefeature: 
comments: true
share: true
---

12/24/2015 10:26:19 AM 

参考 

1. [http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/ "IMB 深入分析 Java 中的中文编码问题")
2. [http://cosmo1987.iteye.com/blog/1116959](http://cosmo1987.iteye.com/blog/1116959 "HTML GET和POST编码问题")

## 1.请求 ##

### 1.1 GET操作 ###

 在tomcat的server.xml中设置connector为`URIEncoding="utf-8" useBodyEncodingForURI=”true”`
 
 否则tomcat将采用默认的ISO-8859-1对请求的URL进行编码。
 
### 1.2 POST操作 ###

POST传递表单参数,一般在界面上设置即可

	/* 优先级最高。(JSP界面) */
	pageEncoding="utf-8"
	/* 优先级次之。同时这也是界面解析返回数据所采用的编码格式 */
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	/* 优先级最低 */
	<meta charset="utf-8" />

提交表单的时候，先根据界面头部的Content-Type的charset的设置，对参数进行编码。

服务端接受到参数时，第一次解码发生在request.getParameter，此时解码也是根据Content-Type的设置来的。

所以一般POST传参数一般不会出现乱码。

## 2.响应 ##

response默认采用ISO-8859-1进行编码。

## 3.其他需要编码的地方 ##

### 3.1 xml文件 ###

xml 文件可以通过设置头来制定编码格式

	<?xml version="1.0" encoding="UTF-8"?>

### 3.2 Velocity ###

Velocity 模版设置编码格式：

	services.VelocityService.input.encoding=UTF-8

### 3.3 JSP ###

JSP 设置编码格式：

	<%@page contentType="text/html; charset=UTF-8"%>

### 3.4 JDBC ###

访问数据库都是通过客户端 JDBC 驱动来完成，用 JDBC 来存取数据要和数据的内置编码保持一致，可以通过设置 JDBC URL 来制定，如： 

	/* MySQL */
	url="jdbc:mysql://localhost:3306/DB?useUnicode=true&characterEncoding=GBK"

## 4.常见编码错误 ##

### 4.1 一个汉字变成一个问号 ###

例如 字符串"淘！我喜欢！"变成了"？？？？？？"

"淘！我喜欢！"-->ISO-8859-1编码-->ISO-8859-1解码-->"？？？？？？"

![一个汉字变成一个问号](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/image029.gif)

### 4.2 一个汉字变成两个问号 ###

例如 字符串"淘！我喜欢！"变成了"？？？？？？？？？？？？"

"淘！我喜欢！"-->GBK编码-->ISO-8859-1解码-->GBK编码-->GBK解码-->"？？？？？？"

这种情况比较复杂，中文经过多次编码，但是其中有一次编码或者解码不对仍然会出现中文字符变成“？”现象，出现这种情况要仔细查看中间的编码环节，找出出现编码错误的地方。

![一个汉字变成两个问号](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/image031.gif)

### 4.3中文变成了看不懂的乱码字符 ###

例如，字符串"淘！我喜欢！"变成了"Ì Ô £ ¡Î Ò Ï²»¶ £ ¡"

"淘！我喜欢！"-->GBK编码-->ISO-8859-1解码-->"Ì Ô £ ¡Î Ò Ï²»¶ £ ¡"

![中文变成了看不懂的乱码字符](http://www.ibm.com/developerworks/cn/java/j-lo-chinesecoding/image027.gif)

