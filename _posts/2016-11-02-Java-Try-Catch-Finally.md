---
layout: post
title: try--catch--finall优先级总结
description: "基础啊基础"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,GIS]
imagefeature: 
comments: true
share: true
---

# 前言 #

先放上main方法,下文中的测试代码只修改test()的实现.

	public static void main(String[] args) {
        String s = test();
        System.out.println(s);
    }

# 1.出现异常后不会阻断try-catch之外语句的执行 #

测试代码

    public static String test() {
        String s = "a";
        System.out.println("test_1");
        try {
            System.out.println("try_1");
            Integer.parseInt(s);
	//            return "try_return";
        } catch (Exception e) {
            System.out.println("catch_1");
            e.printStackTrace();
            System.out.println("catch_2");
            return "catch_return";
        } finally {
            System.out.println("finally_1");
	//            return "finally_return";
        }
        System.out.println("test_2");
        return "test_return";
    }

输出:
	
	java.lang.NumberFormatException: For input string: "a"
	test_1
	try_1
	catch_1
	catch_2
	finally_1
	finally_return

# 2.finally语句里的return优先级最高 #

## 2.1 有异常时 ##

测试代码:

	public static String test() {
	        String s = "1";
	        System.out.println("test_1");
	        try {
	            System.out.println("try_1");
	            Integer.parseInt(s);
	            return "try_return";
	        } catch (Exception e) {
	            System.out.println("catch_1");
	            e.printStackTrace();
	            System.out.println("catch_2");
	            return "catch_return";
	        } finally {
	            System.out.println("finally_1");
	            return "finally_return";
	        }
	//        System.out.println("test_2");
	//        return "test_return";
	    }

输出:

	test_1
	try_1
	catch_1
	java.lang.NumberFormatException: For input string: "a"
	catch_2
	finally_1
	finally_return

## 2.2 没有异常时 ##

测试代码:

		public static String test() {
	        String s = "1";
	        System.out.println("test_1");
	        try {
	            System.out.println("try_1");
	            Integer.parseInt(s);
	            return "try_return";
	        } catch (Exception e) {
	            System.out.println("catch_1");
	            e.printStackTrace();
	            System.out.println("catch_2");
	            return "catch_return";
	        } finally {
	            System.out.println("finally_1");
	            return "finally_return";
	        }
	//        System.out.println("test_2");
	//        return "test_return";
	    }

输出:

	test_1
	try_1
	finally_1
	finally_return

# 3.总结 #

我们将上面的代码简化下:

	Object test() {
	        外部代码1
	        try {
	            try代码块
	            tryReturn
	        } catch (Exception e) {
	            catch代码块
	            catchReturn
	        } finally {
	            finally代码块
	            finallyReturn
	        }
	        外部代码2
	        外部代码Return
	    }

当然事实上也不可能同时存在这么多return语句,因为编译都不会通过.

代码的执行流程是:

出现异常时:

外部代码1-->try代码块-->catch代码块-->finally代码块-->finallyReturn-->catchReturn-->外部代码2-->外部代码Return

没有异常时:

外部代码1-->try代码块-->finally代码块-->finallyReturn-->tryReturn-->外部代码2-->外部代码Return

中间任何一个环节return了,那就不再执行剩下的语句.