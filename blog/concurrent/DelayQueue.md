#### DelayQueue

DelayQueue是一个支持延时获取元素的无界阻塞队列。里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行。也就是说只有在延迟期到时才能够从队列中取元素。 DelayQueue主要用于两个方面：

- 缓存：清掉缓存中超时的缓存数据
- 任务超时处理

DelayQueue实现的关键主要有如下几个：
```
可重入锁ReentrantLock
用于阻塞和通知的Condition对象
根据Delay时间排序的优先级队列：PriorityQueue
用于优化阻塞通知的线程元素leader
```

> Delayed

Delayed接口是用来标记那些应该在给定延迟时间之后执行的对象，它定义了一个long getDelay(TimeUnit unit)方法，该方法返回与此对象相关的的剩余时间。同时实现该接口的对象必须定义一个compareTo 方法，该方法提供与此接口的 getDelay 方法一致的排序。
```
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
        implements BlockingQueue<E> {
    /** 可重入锁 */
    private final transient ReentrantLock lock = new ReentrantLock();
    /** 支持优先级的BlockingQueue */
    private final PriorityQueue<E> q = new PriorityQueue<E>();
    /** 用于优化阻塞 */
    private Thread leader = null;
    /** Condition */
    private final Condition available = lock.newCondition();

    /**
     * 省略很多代码
     */
}

```
DelayQueue的元素都必须继承Delayed接口。

同时也可以从这里初步理清楚DelayQueue内部实现的机制了：

以支持优先级无界队列的PriorityQueue作为一个容器，容器里面的元素都应该实现Delayed接口，在每次往优先级队列中添加元素时以元素的过期时间作为排序条件，最先过期的元素放在优先级最高。

```
public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 向 PriorityQueue中插入元素
            q.offer(e);
            // 如果当前元素的对首元素（优先级最高），leader设置为空，唤醒所有等待线程
            if (q.peek() == e) {
                leader = null;
                available.signal();
            }
            // 无界队列，永远返回true
            return true;
        } finally {
            lock.unlock();
        }
    }
```

> take()

```
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                // 对首元素
                E first = q.peek();
                // 对首为空，阻塞，等待off()操作唤醒
                if (first == null)
                    available.await();
                else {
                    // 获取对首元素的超时时间
                    long delay = first.getDelay(NANOSECONDS);
                    // <=0 表示已过期，出对，return
                    if (delay <= 0)
                        return q.poll();
                    first = null; // don't retain ref while waiting
                    // leader != null 证明有其他线程在操作，阻塞
                    if (leader != null)
                        available.await();
                    else {
                        // 否则将leader 设置为当前线程，独占
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            // 超时阻塞
                            available.awaitNanos(delay);
                        } finally {
                            // 释放leader
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            // 唤醒阻塞线程
            if (leader == null && q.peek() != null)
                available.signal();
            lock.unlock();
        }
    }
```

首元素的延时时间 delay <= 0 ，则可以出对了，直接return即可。

如果 leader != null 则表示当前有线程占用，则阻塞，否则设置leader为当前线程，然后调用awaitNanos()方法超时等待。

通过leader来减少不必要阻塞。
