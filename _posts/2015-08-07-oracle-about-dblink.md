---
layout: post
title: 关于ORACLE的DB_LINK
description: "笨就要多读书，笨就要多记笔记T.T"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Oracle]
imagefeature: 
comments: true
share: true
---

db_link目的是为了使我们跨越本地数据库访问另外一个数据库中的表中的数据。

## 1.创建DB_LINK ##
	
   1. 前提:

   创建dblink的用户有对应的数据库权限:create public database link 或者 create database link 
   可以使用

		grant create public database link,create database link to myAccount;
        
   来授权.

   2. 创建
   

   	    create public database link dblinkname connect to username 
		identified by password 
		using '(DESCRIPTION = 
		(ADDRESS_LIST =
		(ADDRESS = (PROTOCOL = TCP)(HOST = database_ip)(PORT = 1521))
		)
		(CONNECT_DATA =
		(SERVICE_NAME =servicename)
		)
		)';

   说明:
      
   + host=数据库的ip地址，service_name=数据库的ssid;
   + 如果不加 `public`，那么创建的dblink只有当前用户可用; 
	
   

## 2. 查看DB_LINK ##

		select * from dba_db_links;
		
		select * from dba_objects where object_type='DATABASE LINK';

## 3. 使用DB_LINK ##

		SELECT * FROM table_name@db_link_name;

## 4. 删除DB_LINK ##
 	
		drop public database link db_link_name

