#### Hashtable

> Hashtable和HashMap，从存储结构和实现来讲基本上都是相同的。它和HashMap的最大的不同是它是线程安全的，另外它不允许key和value为null。Hashtable是个过时的集合类，不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

一个Hashtable的实例有两个影响它行为的参数：初始化容量initial capacity 和负载因子load factor。容量是哈希表中桶的数量，初始化容量是哈希表被创建时的容量。

哈希表是开放的，就哈希碰撞来说，一个桶存储数个必须被顺序查找的node。负载因子是哈希表在自动扩容之前可以多满的一个度量（哈希表几乎不会在满时才会扩容，加载因子越大，在扩容前哈希表可以存放的节点就越多）。初始化容量initial capacity 和负载因子load factor仅仅对实现有细微的暗示。何时扩容，是否扩容取决于具体的实现。

一般来说，默认的加载因子0.75在哈希表和时间和空间花销上是一个很好的平衡。更高的加载因子减少了空间开销但增加了查找操作的时间开销（影响了大多的哈希表操作，包括get和put操作）。


iterator方法返回的迭代器是fail-fast的。如果在迭代器被创建后hashtable被结构型地修改了，除了迭代器自己的remove方法，迭代器会抛出一个ConcurrentModificationException异常。因此，面对在并发的修改，迭代器干脆利落的失败，而不是冒险的继续。哈希表的key和元素集合返回的Enumerations不是fail-fast的。   

> 自从Java2开始，Hashtable继承Map接口，成为了容器中的一员。

**Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。**

```
/**
     * 使用默认初始化容量（11）和默认负载因子（0.75）来构造一个空的hashtable.
     *
     * 这里可以看到，Hashtable默认初始化容量为16，而HashMap的默认初始化容量为11。
     */
    public Hashtable() {
        this(11, 0.75f);
    }
```

> 构造方法直接初始化数组，而HashMap是在第一次放入数据的时候才初始化

```
/**
  * 使用指定参数初始化容量和指定参数负载因子来构造一个空的hashtable.
  *
  * @param      initialCapacity   指定参数初始化容量
  * @param      loadFactor        指定参数负载因子
  * @exception  IllegalArgumentException  如果initialCapacity小于0或者负载因子为非正数。
  */
 public Hashtable(int initialCapacity, float loadFactor) {
     //如果指定参数初始化容量小于0，抛出异常
     if (initialCapacity < 0)
         throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
     //如果指定参数负载因子为非正数，抛出异常
     if (loadFactor <= 0 || Float.isNaN(loadFactor))
         throw new IllegalArgumentException("Illegal Load: "+loadFactor);
     //初始化hashtable的loadFactor、table、threshold属性
     if (initialCapacity==0)
         initialCapacity = 1;
     this.loadFactor = loadFactor;
     table = new Entry<?,?>[initialCapacity];
     //如果initialCapacity * loadFactor超出了MAX_ARRAY_SIZE，就使用MAX_ARRAY_SIZE 作为threshold
     threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
 }
```

Hashtable和HashMap确认key在数组中的索引的方法不同。

- Hashtable 通过index = (hash & 0x7FFFFFFF) % tab.length;来确认  
- HashMap 通过i = (n - 1) & hash;来确认

```
0x7FFFFFFF 是多少？
每个bai十六进制数4bit，因du此8位16进制是4个字节zhi，刚好是一个int整型
F的二进dao制码为 1111
7的二进制码为 0111
这样一来，整个整数 0x7FFFFFFF 的二进制表示就是除了首位是 0，其余都是1
就是说，这是最大的整型数 int（因为第一位是符号位，0 表示他是正数）
```

**rehash的总体思路为：**
```

1.  新建变量_新的容量_，值为旧的容量的2倍+1
2.  如果新的容量大于容量的最大值MAX\_ARRAY\_SIZE  
    1.  如果旧容量为MAX\_ARRAY\_SIZE，容量不变，中断方法的执行
    2.  如果旧容量不为MAX\_ARRAY\_SIZE，新容量变为MAX\_ARRAY\_SIZE
3.  创建新的数组，容量为新容量
4.  将旧的数组中的键值对转移到新数组中

这里可以看到，一般情况下，HashMap扩容后容量变为原来的两倍，而Hashtable扩容后容量变为原来的两倍+1。
```

**被添加的键值对中的key和value都不能为null**， put方法直接获取key的hashcode

synchronized修饰的put方法
```
/**
 * 添加指定键值对到hashtable中
 * 被添加的键值对中的key和value都不能为null
 *
 * value可以通过get方法被取出。
 *
 * @param      key     the hashtable key
 * @param      value   the value
 * @return     如果hashtable中已经存在key，则返回原来的value
 * @exception  NullPointerException  如果key或者value为null
 * @see     Object#equals(Object)
 * @see     #get(Object)
 */
public synchronized V put(K key, V value) {
    // 确认value不为null
    if (value == null) {
        throw new NullPointerException();
    }

    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    //找到key在table中的索引
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    //获取key所在索引的entry
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    //遍历entry，判断key是否已经存在
    for(; entry != null ; entry = entry.next) {
        //如果key已经存在
        if ((entry.hash == hash) && entry.key.equals(key)) {
            //保存旧的value
            V old = entry.value;
            //替换value
            entry.value = value;
            //返回旧的value
            return old;
        }
    }
    //如果key在hashtable不是已经存在，就直接将键值对添加到table中，返回null
    addEntry(hash, key, value, index);
    return null;
}
```

**不同点**

![](../pic/collection/hashtable.png)
