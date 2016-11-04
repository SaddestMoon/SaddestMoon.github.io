---
layout: post
title: StringTokenizer、split、substring对比
description: "笨就要多读书"
category: 成长の足迹
tags: [笔记 , 代码狗 ,Java]
imagefeature: 
comments: true
share: true
---


# StringTokenizer、split、substring对比 #

对String进行分割,JDK提供了三种方法:分别是 `java.lang.String`的 `split`方法和`substring`方法,以及`java.util.StringTokenizer`类.

最常用的就`String`自带的两种方法,`StringTokenizer`极为少见.

下面就来对比下这三种的用法,分析下优缺点.

在此事先声明,本文代码示例基于JDK1.8_102.

JDK1.6的substring方法有缺陷,在大量使用大量频繁使用substring的时候会造成严重的性能问题,1.7+已修复,故本文一些结论在1.6并不适用.

# 1.用法介绍 #

##  1.1 split方法 ##

split是基于正则表达式的字符串分割,一共有两个重载方法: `split(String regex)` 和 `split(String regex, int limit)`.

相信很多人在使用时,通常就是简单的使用 `split(String regex)`,对 `split(String regex, int limit)` 很少有了解.实际上 `split(String regex)` 是调用了 `split(regex, 0)` 来进行的实现.

我们就来详细的说下 `split(String regex, int limit)`.

对于第一个参数 `regex`,即所谓的正则表达式字符串.但是查看了源码我们就会发现,如果 `regex`是非正则表达式元字符的(指".$\|()[{^?*+\\")单字符字符串,或者是长度为2且第一个字符为"\"第二个字符为非数字非大小写字母的字符时,`split`实际上是借助的substring来进行的实现,这样会有更好的性能上的表现;否则才会使用正则表达式进行匹配.

对于第二个参数 `limit`,即参数 `regex`的匹配次数(最多匹配`limit-1`次,前提`limit`为正整数),或者说是结果数组的最大长度(最大长度`limit`).此参数的可取值可以为包括负数和0在内的**任意整数**.

假定`limit`为0时,结果数组的长度为len,那么我们将`limit`的取值范围划分下区间如下:

- 小于0,`regex`匹配尽可能多的次数,结果数组的末尾包含空元素
- 等于0,`regex`匹配尽可能多的次数,结果数组的末尾不包含空元素.
- 大于0小于等于len,`regex`匹配`limit-1`次,结果数组长度为`limit`,结果数组的最后一个元素为剩下的尚未进行正则匹配的字符串.
- 大于len,`regex`匹配`len-1`次,结果数组长度为`len`,结果数组的末尾包含空元素

代码示例如下:

		String s1 = ";ABBD;;;AS;D;;";
        String[] result;

        //--------------split(regex)-----------------
        result = s1.split(";");
        System.out.print("split(regex)                      基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");


        //--------------split(regex,limit==0)-----------------
        result = s1.split(";",0);
        System.out.print("split(regex,limit==0)    limit:0  基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");

        //--------------split(regex,0<limit<len)
        result = s1.split(";",4);
        System.out.print("split(regex,0<limit<len) limit:4  基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");

        //--------------split(regex,limit==len)-----------------
        result = s1.split(";",6);
        System.out.print("split(regex,limit==len)  limit:6  基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");

        //--------------split(regex,limit<0)-----------------
        result = s1.split(";",-1);
        System.out.print("split(regex,limit<0)     limit:-1 基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");

        //--------------split(regex,limit>len)-----------------
        result = s1.split(";",10);
        System.out.print("split(regex,limit>len)   limit:10 基准len:6 结果数组长度:"+result.length+"  数组元素: ");
        for(int i = 0;i<result.length;i++){
            System.out.print(i+":"+result[i]+" ");
        }
        System.out.print("\n");

输出结果如下:

	split(regex)                      基准len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 
	split(regex,limit==0)    limit:0  基准len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 
	split(regex,0<limit<len) limit:4  基准len:6 结果数组长度:4  数组元素: 0: 1:ABBD 2: 3:;AS;D;; 
	split(regex,limit==len)  limit:6  基准len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D;; 
	split(regex,limit<0)     limit:-1 基准len:6 结果数组长度:8  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 7: 
	split(regex,limit>len)   limit:10 基准len:6 结果数组长度:8  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 7: 

