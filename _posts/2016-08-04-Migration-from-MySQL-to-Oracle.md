---
layout: post
title: MySQL数据库迁移到Oracle
description: "冷知识"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,数据库]
imagefeature: 
comments: true
share: true
---
# MySQL数据库迁移到Oracle  #

时间: 2016-08-04 15:10:21

## 前言 ##

1. 本人迁移的数据库规模不大.大型数据库未做尝试
2. 请务必注意文中提到的一些注意事项
3. 本文中,要迁移的mysql数据库名称为jxcms,oracle sql developer中配置的连接oralce数据库的用户为rwdb.所有内容均基于这个前提


## 采用工具 ##

Oracle SQL Developer 版本4.1.3.20

mysql-connector 版本5.1.39

下载地址

[Oracle SQL Developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html "oracle-sql-developer") 

[mysql-connector](http://dev.mysql.com/downloads/connector/j/ "mysql-connector")

## 迁移工作 ##

### 配置mysql数据库 ###

添加驱动：打开Oracle SQL Developer,菜单栏-工具-首选项-数据库-第三方JDBC驱动程序，选择添加条目，导入jar包。

![mysql驱动配置](http://img.blog.csdn.net/20160804145847408)

新建连接.填写mysql相关配置

![mysql连接配置](http://img.blog.csdn.net/20160804145756098)

### 配置oralce数据库 ###

![oracle连接配置](http://img.blog.csdn.net/20160804160553047)

注意：
1. 配置的oralce用户必须要有create table的权限;
2. 移至数据库的时候,SQL Developer会额外创建一个和你要移至的mysql数据库同名的用户,移植过来的数据全在这个同名用户下.比如本文中要移植的mysql数据库名称为jxcms,配置oracle连接时配置的用户为rwdb,那么在移植的时候,会在oracle中创建一个名称和密码均为jxcms的用户,移植过来的数据就在这个jxcms用户下,而rwdb用户下则会创建一系列移植数据库相关的表,移植完成后手动删除即可.


### 数据库迁移 ###

展开mysql连接,右键要移植的数据库,选择 移植到Oracle

![移植向导](http://img.blog.csdn.net/20160804150016726)

下一步X10,完成.

导入的数据在名为jxcms、密码为jxcms的用户下


### 收尾工作 ###

移植数据库的过程中,会在rwdb用户下额外创建一些表(即移植资料档案库),同时也会创建额外的角色ROLE_JXCMS.

那么在移植完成后

1. 右键oralce连接,选择 移植资料档案库-删除移植资料档案库,即可把这些额外表删除;
2. 执行如下SQL
 
```
DROP USER EMULATION CASCADE; //该用户为创建移植资料档案库时自动创建
	
DROP ROLE ROLE_JXCMS CASCADE; //移植数据库时,会自动创建名为jxcms的用户和名为role_jxcms的角色,根据自身需要可酌情删除ROLE_JXCMS.


ALTER USER JXCMS identified by jxcms default tablespace JX_CMS temporary tablespace JX_CMS_TEMP; //JXCMS 默认的表空间为USER,JX_CMS、JX_CMS_TEMP为自己事先创建的表空间

```

## 可能用到的SQL ##

### 创建表空间 ###

```
/*  创建临时表空间  */
create temporary tablespace JX_CMS_TEMP  
tempfile 'D:\app\Administrator\oradata\orcl2\JX_CMS_TEMP.dbf' 
size 100m  
autoextend on  
next 50m maxsize 500m  
extent management local;  
 
/*  创建数据表空间  */
create tablespace JXCMS
logging  
datafile 'D:\app\Administrator\oradata\orcl2\JX_CMS.dbf' 
size 100m  
autoextend on  
next 100m maxsize 1000m  
extent management local;  
```

###  删除用户/角色 ###

```
/*  删除用户  */
DROP USER XXX CASCADE; 
/*  删除角色  */
DROP ROLE XXX CASCADE; 
```

###  修改默认表空间 ###

```
ALTER USER XXX1 identified by XXX2 default tablespace XXX3 temporary tablespace XXX4;
```

###  查看一共有哪些用户/角色 ###

```
SELECT * FROM SYS.user$ ;
```

###  查看用户默认表空间、临时表空间 ###

```
select username,default_tablespace,temporary_tablespace from dba_users ;
```

