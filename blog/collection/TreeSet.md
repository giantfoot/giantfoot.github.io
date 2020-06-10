#### TreeSet

TreeSet是依赖于TreeMap的NavigableSet接口的实现，实际上是个TreeMap的实例。

```
private transient NavigableMap<E,Object> m;

   TreeSet(NavigableMap<E,Object> m) {
       this.m = m;
   }

   public TreeSet() {
       this(new TreeMap<E,Object>());
   }

   /**
   * The backing map.
   */
  private transient NavigableMap<E,Object> m;

  /**
   *  前面讲了，TreeSet是依赖于TreeMap的，底层就是一个TreeMap实例。TreeMap是保存键值对的，但我们保存TreeSet的时候肯定只是想保存key，那么调用hashMap(key,value)时value应该传什么值呢？PRESENT就是要传value。
   */
  private static final Object PRESENT = new Object();
```
