---
layout: post
title: 关于JDK1.8的`HashMap`的探究
description: "JDK源码有毒"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Java ,JDK源码分析]
imagefeature: 
comments: true
share: true
---

## HashMap的几个疑问

关于`HashMap`源码分析的文章满大街都是,如果没干货,我也不会再写个这个.

大家都知道,`HashMap`是个散列表,笼统的说是就是数组(`hash table`)+链表/红黑树的形式.通过`key`的`hashcode`来确定在`hash table`中的位置,如果计算出来的位置相同,那么会将对应位置转换为链表,`JDK1.8`中如果链表上元素个数超过8个(**实际上并不完全是**),会转换为红黑树.

先提出以下几个问题,如果:

1. 链表的数据结构是怎样的(单向链表还是双向链表)?
2. `HashMap`如何根据`key`的`hashcode`计算位置的?为什么是这样做?加入扰动函数的目的又是什么?
3. 为什么说发生大规模的hash碰撞,会导致`HashMap`的性能严重下降.
4. 在链表的长度为8时,一定会转换为红黑树吗?
5. 链表转换为红黑树的临界长度为什么是8?
6. 使用红黑树的优点在哪里?
7. 红黑树会不会再转成链表?
8. 为什么说数据量大时,最好在初始化时手动指定`HashMap`大小
9. 使用`HashMap`时,需要注意什么

实际上这几个问题的答案,大部分都可以在`HashMap`的`put`方法中找到.

直接看代码:

    // put方法
    public V put(K key, V value) {    
        return putVal(hash(key), key, value, false, true);
    }

	/**
     * 实现了 Map.put 以及其它相关的方法.
     *
     * @param hash key的hash值
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent true:在key相同时,不会覆盖原有的value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
		// hash table 对象 
       Node<K,V>[] tab;
		// 下标i对应的Node对象
		Node<K,V> p;
		// hash table的长度 
		int n; 
		// key在hash table中存放的索引下标
		int i;
		// 获取hash table的长度
       if ((tab = table) == null || (n = tab.length) == 0){
			n = (tab = resize()).length;
		}
        
		// i = (n - 1) & hash  根据hash值计算key在hash table中的位置
		// 那么根据这行代码可以得到个结论:如果key为null(此时对应的hash为0),那么一定是在下标为0的位置
        if ((p = tab[i = (n - 1) & hash]) == null){
			// 如果下标i的位置是null(尚未有元素),那么直接放入
			tab[i] = newNode(hash, key, value, null);
		} else {
			// 
            Node<K,V> e; K k;
            if ( p.hash == hash
                 && ((k = p.key) == key || (key != null && key.equals(k)))
			){
				// 如果p的key和输入的key相等
				e = p;
			} else if (p instanceof TreeNode){
				// 此处已经是个红黑树了,继续往红黑树里增加新元素
				e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
			} else {
				// 如果是个传统的列表对象
                for (int binCount = 0; ; ++binCount) {
					// 此if的目的是找到位置i上的链表/红黑树的最后一个Node元素
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
						// TREEIFY_THRESHOLD = 8
						if (binCount >= TREEIFY_THRESHOLD - 1) {
							// 转换为红黑树(如果hash table的长度不到MIN_TREEIFY_CAPACITY即64,那么只是做扩容处理,并不会转换为红黑树)
							treeifyBin(tab, hash);
						}// -1 for 1st
                        break;
                    }
					// 如果链表上已有相同key的Node,那么直接返回此Node
                    if ( e.hash == hash 
						 &&
                         ((k = e.key) == key || (key != null && key.equals(k)))){
						break;
					}
                        
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null){
					e.value = value;
				}    
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

    
## 链表的数据结构是怎样的

`HashMap`中,每个元素实际都是一个`Node`(定义在`HashMap`内)对象.具体代码如下:

    static class Node<K,V> implements Map.Entry<K,V> {
            // 缓存key的hash值
            final int hash;
            final K key;    
            V value;
            // 下一个Node引用
            Node<K,V> next;
            // 唯一的一个构造函数
            Node(int hash, K key, V value, Node<K,V> next) {        
                this.hash = hash;        
                this.key = key;        
                this.value = value;        
                this.next = next;    
            }
            // equals 等方法
            ...
    }    

从`Node`的定义上也可以看到,其实是个单向链表.

## `HashMap`如何根据`key`的`hashcode`计算位置的?为什么是这样做?加入扰动函数的目的又是什么?

	// 首先是在 put()方法中,调用了hash(key)
	// 注意此方法在不同的jdk版本中,具体的实现不一样
	static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

	//接着在putVal()方法中通过如下代码寻址.其中n为hash table的长度(不是HashMap的size)
	i = (n - 1) & hash

关于`hash()`中为什么要这么做,简单的说,这段代码(`(h = key.hashCode()) ^ (h >>> 16)`)叫做**扰动函数**,**增加原始hash值的随机性**,**尽可能的**避免在key的hascode设计不合理的情况下,出现大规模hash碰撞的概率.

更具体参照逼乎关于此问题的回答,很详细也很到位,我就不搬运了,强烈建议看一下: [JDK 源码中 HashMap 的 hash 方法原理是什么？ - 胖君的回答 - 知乎](https://www.zhihu.com/question/20733617/answer/111577937)

至于这里的`(n - 1) & hash`,为什么是`n-1`而不是`n`或者其他,则是因为当 `n=2^m` 时，`(n - 1) & hash = hash%n`.

恰好,`hash table`的长度就是限定的必须是`2^m`.

同时,采用位运算`&`而不是取模运算`%`,是因为位运算`&`效率更高.

## 在链表的长度为8时,一定会转换为红黑树吗?

来来来,敲黑板划重点了.这个就是自己不深入看源码,只是看网上的博客,肯定会忽略的一点.

答案是:否.

只有在`hash table`的长度(指的是数组的长度,而不是HashMap的size,这是两个概念)超过`MIN_TREEIFY_CAPACITY`也就是`64`时,才会转换为红黑树.小于`64`,则是对`hash table`进行扩容.

具体见 `treeifyBin`方法(该方法在前文中的`putVal()方法中被调用`);
	
	/**
	 * 如果hash table的长度大于64,则将指定位置上的所有节点转换为TreeNode;
	 * 否则只对hash table进行扩容.
	 */
	final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
		// MIN_TREEIFY_CAPACITY = 64
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY){
			// 扩容
			resize();
		}else if ((e = tab[index = (n - 1) & hash]) != null) {
			// 转换为红黑树节点
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }

