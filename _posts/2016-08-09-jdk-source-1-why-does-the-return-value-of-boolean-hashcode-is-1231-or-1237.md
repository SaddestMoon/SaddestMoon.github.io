---
layout: post
title: 你可能不清楚的Java细节(1)--为什么Boolean的hashCode()方法返回值是1231或1237
description: "JDK源码有毒"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Java ,JDK源码分析]
imagefeature: 
comments: true
share: true
---
## 问题
本文中涉及到的JDK源码基于JDK1.8_102.

Boolean的hashCode()方法源码如下:

    public static int hashCode(boolean value) {
        return value ? 1231 : 1237;
    }

为什么是1231、1237？很奇怪是不是?

## 结论 ##

直接先上结论,因为1231和1237是两个比较大的素数(质数),实际上这里采用其它任意两个较大的质数也是可以的.而采用1231和1237更大程度的原因是Boolean的作者个人爱好(看到这句别打我).

如果你看懂了上面这段话,或者你对hash碰撞有了解,并且清楚计算hashCode的散列算法设计原则,以及为什么常见的散列算法大都采用了31作为乘法因子,那么可以直接关闭这篇文章.

如果你没看懂或者不清楚,那下面我来一步一步来剖析,能用JDK源码的我尽量用源码.

## 分析 ##

### hashCode是用来干嘛的 ###

