---
layout: post
title: IDEA中maven项目创建自己的archetype并使用
description: "论偷懒的办法"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Java]
imagefeature: 
comments: true
share: true
---

## 1.前言 ##
	
	本文最初写于2016-09-28,文中有些不合适,以及不完善或过时的地方.特进行补充修正.
	
	1. 关于在IDEA中添加archetype时,respository url相关内容不合适的修改
	2. 增加配合私服,在其它人电脑上创建项目失败问题的解决方案(第三章节)
	

	目前(截止2019-01-21)`maven-archetype-plugin`插件最新的版本是3.0.1,
	这个版本的插件会忽略掉`-DarchetypeRepository`参数,
	不确定这是个bug还是是3.X设计本是如此.
	本次更新中就按照3.X设计本是如此来写.
	
	如果创建的archetype只是自己一个人使用,那么仅看第二章节就够用了.
	
	但是如果是要上传到私服提供给团队内的其他人使用,那么可能就需要看下第三章节(上传到中央库则不用看).
	
																		--2019-01-21 

采用环境:

windows 10,IDEA2018.3,maven3.3.9,JDK1.8_102.

### 1.1 archetype是什么 ###

archetype 字面意思是 原型.可以理解为archetype相当于一个脚手架/模板,通过这个脚手架/模板我们可以快速的创建出一个项目.

比如下图中的这些就是maven给我们默认提供的archetype

