---
layout: post
categories: 并发
title: 阿粉昨天说我动不动就内存泄漏，我好委屈...
tagline: by 郑璐璐
tags: 
  - 郑璐璐
---
大家好，我是 ThreadLocal ，昨天阿粉说我动不动就内存泄漏，我蛮委屈的，我才没有冤枉他嘞，证据在这里：  [ThreadLocal 你怎么动不动就内存泄漏？](https://mp.weixin.qq.com/s/S0IwbXadRgZ86fFLSFObVQ)

<!-- more -->

因为人家明明也考虑到了很多情况，做了很多事情，保证了如果没有 remove ，也有对 key 值为 null 时进行回收的处理操作

啥？你竟然不信？我 ThreadLocal 难道会骗你么

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/04-委屈.gif)

今天为了证明一下自己，我打算从组成的源码开始讲起，在 get ， set 方法中都有对 key 值为 null 时进行回收的处理操作，先来看 set 方法是怎么做的

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/05-开始你的表演.gif)

## set

下面是 set 方法的源码：

```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
        // 如果 e 不为空,说明 hash 冲突,需要向后查找
        e != null;
        // 从这里可以看出, ThreadLocalMap 采用的是开放地址法解决的 hash 冲突
        // 是最经典的 线性探测法 --> 我觉得之所以选择这种方法解决冲突时因为数据量不大
        e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        // 要查找的 ThreadLocal 对象找到了,直接设置需要设置的值,然后 return
        if (k == key) {
            e.value = value;
            return;
        }

        // 如果 k 为 null ,说明有 value 没有及时回收,此时通过 replaceStaleEntry 进行处理
        // replaceStaleEntry 具体内容等下分析
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // 如果 tab[i] == null ,则直接创建新的 entry 即可
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // 在创建之后调用 cleanSomeSlots 方法检查是否有 value 值没有及时回收
    // 如果 sz >= threshold ,则需要扩容,重新 hash 即, rehash();
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

通过源码可以看到，在 set 方法中，主要是通过 `replaceStaleEntry` 方法和 `cleanSomeSlots` 方法去做的检测和处理

接下来瞅瞅 `replaceStaleEntry` 都干了点儿啥

## replaceStaleEntry

```java
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    // 从当前 staleSlot 位置开始向前遍历
    int slotToExpunge = staleSlot;
    for (int i = prevIndex(staleSlot, len);
        (e = tab[i]) != null;
        i = prevIndex(i, len))
        if (e.get() == null)
            // 当 e.get() == null 时, slotToExpunge 记录下此时的 i 值
            // 即 slotToExpunge 记录的是 staleSlot 左手边第一个空的 Entry
            slotToExpunge = i;

    // 接下来从当前 staleSlot 位置向后遍历
    // 这两个遍历是为了清理在左边遇到的第一个空的 entry 到右边的第一个空的 entry 之间所有过期的对象
    // 但是如果在向后遍历过程中,找到了需要设置值的 key ,就开始清理,不会再继续向下遍历
    for (int i = nextIndex(staleSlot, len);
        (e = tab[i]) != null;
        i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();

        // 如果 k == key 说明在插入之前就已经有相同的 key 值存在,所以需要替换旧的值
        // 同时和前面过期的对象进行交换位置
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            // 如果 slotToExpunge == staleSlot 说明向前遍历时没有找到过期的
            if (slotToExpunge == staleSlot)
                slotToExpunge = i;
            // 进行清理过期数据
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

        // 如果在向后遍历时,没有找到 value 被回收的 Entry 对象
        // 且刚开始 staleSlot 的 key 为空,那么它本身就是需要设置 value 的 Entry 对象
        // 此时不涉及到清理
        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    // 如果 key 在数组中找不到,那就好说了,直接创建一个新的就可以了
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    // 如果 slotToExpunge != staleSlot 说明存在过期的对象,就需要进行清理
    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}
```

在 `replaceStaleEntry` 方法中,需要注意一下刚开始的两个 for 循环中内容（在这里再贴一下）：

```java
if (e.get() == null)
    // 当 e.get() == null 时, slotToExpunge 记录下此时的 i 值
    // 即 slotToExpunge 记录的是 staleSlot 左手边第一个空的 Entry
    slotToExpunge = i;

