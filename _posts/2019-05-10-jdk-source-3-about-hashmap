---
layout: post
title: 你可能不清楚的Java细节(3)--JDK1.8中HashMap在出现hash碰撞时链表长度超过8就一定会变成红黑树吗
description: "JDK源码有毒"
categories: 成长の足迹
tags: [笔记 , 代码狗 ,Java ,JDK源码分析]
imagefeature: 
comments: true
share: true
---

如题,答案是:否.

至少迄今为止零零散散看过的关于JDK1.8 `HashMap`的源码分析文章不下10个了.但印象中都是众口一词,说链表长度超过8就会转换成红黑树.但是很可惜,实际上不是的.

## 原因

核心代码如下(大体上调用关系就是 `put`->`putVal`->`treeifyBin`):
	
`putVal`方法中相关部分
	
	// putVal方法中相关部分(put方法调用了putVal())
	if (binCount >= TREEIFY_THRESHOLD - 1) {
		// 转换成红黑树,如果hash table的长度不到MIN_TREEIFY_CAPACITY即64,
		// 那么只是做扩容处理,并不会转换为红黑树
	    treeifyBin(tab, hash);
	}
`treeifyBin`方法中相关部分

	// treeifyBin方法中相关部分
	// MIN_TREEIFY_CAPACITY = 64
	if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY){
		// 扩容,不转换
		resize();
	}else if ((e = tab[index = (n - 1) & hash]) != null) {
		// 转换为红黑树节点
        TreeNode<K,V> hd = null, tl = null;
        do {
         // 具体转换逻辑
         ...
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null){
        	hd.treeify(tab);// 转换为红黑树
        }
    }

从上面两段代码可以看出来,实际上转换红黑树有个大前提,就是当前`hash table`的长度也就是`HashMap`的`capacity`(不是`size`)不能小于64.

小于64就只是做个扩容.

在不小于64的情况下,标题中的说法还有个不严谨的地方就是,在链表的长度为8时(准确的说是长度为7并且在继续塞第8个时),转换成红黑树,而不是超过8.当然,这不是语文题,死扣字眼很无聊.

## 相关方法完整源码:

### put方法
	/**
	 * HashMap的put方法
	 */
    public V put(K key, V value) {    
        return putVal(hash(key), key, value, false, true);
    }

### putVal方法
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

### treeifyBin方法
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
                hd.treeify(tab);//转换为红黑树
        }
    }

`treeify`方法是具体的转换逻辑,和本文无关,不再占用篇幅.
