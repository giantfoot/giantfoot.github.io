#### PriorityBlockingQueue

我们知道线程Thread可以调用setPriority(int newPriority)来设置优先级的，线程优先级高的线程先执行，优先级低的后执行。

而前面介绍的ArrayBlockingQueue、LinkedBlockingQueue都是采用FIFO原则来确定线程执行的先后顺序，那么有没有一个队列可以支持优先级呢？

PriorityBlockingQueue是一个支持优先级的无界阻塞队列。

默认情况下元素采用自然顺序升序排序，当然我们也可以通过构造函数来指定Comparator来对元素进行排序。需要注意的是PriorityBlockingQueue不能保证同优先级元素的顺序。

> 二叉堆

所谓完全二叉树，即高度为n的二叉树，其前n-1层必须被填满，第n层也要从左到右顺序填满。

二叉堆是堆的一种，使用完全二叉树来实现。

在二叉堆中，所有非终端结点的值均不大于（或不小于）其左右孩子的值。

若非终端结点的值均不大于其左右孩子结点的值，这样的二叉堆叫做最小堆，小根堆根结点的值是该堆中所有结点的最小值；同样的，当所有非终端结点的值都不小于其左右孩子的值时，这样的对叫做最大堆，大根堆根结点的值为改堆所有结点的最大值。

堆一般使用数组来构建，假设为数组a[]，根结点通常存储在a[1]，这样对于下标为k的结点a[k]来说，其左孩子的下标为2*k，右孩子的下标为2*k+1，其父节点为（k - 1） / 2 处。

> 添加元素

首先将要添加的元素N插添加到堆的末尾位置（在二叉堆中我们称之为空穴）。如果元素N放入空穴中而不破坏堆的序（其值大于跟父节点值（最大堆是小于父节点）），那么插入完成。否则，我们则将该元素N的节点与其父节点进行交换，然后与其新父节点进行比较直到它的父节点不在比它小（最大堆是大）或者到达根节点。

> 删除元素

删除元素与增加元素一样，需要维护整个二叉堆的序。删除位置1的元素（数组下标0），则把最后一个元素空出来移到最前边，然后和它的两个子节点比较，如果两个子节点中较小的节点小于该节点，就将他们交换，知道两个子节点都比该元素大为止。

> PriorityBlockingQueue

```
// 默认容量
    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    // 最大容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 二叉堆数组
    private transient Object[] queue;

    // 队列元素的个数
    private transient int size;

    // 比较器，如果为空，则为自然顺序
    private transient Comparator<? super E> comparator;

    // 内部锁
    private final ReentrantLock lock;

    private final Condition notEmpty;

    //
    private transient volatile int allocationSpinLock;

    // 优先队列：主要用于序列化，这是为了兼容之前的版本。只有在序列化和反序列化才非空
    private PriorityQueue<E> q;
```

内部仍然采用可重入锁ReentrantLock来实现同步机制，但是这里只有一个notEmpty的Condition，了解了ArrayBlockingQueue我们知道它定义了两个Condition，之类为何只有一个呢？

**原因就在于PriorityBlockingQueue是一个无界队列，插入总是会成功，除非消耗尽了资源导致服务器挂。**

PriorityBlockingQueue是无界的，所以不可能会阻塞。

> 内部调用offer(E e)：
```
public boolean offer(E e) {
        // 不能为null
        if (e == null)
            throw new NullPointerException();
        // 获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        int n, cap;
        Object[] array;
        // 扩容
        while ((n = size) >= (cap = (array = queue).length))
            tryGrow(array, cap);
        try {
            Comparator<? super E> cmp = comparator;
            // 根据比较器是否为null，做不同的处理
            if (cmp == null)
                siftUpComparable(n, e, array);
            else
                siftUpUsingComparator(n, e, array, cmp);
            size = n + 1;
            // 唤醒正在等待的消费者线程
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
        return true;
    }
```
整个添加元素的过程和上面二叉堆一模一样：先将元素添加到数组末尾，然后采用“上冒”的方式将该元素尽量往上冒。

> 出列

PriorityBlockingQueue提供poll()、remove()方法来执行出对操作。出对的永远都是第一个元素：array[0]。
```
public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

    private E dequeue() {
          // 没有元素 返回null
          int n = size - 1;
          if (n < 0)
              return null;
          else {
              Object[] array = queue;
              // 出对元素
              E result = (E) array[0];
              // 最后一个元素（也就是插入到空穴中的元素）
              E x = (E) array[n];
              array[n] = null;
              // 根据比较器释放为null，来执行不同的处理
              Comparator<? super E> cmp = comparator;
              if (cmp == null)
                  siftDownComparable(0, x, array, n);
              else
                  siftDownUsingComparator(0, x, array, n, cmp);
              size = n;
              return result;
          }
      }
```

PriorityBlockingQueue采用二叉堆来维护，所以整个处理过程不是很复杂，添加操作则是不断“上冒”，而删除操作则是不断“下掉”。掌握二叉堆就掌握了PriorityBlockingQueue，无论怎么变还是。

对于PriorityBlockingQueue需要注意的是他是一个无界队列，所以添加操作是不会失败的，除非资源耗尽。