## 1.2 substring方法 ##

substring方法也有两个重载的方法:`substring(int beginIndex)` 和 `substring(int beginIndex, int endIndex)`.

不像大多数重载的方法,`substring(int beginIndex)` 在实际的实现上并没有去调用 `substring(int beginIndex, int endIndex)`,只是两者的实现代码比较类似.

也不同于`split`和`StringTokenizer类`--这两个都是有近乎标准化的实现,`substring`通常需要我们自己写代码再进行多次循环,反复的截取才能获取到想要的结果,所以其性能什么的,和我们自身的代码实现有很大的关系.

以下仅作一个demo:

		String s1 = ";ABBD;;;AS;D;;";
        String subStr = s1;
        List<String> result = new ArrayList<>();
        int index;
        while ((index = subStr.indexOf(";")) >= 0) {
            result.add(subStr.substring(0, index));
            subStr = subStr.substring(index + 1);
        }
        System.out.print("substring                               结果数组长度:" + result.size() + "  数组元素: ");
        for (int i = 0; i < result.size(); i++) {
            System.out.print(i + ":" + result.get(i) + " ");
        }
        System.out.print("\n");

输出结果如下:

	substring                               结果数组长度:7  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 

## 1.3 StringTokenizer类 ##

`StringTokenizer`类就很少见到有人使用了,相当生僻的一个类.有`java.util`包提供.

首先声明下,`StringTokenizer`的返回值的每一项被称为标记(token).

