#### HashSet

HashSet是依赖于HashMap的Set接口的实现，实际上是个HashMap的实例。HashSet的特点是不保证set的迭代顺序，特别是它不保证该顺序恒久不变，允许使用null 元素。

```
private transient HashMap<E,Object> map;

   /*
    * 构造方法之一，其他几个构造方法也是类似的。
    */
   public HashSet() {
       map = new HashMap<>();
   }

   /**
    * 说明HashSet是依赖于HashMap的，底层就是一个HashMap实例。
    */
   private transient HashMap<E,Object> map;
   /**
    * 前面讲了，HashSet是依赖于HashMap的，底层就是一个HashMap实例。HashMap是保存键值对的，但我们保存hashSet的时候肯定只是想保存key，那么调用hashMap(key,value)时value应该传什么值呢？PRESENT就是value。
    */
   private static final Object PRESENT = new Object();


   public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
```

> 对此set进行迭代所需的时间与 HashSet 实例的大小（元素的数量）和底层 HashMap 实例（桶的数量）的“容量”的和成比例。因此，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。