if (k == key) {
    e.value = value;

    tab[i] = tab[staleSlot];
    tab[staleSlot] = e;
                                        
    // 如果 slotToExpunge == staleSlot 说明向前遍历时没有找到过期的
    if (slotToExpunge == staleSlot)
        slotToExpunge = i;
        // 进行清理过期数据
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
        return;
}
```

这两个 for 循环中的 if 到底是在做什么？

看第一个 if ，当 `e.get() == null` 时，此时将 i 的值给 slotToExpunge

第二个 if ，当 `k ==key` 时，此时将 i 给了 staleSlot 来进行交换

为什么要对 `staleSlot` 进行交换呢？画图说明一下

如下图，假设此时表长为 10 ，其中下标为 3 和 5 的 key 已经被回收（ key 被回收掉的就是 null ），因为采用的开放地址法，所以 15 mod 10 应该是 5 ，但是因为位置被占，所以在 6 的位置，同样 25 mod 10 也应该是 5 ，但是因为位置被占，下个位置也被占，所以就在第 7 号的位置上了

按照上面的分析，此时 `slotToExpunge` 值为 3 ， `staleSlot` 值为 5 ， i 为 6

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/06-exchangeOne.jpg)

假设，假设这个时候如果不进行交换，而是直接回收的话，此时位置为 5 的数据就被回收掉，然后接下来要插入一个 key 为 15 的数据，此时 15 mod 10 算出来是 5 ，正好这个时候位置为 5 的被回收完毕，这个位置就被空出来了，那么此时就会这样：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/07-exchangeTwo.jpg)

同样的 key 值竟然出现了两次？！

这肯定是不希望看到的结果，所以一定要进行数据交换

在上面代码中有一行代码 `cleanSomeSlots(expungeStaleEntry(slotToExpunge), len); ` ，说明接下来的处理是交给了 `expungeStaleEntry` ，接下来去分析一下 `expungeStaleEntry`

## expungeStaleEntry

```java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
        (e = tab[i]) != null;
        i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        // 如果 k == null ,说明 value 就应该被回收掉
        if (k == null) {
            // 此时直接将 e.value 置为 null 
            // 这样就将 thread -> threadLocalMap -> value 这条引用链给打破
            // 方便了 GC
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            // 这个时候要重新 hash ,因为采用的是开放地址法,所以可以理解为就是将后面的元素向前移动
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;

                // Unlike Knuth 6.4 Algorithm R, we must scan until
                // null because multiple entries could have been stale.
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

因为是在 `replaceStaleEntry` 方法中调用的此方法，传进来的值是 `staleSlot` ，继续上图，经过 `replaceStaleEntry` 之后，它的数据结构是这样：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/08-exchangeThree.jpg)

此时传进来的 `staleSlot` 值为 6 ，因为此时的 key 为 null ，所以接下来会走 `e.value = null` ，这一步结束之后，就成了：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/09-exchangeFour.jpg)

接下来 i 为 7 ，此时的 key 不为 null ，那么就会重新 hash : `int h = k.threadLocalHashCode & (len - 1);` ，得到的 h 应该是 5 ，但是实际上 i 为 7 ，说明出现了 hash 冲突，就会继续向下走，最终的结果是这样：

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/10-exchangeFive.jpg)

可以看到，原来的 key 为 null ，值为 V5 的已经被回收掉了。我认为之所以回收掉之后，还要再次进行重新 hash ，就是为了防止 key 值重复插入情况的发生

假设 key 为 25 的并没有进行向前移动，也就是它还在位置 7 ，位置 6 是空的，再插入一个 key 为 25 ，经过 hash 应该在位置 5 ，但是有数据了，那就向下走，到了位置 6 ，诶，竟然是空的，赶紧插进去，这不就又造成了上面说到的问题，同样的一个 key 竟然出现了两次？！

而且经过 `expungeStaleEntry` 之后，将 key 为 null 的值，也设置为了 null ，这样就方便 GC

分析到这里应该就比较明确了，在 `expungeStaleEntry` 中，有些地方是帮助 GC 的，而通过源码能够发现， set 方法调用了该方法进行了 GC 处理， get 方法也有，不信你瞅瞅：

## get

```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    // 如果能够找到寻找的值,直接 return 即可
    if (e != null && e.get() == key)
        return e;
    else
        // 如果找不到,则调用 getEntryAfterMiss 方法去处理
        return getEntryAfterMiss(key, i, e);
}

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    // 一直探测寻找下一个元素,直到找到的元素是要找的
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            // 如果 k == null 说明有 value 没有及时回收
            // 调用 expungeStaleEntry 方法去处理,帮助 GC
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

get 和 set 方法都有进行帮助 GC ，所以正常情况下是不会有内存溢出的，但是如果创建了之后一直没有调用 get 或者 set 方法，还是有可能会内存溢出

所以最保险的方法就是，使用完之后就及时 remove 一下，加快垃圾回收，就完美的避免了垃圾回收

我 ThreadLocal 虽然没办法做到 100% 的解决内存泄漏问题，但是我能做到 80% 不也应该夸夸我嘛

![](http://www.justdojava.com/assets/images/2019/java/image-zll/2020/09/11-求夸.gif)