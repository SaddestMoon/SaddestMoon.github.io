---
layout: post
title: 关于Android SDK更新那点P事
description: "其实是关于如何翻墙"
categories: 生活の印迹
tags: [Android]
imagefeature: 
comments: true
share: true
---

更新Andorid SDK的时候，在我大天朝，总是无法回避的一个问题是，如何绕过那道神秘的墙的限制。方法无外乎就是翻墙，或者是改hosts。我们是遵纪守法的好公民，翻墙不谈，偷偷改下hosts还是可以的。 

改hosts文件，关键问题在于，怎样才能找到一个合适的、可用的IP。
以Android开发者的[官网](developer.android.com) http://developer.android.com为例，一般步骤如下:

1. 打开[站长工具](http://tool.chinaz.com/ "打开站长工具"),选择** 其他工具--超级ping**，或者直接点[这里](http://ping.chinaz.com/ "打开超级ping")打开**超级ping**。
2. 弹出的页面，输入域名 **developer.android.com**，检测类型选择 **网站测速**，检测点选择 **海外**。然后点查询。
3. 查询结果里面选择一个总耗时最短的，复制相应的IP，比如我选择的是 74.125.239.131。
4. 将 ***74.125.239.131    developer.android.com*** 追加到hosts最后一行。

这样子就完事了。其他访问不了的网站也可以通过这样的方式来进行修改。

这种方式的缺点就在于每个域名都要在Hosts里面加一条记录，不过胜在免费，比较安全。

---
还是说说翻墙吧。

1. 使用翻墙软件。百度搜 "自由 门",注意中间的空格，不然会被度娘和谐。然后去下载一个。
2. Chrome下有个叫**红杏**的插件。不过要收费。以前免费的时候用过，效果蛮赞。
3. 使用公用VPN。无论免费还是付费都不太安全。就像让你连个公共Wifi上网银，妥？
4. 自己搭建个VPN去。搭建方法不谈，自行度娘。

---

话说元旦进行了系统还原，又格式化了其他分区，所有东西都要重来。泪奔T.T~