## 链表转换为红黑树的临界长度为什么是8

这个问题可能很少会有人会注意到.

在`HashMap`源码的最开始,就已经做了说明.不过因为是夹杂在一段近百行的注释中,可能很多人都会忽略掉.

答案就是因为根据泊松分布估算,第N次往Map里put元素时,恰好导致某个链表长度增长到8而不是出现在`hash table`的其它位置的概率只有0.000006%(亿分之六).所以至少在理论上来说,在链表的长度增长到8之前,hash table早就自动扩容了.

    * 具体的概率
    * 0:    0.60653066
    * 1:    0.30326533
    * 2:    0.07581633
    * 3:    0.01263606
    * 4:    0.00157952
    * 5:    0.00015795
    * 6:    0.00001316
    * 7:    0.00000094
    * 8:    0.00000006
    * more: less than 1 in ten million

只是在实际情况下,由于我们的不当使用,或者是对`key`的`hashcode`方法做了不合适的重写,导致哈希碰撞的情况还是偶尔会撞到.

## 使用红黑树的优点在哪里

简单的说,就是为了解决出现Hash碰撞时链表的性能问题.

`HashMap`这个数据结构的设计初衷就是为了能快速的获取到指定`key`对应的`value`.

快的原因就是是`key`对应的元素在`hash table`中的下标位置,是通过`hash`值计算出来的,而不是遍历.

但是在出现大规模的`hash`碰撞时,即大量具有相同hash值(严格的说,`hash table`寻址使用的hash值,是经过计算的,具体看前文)的`key`被put进来时,那么在检索时就变成了:

1. 根据`key`的`hash`值定位到`hash table`中的某个位置
2. 由于`hash`碰撞,此位置上是个很长的列表.需要逐项遍历.

此时`HashMap`的高效性就会大打折扣.

而如果采用的是红黑树(关于红黑树的原理不在本文赘述),红黑树的特点就是,检索速度比较稳定,即便是在最坏情况下依旧很高效.
    


## 为什么说数据量大时,最好手动指定HashMap大小

比如现在有100万条数据,要放到HashMap中去.

原因有2:

1. 空间上的开销
2. 性能上的开销

### 空间上的开销

上文中有提到,`HashMap`的底层是数组+链表/红黑树来实现的.当数组被使用了75%(按默认的负载因子0.75)时就会对数组进行扩容.

而数组的存储空间是连续的.频繁的扩容,导致`HashMap`需要不停的去申请越来越大的连续的内存空间.当在堆内存中没有足够大的空闲的连续空间时,就会不停的触发GC.

### 性能上的开销

依旧是频繁的扩容导致的.

在扩容的时候,除了内存空间之外.每次扩容时,还需要将`HashMap`中所有元素按照扩容后的`hash table`大小,重新计算下位置,而红黑树也有可能因为扩容后,重新退化成链表.

这个计算量还是很大的.

## 使用`HashMap`时,需要注意什么

1. 尽可能避免频繁扩容.数据量大时,初始化时手动指定HashMap大小.
2. 尽可能避免hash碰撞.作为`HashMap`的`key`的对象的`hashcode`方法,要合理设计.
3. 不要在多线程情况下使用.`HashMap`是同步的,多线程情况下,优先考虑`ConcurrentHashMap`.
