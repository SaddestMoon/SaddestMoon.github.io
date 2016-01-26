---
layout: post
title: OGC Filter
description: "GIS就TM是个坑"
category: 成长の足迹
tags: [笔记 , 代码狗 ,GIS]
imagefeature: 
comments: true
share: true
---


12/28/2015 6:29:52 PM 

参考 

1. [http://jackyhua.iteye.com/blog/732287](http://jackyhua.iteye.com/blog/732287 "WFS XML")

声明

1. 本文中的代码,为了方便阅读,做了换行对齐.建议使用时去掉换行.
2. 不同产品对OGC标准的实现可能会略有不同,本文中的示例代码均基于*超图*SuperMap iserver 6R发布的WFS服务下进行.理论上适用超图的整个系列产品发布的WFS服务

## 1.Equal ##

关键字:PropertyIsEqualTo

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsEqualTo>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsEqualTo>

## 2.NotEqual ##

关键字:PropertyIsNotEqualTo

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsNotEqualTo>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsNotEqualTo> 

## 3.Less ##

关键字:PropertyIsLessThan

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsLessThan>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsLessThan> 

## 4.Greater ##

关键字:PropertyIsGreaterThan

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsGreaterThan>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsGreaterThan> 

## 5.LessOrEqual ##

关键字:PropertyIsGreaterThan

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsLessThanOrEqualTo>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsLessThanOrEqualTo> 

## 6.GreaterOrEqual ##

关键字:PropertyIsGreaterThanOrEqualTo

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsGreaterThanOrEqualTo>
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsGreaterThanOrEqualTo> 

## 7.Like ##

关键字:PropertyIsLike

	/* 第一个%s填写字段名称，第二个%s填写字段值 */
	<PropertyIsLike wildCard="*" singleChar="?" escapeChar="\">
		<PropertyName>%s</PropertyName>
		<Literal>%s</Literal>
	</PropertyIsLike> 

示例:

	/* 查询SWS_CD的值为以42开头的数据 */
	<PropertyIsLike wildCard="*" singleChar="?" escapeChar="\">
		<PropertyName>SWS_CD</PropertyName>
		<Literal>42*</Literal>
	</PropertyIsLike>

## 8.IsNull ##

关键字:PropertyIsNull

	/* 第一个%s填写字段名称 */
	<PropertyIsNull>
		<PropertyName>%s</PropertyName>
	</PropertyIsNull>

## 9.Between ##

关键字:PropertyIsBetween

	/* 第一个%s填写字段名称，第二个%s填写字段值下限，第三个%s填写字段值上限 */
	<PropertyIsBetween>
		<PropertyName>%s</PropertyName>
		<LowerBoundary>%s</LowerBoundary>
		<UpperBoundary>%s</UpperBoundary>
	</PropertyIsBetween>

示例:

	/* 查询SWS_CD的值介于420000000000与420000000009的数据 */
	<ogc:PropertyIsBetween>
		<ogc:PropertyName>SWS_CD</ogc:PropertyName>
		<ogc:LowerBoundary>
			<ogc:Literal>420000000000</ogc:Literal>
		</ogc:LowerBoundary>
		<ogc:UpperBoundary>
			<ogc:Literal>420000000009</ogc:Literal>
		</ogc:UpperBoundary>
	</ogc:PropertyIsBetween>

## 多个查询条件组合在一起 ##

使用AND进行连接。

例如,大于等于且小于:

	<And>
		<PropertyIsGreaterThanOrEqualTo>
			<PropertyName>%s</PropertyName>
			<Literal>%s</Literal>
		</PropertyIsGreaterThanOrEqualTo>
		<PropertyIsLessThan>
			<PropertyName>%s</PropertyName>
			<Literal>%s</Literal>
		</PropertyIsLessThan> 
	</And> 