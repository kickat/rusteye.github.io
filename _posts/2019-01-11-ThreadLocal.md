---
layout: post
title:  "threadlocal"
date:   2019-01-07 00:00:00
categories: java
permalink: /archivers/java/threadlocal
---

<!--more-->

### ThreadLocal

ThreadLocal保证了各个线程的数据互不干扰。

```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

每个线程中都有一个ThreadLocalMap数据结构，当执行set方法时，其值是保存在当前线程的threadLocals变量中，当执行get方法中，是从当前线程的threadLocals变量获取。

ThreadLoalMap是一个类似HashMap的数据结构，但是在ThreadLocal中，并没实现Map接口。

在ThreadLoalMap中，初始化一个大小16的Entry数组，Entry对象用来保存每一个key-value键值对，key是ThreadLocal对象，通过ThreadLocal对象的set方法，把ThreadLocal对象自己当做key，放进了ThreadLoalMap中。

ThreadLoalMap的Entry是继承WeakReference，和HashMap很大的区别是，Entry中没有next字段，所以就不存在链表的情况了。

### hash冲突

```
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

每个ThreadLocal对象都有一个hash值threadLocalHashCode，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小0x61c88647。

在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下：
1、如果当前位置是空的，那么正好，就初始化一个Entry对象放在位置i上；
2、不巧，位置i已经有Entry对象了，如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value；
3、很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置；
这样的话，在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该位置Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置
可以发现，set和get如果冲突严重的话，效率很低，因为ThreadLoalMap是Thread的一个属性，所以即使在自己的代码中控制了设置的元素个数，但还是不能控制其它代码的行为。

### 内存泄露
在ThreadLocalMap的实现中，key被保存到了WeakReference对象中。ThreadLocal在没有外部强引用时，发生GC时会被回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

#### 解决
使用完ThreadLocal之后，调用remove方法。

