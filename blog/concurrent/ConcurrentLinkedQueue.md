#### ConcurrentLinkedQueue

要实现一个线程安全的队列有两种方式：阻塞和非阻塞。

阻塞队列无非就是锁的应用，而非阻塞则是CAS算法的应用。

下面我们就开始一个**非阻塞算法**的研究：CoucurrentLinkedQueue。

ConcurrentLinkedQueue是一个基于链接节点的无边界的线程安全队列，它采用FIFO原则对元素进行排序。
采用“wait-free”算法（即CAS算法）来实现的。

CoucurrentLinkedQueue规定了如下几个不变性：
```
在入队的最后一个元素的next为null
队列中所有未删除的节点的item都不能为null且都能从head节点遍历到
对于要删除的节点，不是直接将其设置为null，而是先将其item域设置为null（迭代器会跳过item为null的节点）
允许head和tail更新滞后。这是什么意思呢？意思就说是head、tail不总是指向第一个元素和最后一个元素（后面阐述）。
```

```
private static class Node<E> {
     /** 节点元素域 */
     volatile E item;
     volatile Node<E> next;

     //初始化,获得item 和 next 的偏移量,为后期的CAS做准备

     Node(E item) {
         UNSAFE.putObject(this, itemOffset, item);
     }

     boolean casItem(E cmp, E val) {
         return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
     }

     void lazySetNext(Node<E> val) {
         UNSAFE.putOrderedObject(this, nextOffset, val);
     }

     boolean casNext(Node<E> cmp, Node<E> val) {
         return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
     }

     // Unsafe mechanics

     private static final sun.misc.Unsafe UNSAFE;
     /** 偏移量 */
     private static final long itemOffset;
     /** 下一个元素的偏移量 */

    private static final long nextOffset;

     static {
         try {
             UNSAFE = sun.misc.Unsafe.getUnsafe();
             Class<?> k = Node.class;
             itemOffset = UNSAFE.objectFieldOffset
                     (k.getDeclaredField("item"));
             nextOffset = UNSAFE.objectFieldOffset
                     (k.getDeclaredField("next"));
         } catch (Exception e) {
             throw new Error(e);
         }
     }
 }
```

入列，我们认为是一个非常简单的过程：

tail节点的next执行新节点，然后更新tail为新节点即可。

从单线程角度我们这么理解应该是没有问题的，但是多线程呢？

如果一个线程正在进行插入动作，那么它必须先获取尾节点，然后设置尾节点的下一个节点为当前节点，但是如果已经有一个线程刚刚好完成了插入，那么尾节点是不是发生了变化？

对于这种情况ConcurrentLinkedQueue怎么处理呢？

我们先看源码： offer(E e)：将指定元素插入都队列尾部：
```
public boolean offer(E e) {
        //检查节点是否为null
        checkNotNull(e);
        // 创建新节点
        final Node<E> newNode = new Node<E>(e);

        //死循环 直到成功为止
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            // q == null 表示 p已经是最后一个节点了，尝试加入到队列尾
            // 如果插入失败，则表示其他线程已经修改了p的指向
            if (q == null) {                                // --- 1
                // casNext：t节点的next指向当前节点
                // casTail：设置tail 尾节点
                if (p.casNext(null, newNode)) {             // --- 2
                    // node 加入节点后会导致tail距离最后一个节点相差大于一个，需要更新tail
                    if (p != t)                             // --- 3
                        casTail(t, newNode);                    // --- 4
                    return true;
                }
            }
            // p == q 等于自身
            else if (p == q)                                // --- 5
                // p == q 代表着该节点已经被删除了
                // 由于多线程的原因，我们offer()的时候也会poll()，如果offer()的时候正好该节点已经poll()了
                // 那么在poll()方法中的updateHead()方法会将head指向当前的q，而把p.next指向自己，即：p.next == p
                // 这样就会导致tail节点滞后head（tail位于head的前面），则需要重新设置p
                p = (t != (t = tail)) ? t : head;           // --- 6
            // tail并没有指向尾节点
            else
                // tail已经不是最后一个节点，将p指向最后一个节点
                p = (p != t && t != (t = tail)) ? t : q;    // --- 7
        }
    }
```
 初始化 ConcurrentLinkedQueue初始化时head、tail存储的元素都为null，且head等于tail：
 ```
 public ConcurrentLinkedQueue() {
    head = tail = new Node<E>(null);
}
```
Node是个单向链表节点，next用于指向下一个Node，item用于存储数据。Node中操作节点数据的API，都是通过Unsafe机制的CAS函数实现的；例如casNext()是通过CAS函数“比较并设置节点的下一个节点”。
