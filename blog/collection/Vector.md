#### Vector 解析

```
public class Vector<E> extends AbstractList<E> implements List<E>,RandomAccess, Cloneable, java.io.Serializable

/**
    * 保存vector中元素的数组。vector的容量是数组的长度，数组的长度最小值为vector的元素个数。
    *
    * 任何在vector最后一个元素之后的数组元素是null。
    *
    * @serial
    */
   protected Object[] elementData;

   /**
    * vector中实际的元素个数。
    *
    * @serial
    */
   protected int elementCount;

   /**
    * vector需要自动扩容时增加的容量。
    *
    * 当vector的实际容量elementCount将要大于它的最大容量时，vector自动增加的容量。
    *
    * 如果capacityIncrement小于或等于0，vector的容量需要增长时将会成倍增长。
    * @serial
    */
   protected int capacityIncrement;

   /**
    * 序列版本号
    */
   private static final long serialVersionUID = -2767605614048989439L;
```

默认构造方法，构造一个指定容量为10、自增容量为0的空vector，扩容时成倍增长。

```
public Vector() {
       this(10);
   }
```

主要方法：
```
/**
    * 将vector中的所有元素拷贝到指定的数组anArray中
    *
    * @param  anArray 指定的数组anArray，用来存放vector中的所有元素
    *
    * @throws NullPointerException 如果指定的数组anArray为null
    * @throws IndexOutOfBoundsException 如果指定的数组anArray的容量小于vector的元素个数
    * @throws ArrayStoreException 如果vector不能被拷贝到anArray中
    * @see #toArray(Object[])
    */
   public synchronized void copyInto(Object[] anArray) {
       System.arraycopy(elementData, 0, anArray, 0, elementCount);
   }

   /**
  * 将底层数组的容量调整为当前vector实际元素的个数，来释放空间。
  */
 public synchronized void trimToSize() {
     modCount++;
     int oldCapacity = elementData.length;
     ////当实际大小小于底层数组的长度
     if (elementCount < oldCapacity) {
         //将底层数组的长度调整为实际大小
         elementData = Arrays.copyOf(elementData, elementCount);
     }
 }

 /**
  * 增加vector容量
  * 如果vector当前容量小于至少需要的容量，它的容量将增加。
  *
  * 新的容量将在旧的容量的基础上加上capacityIncrement，除非capacityIncrement小于等于0，在这种情况下，容量将会增加一倍。
  *
  * 增加后，如果新的容量还是小于至少需要的容量，那就将容量扩容至至少需要的容量。
  *
  * @param minCapacity 至少需要的容量
  */
 public synchronized void ensureCapacity(int minCapacity) {
     if (minCapacity > 0) {
         modCount++;
         ensureCapacityHelper(minCapacity);
     }
 }


 /**
  * ensureCapacity()方法的unsynchronized实现。
  *
  * ensureCapacity()是同步的，它可以调用本方法来扩容，而不用承受同步带来的消耗
  *
  * @see #ensureCapacity(int)
  */
 private void ensureCapacityHelper(int minCapacity) {
     // 如果至少需要的容量 > 数组缓冲区当前的长度，就进行扩容
     if (minCapacity - elementData.length > 0)
         grow(minCapacity);
 }

 /**
  * 分派给arrays的最大容量
  * 为什么要减去8呢？
  * 因为某些VM会在数组中保留一些头字，尝试分配这个最大存储容量，可能会导致array容量大于VM的limit，最终导致OutOfMemoryError。
  */
 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

 /**
 * 扩容，保证vector至少能存储minCapacity个元素。
 * 首次扩容时，newCapacity = oldCapacity + ((capacityIncrement > 0) ?capacityIncrement : oldCapacity);即如果capacityIncrement>0，就加capacityIncrement，如果不是就增加一倍。
 *
 * 如果第一次扩容后，容量还是小于minCapacity，就直接将容量增为minCapacity。
 *
 * @param minCapacity 至少需要的容量
 */
 private void grow(int minCapacity) {
     // 获取当前数组的容量
     int oldCapacity = elementData.length;
     // 扩容。新的容量=当前容量+当前容量/2.即将当前容量增加一倍
     int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                      capacityIncrement : oldCapacity);
     //如果扩容后的容量还是小于想要的最小容量
     if (newCapacity - minCapacity < 0)
         ///将扩容后的容量再次扩容为想要的最小容量
         newCapacity = minCapacity;
     ///如果扩容后的容量大于临界值，则进行大容量分配
     if (newCapacity - MAX_ARRAY_SIZE > 0)
         newCapacity = hugeCapacity(minCapacity);
     elementData = Arrays.copyOf(elementData, newCapacity);
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