![](https://i.imgur.com/pRANNtL.png)

通过其中的 `maven-archetype-webapp`,我们可以快速构建一个webapp项目.可以节省一定的工作量.

毕竟在实际的开发工作中,尤其是在项目型公司,每次新项目,都是要进行类似的项目初始化的搭建工作,工作量还是不算小的,而且一不小心就出错了.

本文讲的就是如何定制一个脚手架/模板(第二章),以及部署到私服上提供给团队使用时撞见的一些问题(第三章).

## 2.创建及使用 ##

### 2.1 创建archetype ###

首先,我们得有一个比较完善的可以用来构建archetype的maven工程.

我这里就随手找一个已有的maven项目来作为示例.

**在项目的根目录下**(即项目的`pom.xml`文件所在目录)下执行`maven`命令:

		mvn archetype:create-from-project

这样本项目的archetype就创建好了.

生成的archetype的信息,默认是在工程根目录下的 `target\generated-sources\archetype` 目录中.

### 2.2 安装archetype ###

**在archetype的根目录下**(即: `项目根目录\target\generated-sources\archetype`)再执行以下`maven`命令:

	mvn install

这样就把该archetype安装到了本地的maven repository中了.

### 2.3 使用archetype ###

#### 2.3.1 通过命令行 ####

任意一个空目录下,执行如下命令

	mvn archetype:generate -DinteractiveMode=false -DgroupId=com.whht -DartifactId=test -Dversion=1.0-SNAPSHOT -DarchetypeGroupId=com.huitu.whht.archetype-project -DarchetypeArtifactId=web-api-archetype -DarchetypeVersion=1.0-RELEASE -DarchetypeRepository=中央库或私服地址(此配置项在最新版本是无效的,3.1有详细说明,一定要看)
 
命令详解:

	-DgroupId=com.whht 		要创建的工程的信息
	-DartifactId=test		要创建的工程的信息
	-Dversion=1.0-SNAPSHOT	要创建的工程的信息
	-DarchetypeGroupId=com.huitu.whht.archetype-project 	采用的archetype的信息
	-DarchetypeArtifactId=web-api-archetype 				采用的archetype的信息
	-DarchetypeVersion=1.0-RELEASE							采用的archetype的信息
	-DinteractiveMode 是否每次执行命令时都需要联网去中央库查
	-DarchetypeRepository=archetype所在的位置,默认中央库,可不填.(3.1有详细说明,一定要看).

#### 2.3.1 通过IDEA ####

IDEA中,新建 maven project,选择 add archetype.
按照 `项目根目录\target\generated-sources\archetype` 目录下的`pom.xml`中的信息填写上 GroupId、ArtifactId、Version.

Repository URL 不需要填,填了也没用,第三章有说明.
Repository URL 不需要填,填了也没用,第三章有说明.
Repository URL 不需要填,填了也没用,第三章有说明.

![archetype目录下的pom.xml](https://i.imgur.com/IEi6dcN.png)

![IDEA配置](https://i.imgur.com/RjYSQ96.png)

点OK完成添加,然后剩下的就跟平时创建maven项目没什么区别了.

## 3. 上传私服但其他人使用失败问题 ##

如果是把创建的archetype上传到maven的中央库中,在使用上就不会有任何问题.

但是如果把创建的archetype上传到私服上,其他人通过私服直接进行项目构建,却始终提示说指定构建不存在,报错信息大致如下:

![命令行出错截图](https://i.imgur.com/WsGaFof.png)

报错信息中如果有这三行,那么本章节应该可以解决这个问题.

	[WARNING] Archetype not found in any catalog. Falling back to central repository.
	[WARNING] Add a repsoitory with id 'archetype' in your settings.xml if archetype's repository is elsewhere.
	[WARNING] The POM for com.huitu.whht.archetype-project:web-api-archetype:jar:1.0-RELEASE is missing, no dependency information available

PS:很奇怪是不是?命令行里不是指定了`-DarchetypeRepository`参数,让去私服找,为什么还说找不到?

找到的其它的博客或教程上,内容基本上和本文第二章节差不多,可能会详细很多.但是,在其它人的电脑上,除非也把第二章的流程走一遍,否则基本上都无法成功执行.下面就一个个的说原因和解决方案.

关于如何搭建私服及上传私服需要增加哪些配置,本文不再赘述.

上传私服,和一般的maven项目上传私服没什么区别. `项目根目录\target\generated-sources\archetype` 目录下执行:`mvn deploy` 上传私服.

### 3.1 命令行创建项目失败 ###

请务必去本地maven库中检查下 `org.apache.maven.plugins:maven-archetype-plugin`这个jar的最新的版本号是什么.

基本上都是因为最新的版本号是3.X导致的,而2.X是不会出现这个问题的.

删了本地的3.X的jar也没用.

#### 3.1.1 原因 ####

`mvn archetype:generate` 命令, 会去maven中央库去下载最新的`org.apache.maven.plugins:maven-archetype-plugin`这个jar,来进行命令的解析执行,目前最新的是3.X版本(截止2019-01-21 为 3.0.1).

命令在执行时,默认是去maven中央库中去找指定的archetype,命令行参数中的`-DarchetypeRepository` 可以指定archetype所在maven库的地址,但是这个参数**只在2.X中有效,3.X中是无效的**.

所以实际的过程就成了:

1. 命令行执行 `mvn archetype:generate` 
2. maven检查`maven-archetype-plugin`版本是否为最新
3. 如果本地不是最新的,maven去中央库下最新的`maven-archetype-plugin`插件(下载下来了3.X版本)
4. `maven-archetype-plugin`插件去本地库找指定我们自己创建的archetype(没有找到)
5. `maven-archetype-plugin`插件去maven中央库找我们自己创建的archetype(也没有找到)

我们在 2.3.2 章节,配置IDEA时,填写的`Repository URL`实际上就是`-DarchetypeRepository`参数.而IDEA在创建时,实际上也是执行的`mvn archetype:generate`命令(可以看IDEA控制台),那必然会失败,这也是前面说`Repository URL`填了也没用的原因.

解决方法有两个:**采用2.X版本**或者是**让3.X去私服中去找**

##### 3.1.1.1 解决方法一--手动指定插件版本(治标) #####

这个仅能解决手动执行命令时的问题.

即在命令行执行时,其余参数不变,只把 `mvn archetype:generate` 改为 `org.apache.maven.plugins:maven-archetype-plugin:2.4:generate`

手动指定`maven-archetype-plugin`的版本为 2.4,这样就可以了.

##### 3.1.1.2 解决方法二--修改settings.xml(推荐,治本) #####

让3.X去私服中去找我们创建的archetype.

**这个可以解决包括通过IDEA或者手动在命令行下执行的问题.**

**这个可以解决包括通过IDEA或者手动在命令行下执行的问题.**

**这个可以解决包括通过IDEA或者手动在命令行下执行的问题.**

打开自己的maven的settings.xml文件.

这里需要注意下,`mvn`命令在执行的时候,会按照如下顺序来找配置:

1. pom 即 `pom.xml`
2. user settings `${user.home}/.m2/settings.xml`
3. globa settings `${MAVEN_HMOE}/conf/settings.xml`

优先级为 pom > user settings > globa settings,而我们这里是基于`archetype`来创建项目,`pom.xml`还没有呢.所以,具体到这里,相同配置实际上会优先采用`${user.home}/.m2/settings.xml`的.

搞清楚优先级,确保改的配置会被加载执行.

1. 在 `settings.xml`的`<profiles></profiles>` 节点下 中增加如下配置.

		<profile>
			<id>profileArchetype</id>
			<repositories>
			<repository>
				<!--  这里的ID必须是叫 archetype  -->
				<id>archetype</id>
				<name>私服地址 name</name>
				<url>私服地址 url</url>
				<releases>
					<enabled>true</enabled>
				</releases>
			 </repository>
			</repositories>
		 </profile>

注意: 

`<repository>`的id必须是叫 **archetype**

`<repository>`的id必须是叫 **archetype**

`<repository>`的id必须是叫 **archetype**

2. 在`<profiles></profiles>`之后,增加如下配置激活profile:
	
		<activeProfiles>
	   		<activeProfile>profileArchetype</activeProfile>
	 	</activeProfiles>

3. (可选)如果这个私服,在下载jar包时,也需要登录认证的话,那么在`<servers></servers>`节点中增加私服登录帐号信息:

		<server>  
			<!--  和上面的repository配置一样,这里的ID必须是叫 archetype  -->
		    <id>archetype</id>  
		    <username>私服登录用户名</username>  
		    <password>私服登录密码</password>  
	  	</server>

### 3.2 IDEA中创建失败 ###

IDEA根据选择的archetype去创建工程时,本质上也是执行的`mvn archetype:generate`这个命令,所以如果命令行下失败,那么IDEA里肯定也是失败的.

原因具体参照 3.1. 

解决方案采用 3.1.1.2,修改`settings.xml`的配置即可.

## 4. 其它 ##

### 4.1 如何删除IDEA中自己添加的archetype ###

在2.3.2步骤中添加的archetype,IDEA中界面上是没提供修改及删除按钮的,如果想修改或删除,可以打开如下文件手动操作:

	${user.home}\.IntelliJIdea2018.3\system\Maven\Indices\UserArchetypes.xml

其中 IntelliJIdea2018.3 是我安装的IDEA版本,各位依自己具体情况而定.
