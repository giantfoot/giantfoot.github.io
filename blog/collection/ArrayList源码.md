#### ArrayList 解析

ArrayList实现了RandomAccess接口，先看下这个接口，**打开源码后，发现接口里面什么也没有，这是个空的接口，并且是1.4才引入的**，可以理解为，让调用方知道这个集合支持快速随机访问，以便加快调用方的运行速度。

```
* @since 1.4
 */
public interface RandomAccess {
}
```
RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持快速随机访问的。也就是说，实现了这个接口的集合是支持 快速随机访问 策略的。**官网还特意说明了，如果是实现了这个接口的 List，那么使用for循环的方式获取数据会优于用迭代器获取数据。**

当要实现某些算法时，会判断当前类是否实现了RandomAccess接口，会选择不同的算法。

比如在Collections二分查找时，
```
public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
```
在进行二分查找时，首先判断list是否实现了RandomAccess，然后选择执行最优算法。
如果集合类是RandomAccess的实现，则尽量用for(int i = 0; i < size; i++) 即for循环来遍历，而不是用Iterator
迭代器来进行迭代。

> List实现所使用的标记接口，用来表明实现了这些接口的list支持快速（通常是常数时间）随机访问。
 这个接口的主要目的是允许一般的算法更改它们的行为，以便在随机或者顺序存取列表时能提供更好的性能.


- List接口是大小可变数组的实现，实现了所有可选列表操作，**并允许包括null在内的所有元素**。

- 此类的iterator和listIterator方法返回的迭代器是快速失败的：在创建迭代器之后，**除非通过迭代器自身的remove或add方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出ConcurrentModificationException**。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

> 注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出ConcurrentModificationException。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测bug。

- size、isEmpty、get、set、iterator和listIterator方法都以固定时间运行，时间复杂度为O(1),添加n个元素需要O(n)时间。

```
public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,java.io.Serializable

/**
   * 初始化默认容量。
   */
  private static final int DEFAULT_CAPACITY = 10;

  /**
   * 指定该ArrayList容量为0时，返回该空数组。
   */
  private static final Object[] EMPTY_ELEMENTDATA = {};

  /**
   * 当调用无参构造方法，返回的是该数组。刚创建一个ArrayList 时，其内数据量为0。
   * 它与EMPTY_ELEMENTDATA的区别就是：该数组是默认返回的，而后者是在用户指定容量为0时返回。
   */
  private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

  /**
   * 保存添加到ArrayList中的元素。
   * ArrayList的容量就是该数组的长度。
   * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，数组将扩容值DEFAULT_CAPACITY。
   * 被标记为transient，在对象被序列化的时候不会被序列化。
   */
  transient Object[] elementData; // non-private to simplify nested class access

  /**
   * ArrayList的实际大小（数组包含的元素个数）。
   * @serial
   */
  private int size;
```

elementData被标记为transient，那么它的序列化和反序列化是如何实现的呢？

> ArrayList自定义了它的序列化和反序列化方式。详情请查看writeObject(java.io.ObjectOutputStream s)和readObject(java.io.ObjectOutputStream s)方法。

