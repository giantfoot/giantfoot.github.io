#### LinkedHashSet

LinkedHashSet继承自HashSet，源码更少、更简单，唯一的区别是LinkedHashSet内部使用的是LinkHashMap。这样做的意义或者好处就是LinkedHashSet中的元素顺序是可以保证的，也就是说遍历序和插入序是一致的。

可是，LinkedHashSet中并没有覆盖add方法，只是加了几个构造函数和一个迭代器，其他全部和HashSet一毛一样，为什么它就能有序呢？？

玄机就藏在这个构造函数中，这几个构造函数其实都是调用了它父类（HashSet）的一个构造函数：
```
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```
这个构造函数跟其他构造函数唯一的区别就在于，它创建的是一个LinkedHashMap对象，所以元素之所以有序，完全是LinkedHashMap的功劳。该构造函数是默认访问权限的，所以在HashSet中是不能直接调用的，留给子类去调用或覆盖。
