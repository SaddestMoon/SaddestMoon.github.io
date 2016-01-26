---
layout: post
title: Oracle Expdp 及 Impdp
description: "mysql多好"
category: 成长の足迹
tags: [笔记 , 代码狗 ,Java]
imagefeature: 
comments: true
share: true
---

时间 12/29/2015 10:18:26 PM 

----

## expdp ##

### 1.创建虚拟目录directory ###

directory是转储文件和日志文件所在的目录。

	/* 创建虚拟目录 */
    create or replace directory dmp as 'd:/dmp';
	/* 为用户分配权限 */
	Grant read,write on directory dmp to SZY_HB;

注意:

- 本语句并不会真实的创建目录
- 如果要进行导出,则本目录必须在数据库服务器上存在
- 如果要导出到本地,请参考[http://blog.itpub.net/24389441/viewspace-1057855/](http://blog.itpub.net/24389441/viewspace-1057855/ "EXPDP/IMPDP：导出文件到本地端")

### 2.导出文件 ###

    expdp SZY_HB/123456@192.168.0.254:1522/ORCL2 directory=dmp  dumpfile=test.dmp 
	content=all logfile=test.log job_name=my_job 

	expdp SZY_HB/123456@127.0.0.1:1521/ORCL directory=dmp  dumpfile=T_GEOINFO_B1.dmp content=all logfile=test.log job_name=my_job tables=T_GEOINFO_B

#### 2.1 content 指定导出内容 ####

该选项用于指定要导出的内容.默认值为ALL

    CONTENT={ALL | DATA_ONLY | METADATA_ONLY}

- 当设置CONTENT为ALL时,将导出对象定义及其所有数据.
- 为DATA_ONLY时,只导出对象数据
- 为METADATA_ONLY时,只导出对象定义

#### 2.2 logfile 日志文件 ####

    logfile=test.log

#### 2.3 job_name 指定job ####

	job_name=test_job

- 如果没有指定导出的JOB名字那么就会产生一个默认的JOB名字
- 导出语句后面不要有分号，否则如上的导出语句中的job表名为‘my_job;’，而不是my_job。因此导致expdp时一直提示找不到job表

#### 2.4 tables 按表导出 ####

	tables=tables1,tables2,tables3

#### 2.5 query 按查询条件导出 ####

    query='"where rownum<11"'

#### 2.6 tablespaces 按表空间导出 ####
    
    tablespaces=SZY_HB.DBF

#### 2.7 full 导出整个数据库 ####

    full=y

## impdp ##

### 1.导入 ###

	impdp SZY_HB/123456@127.0.0.1:1521/ORCL directory=dmp dumpfile=test.dmp remap_tablespace=TS_SZY_HB:HT_SZY_HB

	impdp sl_szy_hb_extra/dhcextra@10.42.1.172:1521/szyhb directory=dmp dumpfile=T_GEOINFO_B.DMP remap_tablespace=HT_SZY_HB:TBS_DATAEXTRA

#### 1.1 更改表空间 #### 

	/* 原数据的表空间为TS_SZY_HB,导入到HT_SZY_HB中 */
    remap_tablespace=TS_SZY_HB:HT_SZY_HB

更多内容请访问[http://www.cnblogs.com/lanzi/archive/2011/01/06/1927731.html](http://www.cnblogs.com/lanzi/archive/2011/01/06/1927731.html "expdp\impdp")