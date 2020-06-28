#### CopyOnWriteArraySet

它是线程安全的无序的集合，可以将它理解成线程安全的HashSet。

有意思的是，CopyOnWriteArraySet和HashSet虽然都继承于共同的父类AbstractSet；但是，HashSet是通过“散列表(HashMap)”实现的，而CopyOnWriteArraySet则是通过“动态数组(CopyOnWriteArrayList)”实现的，并不是散列表。

1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。

2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

CopyOnWriteArraySet包含CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。

而CopyOnWriteArrayList本质是个动态数组队列，
所以CopyOnWriteArraySet相当于通过通过动态数组实现的“集合”！ CopyOnWriteArrayList中允许有重复的元素；但是，CopyOnWriteArraySet是一个集合，所以它不能有重复集合。

因此，CopyOnWriteArrayList额外提供了addIfAbsent()和addAllAbsent()这两个添加元素的API，通过这些API来添加元素时，**只有当元素不存在时才执行添加操作！**

```
// 创建一个空 set。
CopyOnWriteArraySet()
// 创建一个包含指定 collection 所有元素的 set。
CopyOnWriteArraySet(Collection<? extends E> c)

// 如果指定元素并不存在于此 set 中，则添加它。
boolean add(E e)
// 如果此 set 中没有指定 collection 中的所有元素，则将它们都添加到此 set 中。
boolean addAll(Collection<? extends E> c)
// 移除此 set 中的所有元素。
void clear()
// 如果此 set 包含指定元素，则返回 true。
boolean contains(Object o)
// 如果此 set 包含指定 collection 的所有元素，则返回 true。
boolean containsAll(Collection<?> c)
// 比较指定对象与此 set 的相等性。
boolean equals(Object o)
// 如果此 set 不包含任何元素，则返回 true。
boolean isEmpty()
// 返回按照元素添加顺序在此 set 中包含的元素上进行迭代的迭代器。
Iterator<E> iterator()
// 如果指定元素存在于此 set 中，则将其移除。
boolean remove(Object o)
// 移除此 set 中包含在指定 collection 中的所有元素。
boolean removeAll(Collection<?> c)
// 仅保留此 set 中那些包含在指定 collection 中的元素。
boolean retainAll(Collection<?> c)
// 返回此 set 中的元素数目。
int size()
// 返回一个包含此 set 所有元素的数组。
Object[] toArray()
// 返回一个包含此 set 所有元素的数组；返回数组的运行时类型是指定数组的类型。
<T> T[] toArray(T[] a)
```

CopyOnWriteArraySet是通过CopyOnWriteArrayList实现的，它的API基本上都是通过调用CopyOnWriteArrayList的API来实现的。
