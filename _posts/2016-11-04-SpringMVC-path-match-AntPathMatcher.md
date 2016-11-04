---
layout: post
title: SpringMVC路径匹配规则AntPathMatcher
description: "Spring源码有毒"
category: 成长の足迹
tags: [笔记 , 代码狗 ,Java ,Spring]
imagefeature: 
comments: true
share: true
---
# SpringMVC路径匹配规则AntPathMatcher #

# 前言 #

本文是基于Spring Framework 4.3.3分析.

# 正文 #

SpringMVC的路径匹配规则是依照Ant的来的.

实际上**不只是SpringMVC**,**整个Spring框架**的路径解析都是按照Ant的风格来的.

在Spring中的具体实现,详情参见 `org.springframework.util.AntPathMatcher`.

具体规则如下(来自Spring AntPathMatcher源码注释):

	 * {@link PathMatcher} implementation for Ant-style path patterns.
	 *
	 * <p>Part of this mapping code has been kindly borrowed from <a href="http://ant.apache.org">Apache Ant</a>.
	 *
	 * <p>The mapping matches URLs using the following rules:<br>
	 * <ul>
	 * <li>{@code ?} matches one character</li>
	 * <li>{@code *} matches zero or more characters</li>
	 * <li>{@code **} matches zero or more <em>directories</em> in a path</li>
	 * <li>{@code {spring:[a-z]+}} matches the regexp {@code [a-z]+} as a path variable named "spring"</li>
	 * </ul>
	 *
	 * <h3>Examples</h3>
	 * <ul>
	 * <li>{@code com/t?st.jsp} &mdash; matches {@code com/test.jsp} but also
	 * {@code com/tast.jsp} or {@code com/txst.jsp}</li>
	 * <li>{@code com/*.jsp} &mdash; matches all {@code .jsp} files in the
	 * {@code com} directory</li>
	 * <li><code>com/&#42;&#42;/test.jsp</code> &mdash; matches all {@code test.jsp}
	 * files underneath the {@code com} path</li>
	 * <li><code>org/springframework/&#42;&#42;/*.jsp</code> &mdash; matches all
	 * {@code .jsp} files underneath the {@code org/springframework} path</li>
	 * <li><code>org/&#42;&#42;/servlet/bla.jsp</code> &mdash; matches
	 * {@code org/springframework/servlet/bla.jsp} but also
	 * {@code org/springframework/testing/servlet/bla.jsp} and {@code org/servlet/bla.jsp}</li>
	 * <li>{@code com/{filename:\\w+}.jsp} will match {@code com/test.jsp} and assign the value {@code test}
	 * to the {@code filename} variable</li>
	 * </ul>
	 *
	 * <p><strong>Note:</strong> a pattern and a path must both be absolute or must
	 * both be relative in order for the two to match. Therefore it is recommended
	 * that users of this implementation to sanitize patterns in order to prefix
	 * them with "/" as it makes sense in the context in which they're used.


换成人话就是:

- `?` 匹配**1**个字符
- `*` 匹配**0**个或多个字符
- `**` 匹配路径中的**0**个或多个目录
- `{spring:[a-z]+}` 将正则表达式`[a-z]+`匹配到的值,赋值给名为 `spring` 的路径变量.(PS:必须是完全匹配才行,在SpringMVC中只有完全匹配才会进入`controller`层的方法)

## `?` ##

和其它几个不一样的是,`?` 要求必须为一个字符,并且不能是代表路径分隔符的`/`.

	@RequestMapping("/index?")
    @ResponseBody
    public String index(){
        System.out.println("11");
        return "11";
    }

结果:

	index     		false 404错误(必须要有一个字符)
	index/     		false 404错误(不能为"/")
	indexab     	false 404错误(不能是多个字符)
	indexa 			true  输出 11
	

## `*` ##

`*`,虽然可以匹配多个任意的字符,但是,如果你以为 `*` 可以替代 `**` 那就错了,`*` 代表的多个任意字符组成的字符串不能是个 目录 或者说 路径.也就是说,`*` 并不能拿来替代 `**`.

示例代码:

	@RequestMapping("/index*")
    @ResponseBody
    public String index(){
        System.out.println("11");
        return "11";
    }

结果:

	index     		true  输出 11(可以为0字符)
	index/     		true  输出 11(可以为"/")
	indexa 			true  输出 11(可以为1个字符)
	indexabc		true  输出 11(可以为多个字符)
	index/a	 		false 404错误("/a"是一个路径)

## `**` ##

**0**个或多个目录.`**` 代表的字符串本身不一定要包含 `/`

	@RequestMapping("/index/**/a")
    @ResponseBody
    public String index(){
        System.out.println();
        return "11";
    }

结果:

	index/a     	true  输出 11(可以为0个目录)
	index/x/a     	true  输出 11(可以为一个目录)
	index/x/z/c/a 	true  输出 11(可以为多个目录)
	
## `{spring:[a-z]+}` ##

其它的关于 `AntPathMatcher` 的文章里,对 `{spring:[a-z]+}` 的匹配大多是只字未提.这里补充下.

示例代码:

	@RequestMapping("/index/{username:[a-b]+}")
    @ResponseBody
    public String index(@PathVariable("username") String username){
        System.out.println(username);
        return username;
    }

结果:

	index/ab     	true  输出 ab
	index/abbaaa 	true  输出 abbaaa
	index/a	 		false 404错误
	index/ac	 	false 404错误

# 附录(完整测试用例) #

节选自 `AntPathMatcherTests`.不得不说 `Spring` 的测试用例写的实在是太完善了.

	// test exact matching
	assertTrue(pathMatcher.match("test", "test"));
	assertTrue(pathMatcher.match("/test", "/test"));
	assertTrue(pathMatcher.match("http://example.org", "http://example.org")); // SPR-14141
	assertFalse(pathMatcher.match("/test.jpg", "test.jpg"));
	assertFalse(pathMatcher.match("test", "/test"));
	assertFalse(pathMatcher.match("/test", "test"));

	// test matching with ?'s
	assertTrue(pathMatcher.match("t?st", "test"));
	assertTrue(pathMatcher.match("??st", "test"));
	assertTrue(pathMatcher.match("tes?", "test"));
	assertTrue(pathMatcher.match("te??", "test"));
	assertTrue(pathMatcher.match("?es?", "test"));
	assertFalse(pathMatcher.match("tes?", "tes"));
	assertFalse(pathMatcher.match("tes?", "testt"));
	assertFalse(pathMatcher.match("tes?", "tsst"));

	// test matching with *'s
	assertTrue(pathMatcher.match("*", "test"));
	assertTrue(pathMatcher.match("test*", "test"));
	assertTrue(pathMatcher.match("test*", "testTest"));
	assertTrue(pathMatcher.match("test/*", "test/Test"));
	assertTrue(pathMatcher.match("test/*", "test/t"));
	assertTrue(pathMatcher.match("test/*", "test/"));
	assertTrue(pathMatcher.match("*test*", "AnothertestTest"));
	assertTrue(pathMatcher.match("*test", "Anothertest"));
	assertTrue(pathMatcher.match("*.*", "test."));
	assertTrue(pathMatcher.match("*.*", "test.test"));
	assertTrue(pathMatcher.match("*.*", "test.test.test"));
	assertTrue(pathMatcher.match("test*aaa", "testblaaaa"));
	assertFalse(pathMatcher.match("test*", "tst"));
	assertFalse(pathMatcher.match("test*", "tsttest"));
	assertFalse(pathMatcher.match("test*", "test/"));
	assertFalse(pathMatcher.match("test*", "test/t"));
	assertFalse(pathMatcher.match("test/*", "test"));
	assertFalse(pathMatcher.match("*test*", "tsttst"));
	assertFalse(pathMatcher.match("*test", "tsttst"));
	assertFalse(pathMatcher.match("*.*", "tsttst"));
	assertFalse(pathMatcher.match("test*aaa", "test"));
	assertFalse(pathMatcher.match("test*aaa", "testblaaab"));

	// test matching with ?'s and /'s
	assertTrue(pathMatcher.match("/?", "/a"));
	assertTrue(pathMatcher.match("/?/a", "/a/a"));
	assertTrue(pathMatcher.match("/a/?", "/a/b"));
	assertTrue(pathMatcher.match("/??/a", "/aa/a"));
	assertTrue(pathMatcher.match("/a/??", "/a/bb"));
	assertTrue(pathMatcher.match("/?", "/a"));

	// test matching with **'s
	assertTrue(pathMatcher.match("/**", "/testing/testing"));
	assertTrue(pathMatcher.match("/*/**", "/testing/testing"));
	assertTrue(pathMatcher.match("/**/*", "/testing/testing"));
	assertTrue(pathMatcher.match("/bla/**/bla", "/bla/testing/testing/bla"));
	assertTrue(pathMatcher.match("/bla/**/bla", "/bla/testing/testing/bla/bla"));
	assertTrue(pathMatcher.match("/**/test", "/bla/bla/test"));
	assertTrue(pathMatcher.match("/bla/**/**/bla", "/bla/bla/bla/bla/bla/bla"));
	assertTrue(pathMatcher.match("/bla*bla/test", "/blaXXXbla/test"));
	assertTrue(pathMatcher.match("/*bla/test", "/XXXbla/test"));
	assertFalse(pathMatcher.match("/bla*bla/test", "/blaXXXbl/test"));
	assertFalse(pathMatcher.match("/*bla/test", "XXXblab/test"));
	assertFalse(pathMatcher.match("/*bla/test", "XXXbl/test"));

	assertFalse(pathMatcher.match("/????", "/bala/bla"));
	assertFalse(pathMatcher.match("/**/*bla", "/bla/bla/bla/bbb"));

	assertTrue(pathMatcher.match("/*bla*/**/bla/**", "/XXXblaXXXX/testing/testing/bla/testing/testing/"));
	assertTrue(pathMatcher.match("/*bla*/**/bla/*", "/XXXblaXXXX/testing/testing/bla/testing"));
	assertTrue(pathMatcher.match("/*bla*/**/bla/**", "/XXXblaXXXX/testing/testing/bla/testing/testing"));
	assertTrue(pathMatcher.match("/*bla*/**/bla/**", "/XXXblaXXXX/testing/testing/bla/testing/testing.jpg"));

	assertTrue(pathMatcher.match("*bla*/**/bla/**", "XXXblaXXXX/testing/testing/bla/testing/testing/"));
	assertTrue(pathMatcher.match("*bla*/**/bla/*", "XXXblaXXXX/testing/testing/bla/testing"));
	assertTrue(pathMatcher.match("*bla*/**/bla/**", "XXXblaXXXX/testing/testing/bla/testing/testing"));
	assertFalse(pathMatcher.match("*bla*/**/bla/*", "XXXblaXXXX/testing/testing/bla/testing/testing"));

	assertFalse(pathMatcher.match("/x/x/**/bla", "/x/x/x/"));

	assertTrue(pathMatcher.match("/foo/bar/**", "/foo/bar")) ;

	assertTrue(pathMatcher.match("", ""));

	assertTrue(pathMatcher.match("/{bla}.*", "/testing.html"));		
	