首先先要明白,hashCode是用来干嘛的.

    /*
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those  provided by
     * {@link java.util.HashMap}.
     * ...
     * /

以上内容来自java.lang.Object的hashCode()方法.

根据这个,我们得出结论,hashCode的目的就是**为了提高诸如java.util.HashMap这样的散列表(hash table)的性能**.

### hashCode是怎样影响散列表的性能的--以HashMap为例分析 ###

既然是服务于hash table,那么我们干脆就以上面注释里面提到的也是最常用的HashMap为例来说明.

通过查看源码,可以清楚的看到hashCode主要是在对HashMap进行put和get的时候使用,就以put来说明,毕竟没put哪来get.

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

调用的是putVal方法,再看putVal.下面是putVal的源码,可以先跳过直接往下看.

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

其实只看这两行就行.

        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else{
            //省略..
        }

其中:
    
    (n - 1) & hash == hash % n; 

其中hash就是我们调用put(key,value)方法时候的key,table是HashMap实际存储时用的对象,本质上是个散列表,n为table的size.

HashMap在设计的时候,是希望put进去的对象均匀的分布在table里,即散列表table里第i个位置至多存放一个put对象,
在这个时候进行查找效率是最优的,因为只需要根据key的hashCode,计算出i，table[i]就是要查找的value.

但是,如果table[i]存放的对象超过了一个,table[i]的地方就会变成一个链表,在根据key的hashCode找到table[i]后还要在遍历一遍数组才能找到对应的对象,效率就会变得比较糟糕,假设一种比较极端的情况,所有的对象都放在table[0]的位置,那么整个HashMap就相当于变成了一个链表,这就完全背离了HashMap的设计初衷.

在put时,i就是上面这段代码计算出的,可以看到在不考虑table扩容的情况下,唯一的决定因素就只有hash(put进来的key的hashCode).

那么我们就得出个结论,hashCode会决定对象在HashMap中的存放位置,以此影响性能,只要保证key的hashCode不重复,就能避免出现hash碰撞的情况,进而保证HashMap性能最优.

问题又来了,如何保证hashCode尽可能的不重复呢,这就是下面要说的问题.

### hashCode散列算法设计简要分析 ###

一般hashCode()方法的实现里其实就是一个散列算法,每个对象的hashCode()方法都不尽相同.

先来看下hashCode散列算法的原则,依旧来自java.lang.Object的hashCode()方法的注释.我就不贴原文了,直接上译文:

     *<li>在一个Java应用运行期间,只要一个对象的{@code equals}方法所用到的信息没有被修改,
     *     那么对这同一个对象调用多次{@code hashCode}方法,都必须返回同一个整数.
     *     在同一个应用程序的多次执行中,每次执行所返回的整数可以不一样.
     * <li>如果两个对象根据{@code equals(Object)}方法进行比较结果是相等的,那么调用这两个对象的
     *     {@code hashCode}方法必须返回同样的结果.
     * <li>如果两个对象根据{@link java.lang.Object#equals(java.lang.Object)}进行比较是不相等的,
     *     那么并<em>不</em>要求调用这两个对象的{@code hashCode}方法必须返回不同的整数结果.
     *     但是程序员应该了解到,给不相等的对象产生不同的整数结果,可能提高散列表(hash tables)的性能.

简要的说,在一个应用的一次运行期间,equals为true的对象必须返回相同的hashCode,而equals为false的对象不是必须返回不同的hashCode.但是应当让不同的对象返回不同的hashCode.

再来扒一扒JDK的源码,比如String的是这样实现的:

    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
IntBuffer中是这样实现的:

    public int hashCode() {
        int h = 1;
        int p = position();
        for (int i = limit() - 1; i >= p; i--)

            h = 31 * h + get(i);

        return h;
    }

再比如Array中,其中一个的实现是这样的:

    public static int hashCode(int a[]) {
        if (a == null)
            return 0;

        int result = 1;
        for (int element : a)
            result = 31 * result + element;

        return result;
    }

而利用开发工具生成的hashCode方法,比如IDEA生成的,是这样的:

    @Override
    public int hashCode() {
        int result = id != null ? id.hashCode() : 0;
        result = 31 * result + (enable != null ? enable.hashCode() : 0);
        result = 31 * result + (remark != null ? remark.hashCode() : 0);
        result = 31 * result + (createBy != null ? createBy.hashCode() : 0);
        result = 31 * result + (createTime != null ? createTime.hashCode() : 0);
        result = 31 * result + (updateBy != null ? updateBy.hashCode() : 0);
        result = 31 * result + (updateTime != null ? updateTime.hashCode() : 0);
        return result;
    }

eclipse生成的跟IDEA的基本一样,不再浪费篇幅.

可以看到这些散列算法在计算hashCode的时候,除了对组成该对象的每个因子进行循环迭代(实际上这些参与循环迭代的因子也是该对象的equals方法进行比较时要参与对比的)--例如String的hashCode方法中对字符串的每个字符进行了循环迭代,IntBuffer中对每一位进行了循环迭代--之外,还有一个很明显的地方就是,每次循环的时候,都对前一次循环计算出的hashCode乘以31.

这些散列算法的结果等价于下面这个表达式:
    
    s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]

其中s[n]为参与散列计算的第n个因子.

### 为什么是31? ###

因为31是一个素数.

那么为什么要采用素数?为什么要是31而不是其它素数?

一个一个来.

#### 为什么是素数 ####

还是拿最常用的HashMap来说,还记得前面说的put(key,value)方法中,table[i]中下标i是怎么得来的不？忘了没关系,代码再贴下:

    if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else{
            //省略..
        }

下标i就是`(n - 1) & hash`,它等价于`hash % n`,那么要保证`hash % n`尽可能的不一样.

鉴于一般情况下n为偶数(下文中会有说明),那么就需要尽可能的保证

对于hash的值,前面说过,常用的散列算法的计算结果等价于这个:

    s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
    //用常量M替换下31就是
    s[0]*M^(n-1) + s[1]*M^(n-2) + ... + s[n-1]
    //即
    hash =  s[0]*M^(i-1) + s[1]*M^(i-2) + ... + s[i-1]

对于n的取值

继续看HashMap源码,HashMap的构造方法:

    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

内在实现是调用了tableSizeFor方法:
    
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

这段代码的作用就是检查n,如果n为2^x,那么就返回n,否则就返回比n大的最小的正整数2^x,当然这一切都要保证不超出MAXIMUM_CAPACITY即2^30.

这就意味着,存放HashMap数据的散列表的大小(不是HashMap的size),一定是2的x次方.

这样我们的问题就变成了:

    hash =  s[0]*M^(i-1) + s[1]*M^(i-2) + ... + s[i-1];
    n = 2^x;
    求在hash % n余数最多的情况下M的取值.

M肯定是越大越好,而素数在做取模运算时,余数的个数是最多的.
而如果M为素数,上面的hash的计算表达式里相当于每项都有了素数,那么`hash % n`时也就近似相当于素数对n取模,这个时候余数也就会尽可能的多.关于这个结论我是找了个数学系的博士特意求证了下.

那么我们就可以得出个结论,常量M越大越好,且需要是个素数.

#### 为什么是31 ####

素数那么多,为什么要选择31？

根据为什么是素数这一部分的结论,我们应该选择一个尽可能大的素数,但是实际上,又不能选择太大的素数.原因在于hashCode的值为int类型,计算结果不能溢出啊.

所以这个素数不能太小,但也不能太大.

选择31,除了Effective Java一书中提到的计算机计算31比较快(可以直接采用位移操作得到 1<<5-1)之外,个人认为还有一个原因:Java是美帝人编写的语言,再加上大多数情况下我们都是采用String作为key,曾有人对超过5W个英文单词做了测试,在常量取31情况下,碰撞的次数都不超过7次(本人没去验证).采用31已经足够了.


#### 为什么Boolean的又是1231和1237 ####

回到本文标题提出的问题,为什么会是1231和1237,Boolean只有true和false两个值,为什么不能是3或者7,或者其它的素数?

诚然,Boolean只有true和false两个值,理论上任何两个素数都可以.但是在实际使用时,可能作为key的不只是Boolean一种类型啊,可能还会有其它类型,比如最常见的字符串作为key,还有int作为key.至少要保证避开常见hashCode的取值范围吧,Integer还缓存了常用的256个数字着呢...但是太大了也没意义,比如说字符串"00"的hashCode为1536,Boolean的hashCode取值太大的话,指不定又跟字符串的hashCode撞上了,更别说其它对象的了.

所以Boolean的hashCode取值也是一个不能太小也不能太大的事情,至于取值1231和1237就真的没有什么数学上的依据了,更大程度上就是Boolean作者个人爱好罢了.
