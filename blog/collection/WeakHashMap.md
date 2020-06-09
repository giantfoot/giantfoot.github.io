#### WeakHashMap

WeakHashMap和HashMap相似，也是哈希表的实现，以键值对的形式存储数据，key和value都可以为null。不同的是WeakHashMap的键为“弱键”。

当一个键不再正常使用，键对应的键值对将自动从WeakHashMap中删除。更严谨的说法是，键对应的键值对的存在并不阻止key被垃圾回收期回收，这就使该键称为可被终止的，最终被终止，被回收。当某个键被回收，它对应的键值对也就被从map中有效地删除了。

像大多的集合类一样，WeakHashMap是非同步的。可以使用Collections.synchronizedMap来构造同步的WeakHashMap。


> 与HashMap相比，缺少了TREEIFY_THRESHOLD 、UNTREEIFY_THRESHOLD、MIN_TREEIFY_CAPACITY三个静态全局变量，而这三个静态全局变量是针对红黑树与链表的转换的。从这里可以验证WeakHashMap的数据结构为哈希表+链表。


**WeakHashMap.Entry**

```
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }
```

**ReferenceQueue，他的作用是GC会清理掉对象之后，引用对象会被放到ReferenceQueue中。**

**弱引用：如果一个对象具有弱引用，在垃圾回收时候，一旦发现弱引用对象，无论当前内存空间是否充足，都会将弱引用回收。**

WeakHashMap是基于弱引用的，也就是说只要垃圾回收机制一开启，就直接开始了扫荡，看见了就清除。

**为什么需要WeakHashMap？**

WeakHashMap正是由于使用的是弱引用，**因此它的对象可能被随时回收**。更直观的说，当使用 WeakHashMap 时，即使没有删除任何元素，它的尺寸、get方法也可能不一样。比如：

（1）调用两次size()方法返回不同的值；第一次为10，第二次就为8了。

（2）两次调用isEmpty()方法，第一次返回false，第二次返回true；

（3）两次调用containsKey()方法，第一次返回true，第二次返回false；

（4）两次调用get()方法，第一次返回一个value，第二次返回null；

是不是觉得有点恶心，这种飘忽不定的东西好像没什么用，试想一下，你准备使用WeakHashMap保存一些数据，写着写着都没了，那还保存个啥呀。

不过有一种场景，最喜欢这种飘忽不定、一言不合就删除的东西。那就是缓存。在缓存场景下，由于内存是有限的，不能缓存所有对象，因此就需要一定的删除机制，淘汰掉一些对象。

现在我们已经知道了WeakHashMap是基于弱引用，其对象可能随时被回收，适用于 **缓存** 的场景。下面我们就来看看，WeakHashMap是如何实现这些功能。


**WeakHashMap中的Entry被GC后，WeakHashMap是如何将其移除的？**

意思是某一个Entry突然被垃圾回收了，这之后WeakHashMap肯定就不能保留这个Entry了，那他是如何将其移除的呢？

WeakHashMap内部有一个expungeStaleEntries函数，在这个函数内部实现移除其内部不用的entry从而达到的自动释放内存的目的。因此我们每次访问WeakHashMap的时候，都会调用这个expungeStaleEntries函数清理一遍。这也就是为什么前两次调用WeakHashMap的size()方法有可能不一样的原因。


  (01) 新建WeakHashMap，将“键值对”添加到WeakHashMap中。
          实际上，WeakHashMap是通过数组table保存Entry(键值对)；每一个Entry实际上是一个单向链表，即Entry是键值对链表。

  (02) **当某“弱键”不再被其它对象引用**，并被GC回收时。在GC回收该“弱键”时，这个“弱键”也同时会被添加到ReferenceQueue(queue)队列中。

  (03) **当下一次我们需要操作WeakHashMap时，会先同步table和queue**。table中保存了全部的键值对，而queue中保存被GC回收的键值对；**同步它们，就是删除table中被GC回收的键值对。**

  这就是“弱键”如何被自动从WeakHashMap中删除的步骤了。

**首先GC每次清理掉一个对象之后，引用对象会被放到ReferenceQueue中。然后遍历这个queue进行删除即可。**

当然。WeakHashMap的增删改查操作都会直接或者间接的调用expungeStaleEntries()方法，达到及时清除过期entry的目的。

我们可以看看是如何实现的：
```
private void expungeStaleEntries() {
        for (Object x; (x = queue.poll()) != null; ) {
            synchronized (queue) {
                @SuppressWarnings("unchecked")
                    Entry<K,V> e = (Entry<K,V>) x;
                int i = indexFor(e.hash, table.length);

                Entry<K,V> prev = table[i];
                Entry<K,V> p = prev;
                while (p != null) {
                    Entry<K,V> next = p.next;
                    if (p == e) {
                        if (prev == e)
                            table[i] = next;
                        else
                            prev.next = next;
                        // Must not null out e.next;
                        // stale entries may be in use by a HashIterator
                        e.value = null; // Help GC
                        size--;
                        break;
                    }
                    prev = p;
                    p = next;
                }
            }
        }
    }
```
**不要使用基础类型作为WeakHashMap的key**

objectMap.put方法执行的时候i会被封装为Integer类型的，Integer保留了-128到127的缓存。但是对于int来说范围大很多，因此那些Key <= 127的Entry将不会进行自动回收，但是那些大于127的将会被回收，因此最后的尺寸总是会稳定在128左右。


> WeakHashMap的key是弱引用，ThreadLocal的key也是用的弱引用，但是WeakHashMap在被GC回收时value也会被回收了，而ThreadLocal则不会，ThreadLocal必须显示的调用一下remove方法才能将value的值给清空。
在两个的源码中可以看到，ThreadLocal里面并没有一个接受对象被GC回收后通知的ReferenceQueue，所以就算key被回收了，value也是存在的，并不会和WeakHashMap一样，在key被清空后，可以使用ReferenceQueue这个队列接受被GC的通知，然后把value也自己给清空。

在Thread类中保有ThreadLocal.ThreadLocalMap的引用，即在一个Java线程栈中指向了堆内存中的一个ThreadLocal.ThreadLocalMap的对象，此对象中保存了若干个Entry，每个Entry的key(ThreadLocal实例)是弱引用，value是强引用（这点类似于WeakHashMap）。

用到弱引用的只是key，每个key都弱引用指向threadLocal，当把threadLocal实例置为null以后，没有任何强引用指向threadLocal实例，所以threadLocal将会被gc回收，但是value却不能被回收，因为其还存在于ThreadLocal.ThreadLocalMap的对象的Entry之中。

只有当前Thread结束之后，所有与当前线程有关的资源才会被GC回收。所以，如果在线程池中使用ThreadLocal，由于线程会复用，而又没有显示的调用remove的话的确是会有可能发生内存泄露的问题
