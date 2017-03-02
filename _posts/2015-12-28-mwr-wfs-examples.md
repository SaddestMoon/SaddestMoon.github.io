---
layout: post
title: 水利部WFS相关服务调用
description: "没内网有P用"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,GIS]
imagefeature: 
comments: true
share: true
---

12/28/2015 6:55:45 PM 

# 1.缓冲区查询 #

# 2.查询某省的数据 #
	
	/*湖北省BBOX*/
	108.3624809058997,29.032619956462256,116.1324580113027,33.27291933150885
	
	/* 湖北省--水文测站--OBS (总数:167) */
	http://10.1.3.140/iserver/services/data-MWR_OBS_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=OBS:OBS&PROPERTYNAME=(ID,NAME,SMX,SMY,the_geom)&Filter= (<Filter>
	<ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\" ><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>42*</ogc:Literal></ogc:PropertyIsLike></Filter>)

	/* 湖北省--地表水取水口--WINT (总数:10005) */
	http://10.1.3.140/iserver/services/data-MWR_WINT_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=WINT:WINT&PROPERTYNAME=(INT_CD,INT_NM,SMX,SMY,the_geom)&Filter= (<Filter>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>省代码</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo></Filter>)

	/* 湖北省--地表水水源地--SWS (总数:814)*/
	http://10.1.3.140/iserver/services/data-MWR_SUWS_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=SUWS:SUWS&Filter= (<Filter>
	<ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\" ><ogc:PropertyName>SWS_CD</ogc:PropertyName>
	<ogc:Literal>42*</ogc:Literal></ogc:PropertyIsLike></Filter>)

	/* 湖北省--取用水户--WIU (总数:263) */
	http://10.1.3.140/iserver/services/data-MWR_WIU_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=WIU:WIU&Filter= (<Filter>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>省代码</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo></Filter>)
	
	/* 湖北省--省级行政区划--PROV (总数:1) */
	http://10.1.3.140/iserver/services/data-MWR_PROV_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=PROV:GEO_PROV_5T&MAXFEATURES=2&Filter=(<Filter><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>ID</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo></Filter>)

	/* 湖北省--县级级行政区划--CNTY */
	http://10.1.3.140/iserver/services/data-MWR_CNTY_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=CNTY:GEO_CNTY_5T&Filter=(<Filter><ogc:PropertyIsLike wildCard="*" singleChar="?" escapeChar="\" xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>420000*</ogc:Literal></ogc:PropertyIsLike></Filter>)

	/* 湖北省--水功能分区--SZY_WFZ1 (总数:198) */
	http://10.1.3.140/iserver/services/data-MWR_SZY_WFZ1_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=SZY_WFZ1:SZY_WFZ1&BBOX=108.3624809058997,29.032619956462256,116.1324580113027,33.27291933150885

	/* 湖北省--水功能分区--SZY_WFZ2 (总数:177) */
	http://10.1.3.140/iserver/services/data-MWR_SZY_WFZ2_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=SZY_WFZ2:SZY_WFZ2&BBOX=108.3624809058997,29.032619956462256,116.1324580113027,33.27291933150885

	/* 湖北省--湖泊--LAKE (总数:396) */
	http://10.1.3.140/iserver/services/data-MWR_LAKE_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=LAKE:LAKE&BBOX=108.3624809058997,29.032619956462256,116.1324580113027,33.27291933150885
	
	/* 湖北省--水库大坝--DAM (总数:6459) */
	http://10.1.3.140/iserver/services/data-MWR_DAM_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=DAM:DAM&Filter= (<Filter>
	<ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\" ><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>42*</ogc:Literal></ogc:PropertyIsLike></Filter>)

	/* 湖北省--水闸--GATE (总数:7457) */
	http://10.1.3.140/iserver/services/data-MWR_GATE_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=GATE:GATE&PROPERTYNAME=(ID,NAME,USERX,USERY,the_geom)&Filter=(<Filter>
	<ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\" ><ogc:PropertyName>ID</ogc:PropertyName>
	<ogc:Literal>42*</ogc:Literal></ogc:PropertyIsLike></Filter>) 

	/* 湖北省--入河排污口--PDO (总数:921) */
	http://10.1.3.140/iserver/services/data-MWR_PDO_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=PDO:PDO&Filter= (<Filter>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>省代码</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo></Filter>)
	
	/* 湖北省--水库--RSWB */
	/* 水库的服务每次最多返回2000条 */
	/* 大一 总数 10 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName>
	<ogc:Literal>大一</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 大二 总数 65 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName>
	<ogc:Literal>大二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)


	/* 中型 总数 282 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName>
	<ogc:Literal>中型</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小一型 总数 1235 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName>
	<ogc:Literal>小一</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小二型 ID-4201~4205 总数 1226 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4201*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4202*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4203*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4204*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4205*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小二型 ID-4206~4209 总数 1394 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4206*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4207*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4208*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4209*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小二型 ID-4210~4212 总数 1450 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4210*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4211*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4212*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小二型 ID-4213~4219 总数 589 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4213*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4214*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4215*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4216*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4217*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4218*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>4219*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)

	/* 小二型 ID-422~424 总数 176 */
	http://10.1.3.140/iserver/services/data-MWR_RSWB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RSWB:RSWB&Filter= (<Filter><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>422*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>423*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND><AND><ogc:PropertyIsLike xmlns:ogc="http://www.opengis.net/ogc" wildCard="*" singleChar="?" escapeChar="\"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>424*</ogc:Literal></ogc:PropertyIsLike><ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>GRADE</ogc:PropertyName><ogc:Literal>小二</ogc:Literal></ogc:PropertyIsEqualTo></AND></Filter>)		


	/* 河流--RIVER */
	http://10.1.3.140/iserver/services/data-MWR_RIVER_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=RIVER:RIVER&MAXFEATURE=2&Filter= (<Filter>
	<ogc:PropertyIsEqualTo xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>PROV</ogc:PropertyName>
	<ogc:Literal>420000</ogc:Literal></ogc:PropertyIsEqualTo></Filter>)
	
	
	(<Filter><ogc:Within><ogc:PropertyName>SHAPE</ogc:PropertyName></ogc:Within></Filter>)

	/*  行政区界断面  */
	http://10.1.3.140/iserver/services/data-MWR_AB_5T_WFS/wfs100/utf-8?VERSION=1.0.0&SERVICE=WFS&REQUEST=GetFeature&TYPENAME=AB:AB&&Filter=(<Filter><ogc:PropertyIsLike wildCard="*" singleChar="?" escapeChar="\" xmlns:ogc="http://www.opengis.net/ogc"><ogc:PropertyName>ID</ogc:PropertyName><ogc:Literal>36*</ogc:Literal></ogc:PropertyIsLike></Filter>)

	

	