```
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

/**
 * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

- get方法的时间复杂度是O(1)。

```
/**
     * 添加元素到list末尾.
     *
     * @param e 被添加的元素
     * @return true
     */
    public boolean add(E e) {
        //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
   * 增加ArrayList容量。
   *
   * @param   minCapacity   想要的最小容量
   */
   public void ensureCapacity(int minCapacity) {
       // 如果elementData等于DEFAULTCAPACITY_EMPTY_ELEMENTDATA，最小扩容量为DEFAULT_CAPACITY，否则为0
       int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)? 0: DEFAULT_CAPACITY;
       //如果想要的最小容量大于最小扩容量，则使用想要的最小容量。
       if (minCapacity > minExpand) {
           ensureExplicitCapacity(minCapacity);
       }
   }
   /**
   * 数组容量检查，不够时则进行扩容，只供类内部使用。
   *
   * @param minCapacity    想要的最小容量
   */
   private void ensureCapacityInternal(int minCapacity) {
       // 若elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA，则取minCapacity为DEFAULT_CAPACITY和参数minCapacity之间的最大值
       if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
           minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
       }

       ensureExplicitCapacity(minCapacity);
   }
   /**
   * 数组容量检查，不够时则进行扩容，只供类内部使用
   *
   * @param minCapacity 想要的最小容量
   */
   private void ensureExplicitCapacity(int minCapacity) {
       modCount++;

       // 确保指定的最小容量 > 数组缓冲区当前的长度  
       if (minCapacity - elementData.length > 0)
           //扩容
           grow(minCapacity);
   }

   /**
    * 分派给arrays的最大容量
    * 为什么要减去8呢？
    * 因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。
    */
   private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

   /**
   * 扩容，保证ArrayList至少能存储minCapacity个元素
   * 第一次扩容，逻辑为newCapacity = oldCapacity + (oldCapacity >> 1);即在原有的容量基础上增加一半。第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。
   *
   * @param minCapacity 想要的最小容量
   */
   private void grow(int minCapacity) {
       // 获取当前数组的容量
       int oldCapacity = elementData.length;
       // 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一半。
       int newCapacity = oldCapacity + (oldCapacity >> 1);
       //如果扩容后的容量还是小于想要的最小容量
       if (newCapacity - minCapacity < 0)
           //将扩容后的容量再次扩容为想要的最小容量
           newCapacity = minCapacity;
       //如果扩容后的容量大于临界值，则进行大容量分配
       if (newCapacity - MAX_ARRAY_SIZE > 0)
           newCapacity = hugeCapacity(minCapacity);
       // minCapacity is usually close to size, so this is a win:
       elementData = Arrays.copyOf(elementData,newCapacity);
   }
   /**
   * 进行大容量分配
   */
   private static int hugeCapacity(int minCapacity) {
       //如果minCapacity<0，抛出异常
       if (minCapacity < 0) // overflow
           throw new OutOfMemoryError();
       //如果想要的容量大于MAX_ARRAY_SIZE，则分配Integer.MAX_VALUE，否则分配MAX_ARRAY_SIZE
       return (minCapacity > MAX_ARRAY_SIZE) ?
           Integer.MAX_VALUE :
           MAX_ARRAY_SIZE;
   }
```


- add步骤，时间复杂度，即O(1)。

1. 最大容量为Integer.MAX_VALUE - 8;
> 为什么要减去8呢？因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。

2. 如果添加的是第一个数据，那扩容的大小为默认值10，如果size+1<现有容量，则不进行扩容，否则扩容。
3. 新的容量=当前容量+当前容量/2.即将当前容量增加一半。
4. 如果扩容后的容量大于临界值，则进行大容量分配，及扩充为Integer.MAX_VALUE。

- add( int index, E element),线性的时间复杂度，即O(n)。

```
/**
    * 在制定位置插入元素。当前位置的元素和index之后的元素向后移一位
    *
    * @param index 即将插入元素的位置
    * @param element 即将插入的元素
    * @throws IndexOutOfBoundsException 如果索引超出size
    */
   public void add(int index, E element) {
       //越界检查
       rangeCheckForAdd(index);
       //确认list容量，如果不够，容量加1。注意：只加1，保证资源不被浪费
       ensureCapacityInternal(size + 1);  // Increments modCount!!
       // 对数组进行复制处理，目的就是空出index的位置插入element，并将index后的元素位移一个位置
       System.arraycopy(elementData, index, elementData, index + 1,size - index);
       //将指定的index位置赋值为element
       elementData[index] = element;
       //实际容量+1
       size++;
   }
```

- remove( int index)

> 如果有需要左移的元素，就移动（移动后，该删除的元素就已经被覆盖了,相当于被删除了）

```
/**
    * 删除list中位置为指定索引index的元素
    * 索引之后的元素向左移一位
    *
    * @param index 被删除元素的索引
    * @return 被删除的元素
    * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
    */
   public E remove(int index) {
       //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
       rangeCheck(index);
       //结构性修改次数+1
       modCount++;
       //记录索引为inde处的元素
       E oldValue = elementData(index);

       // 删除指定元素后，需要左移的元素个数
       int numMoved = size - index - 1;
       //如果有需要左移的元素，就移动（移动后，该删除的元素就已经被覆盖了）
       if (numMoved > 0)
           System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
       // size减一，然后将索引为size-1处的元素置为null。为了让GC起作用，必须显式的为最后一个位置赋null值
       elementData[--size] = null; // clear to let GC do its work

       //返回被删除的元素
       return oldValue;
   }

   /**
    * 越界检查。
    * 检查给出的索引index是否越界。
    * 如果越界，抛出运行时异常。
    * 这个方法并不检查index是否合法。比如是否为负数。
    * 如果给出的索引index>=size，抛出一个越界异常
    */
   private void rangeCheck(int index) {
       if (index >= size)
           throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
   }