关于其API,可以参见Oracle提供的JDK1.8的 [StringTokenizer API](http://docs.oracle.com/javase/8/docs/api/java/util/StringTokenizer.html "JDK1.8 StringTokenizer API"),或者也可以参照OSChina提供的JDK1.6中文版的 [StringTokenizer API](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/StringTokenizer.html "JDK1.6 StringTokenizer API").1.8和1.6相比没有什么变化.

其用法也相当的简单,通过构造函数创建`StringTokenizer`的实例,然后循环获取结果.

构造函数:

- `StringTokenizer(String str)`
- `StringTokenizer(String str, String delim)`
- `StringTokenizer(String str, String delim, boolean returnDelims)`

第一个参数`str`即要分割的字符串,第二个参数`delim`即分隔符,缺省值`" \t\n\r\f"`(空白字符、制表符、换行符、回车符和换页符),第三个参数`returnDelims`表示分隔符是否作为标记(即token,返回结果的每一项被称为token)返回.

主要方法:

- `boolean hasMoreTokens()` 顾名思义,判断是否还有未返回的token.
- `String nextToken()` 返回字符串中的下一个标记
- `String nextToken(String delim)` 字符串采用指定的`delim`的下一个标记
- `int countTokens()` 返回的是**剩余的**标记数

**需要注意的是**,每调用一次`nextToken()`时,`countTokens()`的返回值就会减1,也只有在调用`nextToken()`时,才会进行字符串的分割.

多余方法:

- `Object nextElement()` 对应 `nextToken()`,只是返回值类型不同
- `boolean hasMoreElements()` 对应 `hasMoreTokens()`

这两个方法纯粹是为了实现`Enumeration`接口而加的,可以无视.

用法示例如下:
		
		String s1 = ";ABBD;;;AS;D;;";
        StringTokenizer stringTokenizer;
        int i = 0;

        stringTokenizer = new StringTokenizer(s1);
        System.out.print("stringtokenizer(str)              		结果数组长度:" + stringTokenizer.countTokens() + "  数组元素: ");
        while (stringTokenizer.hasMoreTokens()) {
            System.out.print(i++ + ":"+stringTokenizer.nextToken()+" ");
        }
        System.out.print("\n");

        stringTokenizer = new StringTokenizer(s1, ";");
        System.out.print("stringtokenizer(str,delim)         		结果数组长度:" + stringTokenizer.countTokens() + "  数组元素: ");
		i = 0;
        while (stringTokenizer.hasMoreTokens()) {
            System.out.print(i++ + ":"+stringTokenizer.nextToken()+" ");
        }
        System.out.print("\n");

		stringTokenizer = new StringTokenizer(s1, ";",false);
        System.out.print("stringtokenizer(str,delim,false)        结果数组长度:" + stringTokenizer.countTokens() + "  数组元素: ");
        i = 0;
        while (stringTokenizer.hasMoreTokens()) {
            System.out.print(i++ + ":"+stringTokenizer.nextToken()+" ");
        }
		System.out.print("\n");

        stringTokenizer = new StringTokenizer(s1, ";",true);
        System.out.print("stringtokenizer(str,delim,true)         结果数组长度:" + stringTokenizer.countTokens() + " 数组元素: ");
        i = 0;
        while (stringTokenizer.hasMoreTokens()) {
            System.out.print(i++ + ":"+stringTokenizer.nextToken()+" ");
        }
		System.out.print("\n");

输出结果如下:

	stringtokenizer(str)              		结果数组长度:1  数组元素: 0:;ABBD;;;AS;D;; 
	stringtokenizer(str,delim)         		结果数组长度:3  数组元素: 0:ABBD 1:AS 2:D 
	stringtokenizer(str,delim,false)        结果数组长度:3  数组元素: 0:ABBD 1:AS 2:D 
	stringtokenizer(str,delim,true)         结果数组长度:10 数组元素: 0:; 1:ABBD 2:; 3:; 4:; 5:AS 6:; 7:D 8:; 9:; 

# 2.返回值对比 #

我们将前面的三个示例的输出结果,放到一起对比下:

	//s1 = ";ABBD;;;AS;D;;"
	split(regex)                      len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 
	split(regex,limit==0)    limit:0  len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 
	split(regex,0<limit<len) limit:4  len:6 结果数组长度:4  数组元素: 0: 1:ABBD 2: 3:;AS;D;; 
	split(regex,limit==len)  limit:6  len:6 结果数组长度:6  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D;; 
	split(regex,limit<0)     limit:-1 len:6 结果数组长度:8  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 7: 
	split(regex,limit>len)   limit:10 len:6 结果数组长度:8  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 7: 
	substring                               结果数组长度:7  数组元素: 0: 1:ABBD 2: 3: 4:AS 5:D 6: 
	stringtokenizer(str)              		结果数组长度:1  数组元素: 0:;ABBD;;;AS;D;; 
	stringtokenizer(str,delim)         		结果数组长度:3  数组元素: 0:ABBD 1:AS 2:D 
	stringtokenizer(str,delim,false)        结果数组长度:3  数组元素: 0:ABBD 1:AS 2:D 
	stringtokenizer(str,delim,true)         结果数组长度:10 数组元素: 0:; 1:ABBD 2:; 3:; 4:; 5:AS 6:; 7:D 8:; 9:; 

可以看到,可以说是没有一个方法的结果是一样的.

通常情况下,对于字符串中的开头和中间部分的**两个或多个相邻的**分隔符,`split` 和 `substring` 都选择了处理为空字符串(`""`)返回,当然`substring`是我们自己选择了处理为`""`,而`split`是JDK直接处理成为了`""`.而`StringTokenizer`则是完全无视.

其实最好还是直接看上面的这个输出结果的对比,自己再写写demo试试,用文字是很难描述清楚的.

# 3.性能测试 #

## 3.1 初步测试 ##

		private static long[] compare(String str) {
        //--------------split(regex)-----------------
        long begin1 = System.nanoTime();
        str.split(";");
        long end1 = System.nanoTime();
        long total1 = end1 - begin1;
		//System.out.println(total1);

        //--------------substring-----------------
        String subStr = str;
        long begin2 = System.nanoTime();
        int index;
        while ((index = subStr.indexOf(";")) >= 0) {
            subStr.substring(0, index);//截取,获取结果
            subStr = subStr.substring(index + 1);//获取剩余字符串
        }
        long end2 = System.nanoTime();
        long total2 = end2 - begin2;
		//System.out.println(total2);

        //--------------substring-----------------
        long begin3 = System.nanoTime();
        StringTokenizer stringTokenizer = new StringTokenizer(str, ";");
        while (stringTokenizer.hasMoreTokens()) {
            stringTokenizer.nextToken();
        }
        long end3 = System.nanoTime();
        long total3 = end3 - begin3;
		//System.out.println(total3);
        return new long[]{total1,total2,total3};
    }

    /**
     *
     * @param length 构造字符串的长度
     * @param times 测试次数
     * @return
     */
    private static List<long[]> test(int length,int times) {
        List<long[]> result = new ArrayList<>(times);
        String testStr = RandomStringUtils.random(length, 'a', 'b', 'c', ';');
        while (times-- > 0) {
            result.add(compare(testStr));
        }
        return result;
    }

    /**
     * 计算平均值.
     * @param list
     * @return
     */
    private static List<Long> countAvg(List<long[]> list){
        List<Long> result = new ArrayList<>();
        //因为list中有些值明显比另外一些大出一个数量级,
        //对平均结果很有影响,故计算平均值时,会先去掉最大的和最小的,尽量保证相对准确
        int size = list.size();
        long t1 = 0;
        long t2 = 0;
        long t3 = 0;
        long[] arr1 = new long[size];
        long[] arr2 = new long[size];
        long[] arr3 = new long[size];
        for(int i = 0;i<size;i++){
            arr1[i] = list.get(i)[0];
            arr2[i] = list.get(i)[1];
            arr3[i] = list.get(i)[2];
        }
        Arrays.sort(arr1);
        Arrays.sort(arr2);
        Arrays.sort(arr3);
        //去掉最大的和最小的
        for(int i = 4;i<size-4;i++){
            t1+=arr1[i];
            t2+=arr2[i];
            t3+=arr3[i];
        }
        //计算平均值
        result.add(t1/(size-8));
        result.add(t2/(size-8));
        result.add(t3/(size-8));
        return result;
    }

    public static void main(String[] args) {
        int[] lengthArr = {10,100,1_000,10_000,100_000,200_000,300_000};
        int times = 28;
        for(int length : lengthArr){
            System.out.println("---------------------"+length+"---------------------");
            System.out.println(countAvg(test(length,times)));
        }
    }

大体上就是先借助`org.apache.commons.lang3.RandomStringUtils`构建指定长度的字符串,然后对这同一个字符串做多次(28次)反复的测试.去掉结果中最大4组和最小4组的结果,剩下的20组结果求了个平均值,尽可能的保证结果的准确.

其中`substring`就采用前文中的示例的代码来做验证.

结果如下,依次是 `split`、`substring`、`stringtokenizer`.**单位纳秒**:

	---------------------10---------------------
	[3801, 577, 1636]
	---------------------100---------------------
	[12158, 7683, 10682]
	---------------------1000---------------------
	[43901, 108142, 32689]
	---------------------10000---------------------
	[263923, 4143798, 108206]
	---------------------100000---------------------
	[812864, 277557016, 632573]
	---------------------200000---------------------
	[1524611, 1112681830, 1241102]
	---------------------300000---------------------
	[2296348, 2526878872, 1853160]

可以看到当字符串的长度达到30万时,`split`和 `stringtokenizer`的性能并无明显的差距,而`substring`的耗时则是已经达到了秒的级别,高出1000倍以上.

但是在字符串比较短,可能从测试来看,可能就是在两三百以内时,`substring`的性能是完全要优于其它两者的,但是依旧不是很明显的差距--毕竟上面的数据的单位是纳秒,或者说是百万分之一毫秒.而可以说99%的情况下我们遇到的字符串都不会超过这个长度,所以三种方法用哪种都行,当然从代码的简洁角度来看,优先`split`.

## 3.2 长字符串测试(长度1000万以上) ##


在3.1测试中,由于`substring`的性能实在是太糟糕(**当然也有可能是3.1代码中 `substring` 的测试代码写的有问题,对结果造成了明显的影响,如果有谁发现请指正**),在字符串长度30万的时候,做28次反复测试累计所需的时间已经达到了1分钟以上,而`split`和 `stringtokenizer`尚在微秒级别,所以接下来我们就只针对这两个做测试.

依旧是3.1的代码,稍作修改,测试下长度1000万以上的字符串的性能,直接上结果(字符串长度1000万~1亿),**单位毫秒**:

	//单位毫秒,前者为split,后者为stringtokenizer
	---------------------10000000---------------------
	[92, 60]
	---------------------20000000---------------------
	[190, 117]
	---------------------30000000---------------------
	[380, 177]
	---------------------40000000---------------------
	[935, 235]
	---------------------50000000---------------------
	[1061, 360]
	---------------------60000000---------------------
	[1252, 750]
	---------------------70000000---------------------
	[1688, 792]
	---------------------80000000---------------------
	[3731, 818]
	---------------------90000000---------------------
	[2202, 978]
	---------------------100000000---------------------
	[2295, 1055]

转换成折线图如下:

![折线图](http://i.imgur.com/YsEWwBG.png)

比较奇怪的是,反复测了好几遍,`split`都是在长度为八千万时,耗时最长.在长度超过1亿的时候,也有类似的情况,不过从总体来看耗时还是在不断增长的.

但是依旧不影响结论,也就是说,在分割很长的字符串(测试用的是长度超过千万的字符串,实际上在百万级别左右就应该考虑优先使用`StringTokenizer`)时,应当优先使用`StringTokenizer`.

## 3.3 结论 ##

1. 在字符串长度较短时,也就是我们绝大多数情况下接触到的,长度在几十、几百以内,`split`、`substring`、`stringtokenizer` 的性能差距是在纳秒级别,可随意选择.
2. 在字符串长度在几百以上,几十万以下这个级别,选择使用`split`和`StringTokenizer`(`substring`的耗时比其它两者要高出一到数个数量级),其中整体来看`StringTokenizer`性能上略占优势,但是由于本身耗时就在1毫秒之内,所以优势几乎可以忽略.从代码简洁易读的角度来看,应当优先采用`split`.
3. 在处理很长的字符串时,尤其是长度超过百万甚至是千万级别,性能的差距已经达到了毫秒级甚至秒级,应当优先选择`StringTokenizer`.

需要注意的是,三种方法的返回值是不一样的,详情参见前文第二节.

# 4.题外话:关于JDK不提倡使用StringTokenizer #

JDK中关于`StringTokenizer`的描述中提到,不建议使用这个类来进行字符串分割,提倡使用的是 `java.lang.String`的 `split`方法.

	StringTokenizer is a legacy class that is retained for compatibility reasons although its use is discouraged in new code.
	It is recommended that anyone seeking this functionality use the split method of String or the java.util.regex package instead.

官方并没有说明原因.

个人猜测可能是如下原因:

1. 最主要的原因.`substring`对于多个分隔符,比如`","`、`";"`、`"#"`等同时做分隔符时,代码就会变得相当冗杂.stringtokenizer虽然是支持多个分隔符,但是分隔符必须为单个字符,也就是说,`",;#"`会被认为是3个单字符的分隔符,但是不能看做一个整体,更别说单字符与多字符的分隔符同时存在的情况.但是`split`则完全不存在前两者的问题,由于使用的是正则表达式,所以`split`支持任何格式、任何数量长度的分隔符.
2. 在绝大多数场景下,我们处理的都是很短的字符串,此时`split`和`StringTokenizer`的性能差距可以忽略.
3. `split`使用最为简单,作为`String`自身提供的方法,直接调用就可返回结果,不像其它两个还需要额外的处理.

既然这样那JDK为何还要提供`StringTokenizer`这么一个实现?主要是因为,Java对正则表达式的支持以及`split`方法都是在JDK1.4才提供的,而`StringTokenizer`是作为早期的JDK(从1.0就已经存在了)的一个"临时性的"分割字符串的解决方案.