```

1. 检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常

2. 将索引大于index的元素左移一位（左移后，该删除的元素就被覆盖了，相当于被删除了）。
3. 将索引为size-1处的元素置为null（为了让GC起作用）。

> 注意：为了让GC起作用，必须显式的为最后一个位置赋null值。上面代码中如果不手动赋null值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。


- set( int index, E element)
```
/**
 * 替换指定索引的元素
 *
 * @param 被替换元素的索引
 * @param element 即将替换到指定索引的元素
 * @return 返回被替换的元素
 * @throws IndexOutOfBoundsException 如果参数指定索引index>=size，抛出一个越界异常
 */
public E set(int index, E element) {
    //检查索引是否越界。如果参数指定索引index>=size，抛出一个越界异常
    rangeCheck(index);

    //记录被替换的元素
    E oldValue = elementData(index);
    //替换元素
    elementData[index] = element;
    //返回被替换的元素
    return oldValue;
}
```

- addAll
调用AbstractList的addAll， 循环遍历添加，实质还是循环调用ArrayList的add方法。
```
public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);
            modified = true;
        }
        return modified;
    }
```
**迭代器**
```
public class ArrayList<E>
{
    /**
     * 保存添加到ArrayList中的元素。
     * ArrayList的容量就是该数组的长度。
     * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，数组将扩容值DEFAULT_CAPACITY。
     * 被标记为transient，在对象被序列化的时候不会被序列化。
     */
    transient Object[] elementData;

    // ArrayList的实际大小（数组包含的元素个数）。
    private int size;

    /**
     * 返回一个用来遍历ArrayList元素的Iterator迭代器
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * AbstractList.Itr的最优化的版本
     */
    private class Itr implements Iterator<E> {
        int cursor;       // 下一个要返回的元素的索引
        int lastRet = -1; // 最近的被返回的元素的索引; 如果没有返回-1。
        int expectedModCount = modCount;
        /**
         * 判断是否有下一个元素
        */
        public boolean hasNext() {
            //如果下一个要返回的元素的索引不等于ArrayList的实际大小，则返回false
            return cursor != size;
        }

        /**
         * 返回下一个元素
         */
        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
        /**
         * 删除最近的一个被返回的元素
         */
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

- iterator(),返回的是一个内部类，利用指针不停的往后遍历。**只能单向移动。**
```
public class ArrayList<E>
{
    /**
     * 保存添加到ArrayList中的元素。
     * ArrayList的容量就是该数组的长度。
     * 该值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时，当第一次添加元素进入ArrayList中时，数组将扩容值DEFAULT_CAPACITY。
     * 被标记为transient，在对象被序列化的时候不会被序列化。
     */
    transient Object[] elementData;

    // ArrayList的实际大小（数组包含的元素个数）。
    private int size;

    /**
     * 返回一个一开始就指向列表索引为index的元素处的ListIterator
     *
     * @throws IndexOutOfBoundsException
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 返回一个一开始就指向列表索引为0的元素处的ListIterator
     *
     * @see #listIterator(int)
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * AbstractList.ListItr的最优化版本
     */
    private class ListItr extends Itr implements ListIterator<E> {
        //用来从list中返回一个指向list索引为index的元素处的迭代器
        ListItr(int index) {
            super();
            cursor = index;
        }

        //获取list中的上个元素
        public boolean hasPrevious() {
            return cursor != 0;
        }

        //获取list中的下个元素的索引
        public int nextIndex() {
            return cursor;
        }

        //获取list中的上个元素的索引
        public int previousIndex() {
            return cursor - 1;
        }

        //获取list中的上个元素
        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        //从列表中将next()或previous()返回的最后一个元素返回的最后一个元素更改为指定元素e
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        //将指定的元素插入列表，插入位置为迭代器当前位置之前。
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
}
```

- listIterator()返回的也是内部类,它最大的优点是可以**双向移动**，它还可以产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引，并且可以使用set()方法替换它访问过的最后一个元素。
```
private class ListItr extends Itr implements ListIterator<E>
```
