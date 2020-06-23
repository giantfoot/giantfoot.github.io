#### ReentrantReadWriteLock

重入锁ReentrantLock是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

读写锁维护着一对锁，一个读锁和一个写锁。

通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

```
公平性：支持公平性和非公平性。
重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁
```

源码

```
/** 内部类  读锁 */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** 内部类  写锁 */
private final ReentrantReadWriteLock.WriteLock writerLock;

final Sync sync;

/** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock() {
    this(false);
}

/** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

/** 返回用于写入操作的锁 */
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
/** 返回用于读取操作的锁 */
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

abstract static class Sync extends AbstractQueuedSynchronizer {
    /**
     * 省略其余源代码
     */
}
public static class WriteLock implements Lock, java.io.Serializable{
    /**
     * 省略其余源代码
     */
}

public static class ReadLock implements Lock, java.io.Serializable {
    /**
     * 省略其余源代码
     */
}
```

在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。

但是读写锁ReentrantReadWriteLock内部维护着两个一对锁，需要用一个变量维护多种状态。

所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，高16为表示读，低16为表示写。

分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？

通过位运算：假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。代码如下：
```
   static final int SHARED_SHIFT   = 16;
   static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
   static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
   static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

   static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
   static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```
写锁的获取最终会调用tryAcquire(int arg)，该方法在内部类Sync中实现：
```
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    //当前锁个数
    int c = getState();
    //写锁
    int w = exclusiveCount(c);
    if (c != 0) {
        //c != 0 && w == 0 表示存在读锁
        //当前线程不是已经获取写锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //超出最大范围
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    //是否需要阻塞
    if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
        return false;
    //设置获取锁的线程为当前线程
    setExclusiveOwnerThread(current);
    return true;
}
```
该方法和ReentrantLock的tryAcquire(int arg)大致一样，在判断重入时增加了一项条件：读锁是否存在。因为要确保写锁的操作对读锁是可见的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。

获取了写锁用完了则需要释放，WriteLock提供了unlock()方法释放写锁：
```
public void unlock() {
      sync.release(1);
  }

  public final boolean release(int arg) {
      if (tryRelease(arg)) {
          Node h = head;
          if (h != null && h.waitStatus != 0)
              unparkSuccessor(h);
          return true;
      }
      return false;
  }
```
写锁的释放最终还是会调用AQS的模板方法release(int arg)方法，该方法首先调用tryRelease(int arg)方法尝试释放锁，tryRelease(int arg)方法为读写锁内部类Sync中定义了，如下：
```
protected final boolean tryRelease(int releases) {
       //释放的线程不为锁的持有者
       if (!isHeldExclusively())
           throw new IllegalMonitorStateException();
       int nextc = getState() - releases;
       //若写锁的新线程数为0，则将锁的持有者设置为null
       boolean free = exclusiveCount(nextc) == 0;
       if (free)
           setExclusiveOwnerThread(null);
       setState(nextc);
       return free;
   }
```
写锁释放锁的整个过程和独占锁ReentrantLock相似，每次释放均是减少写状态，当写状态为0时表示 写锁已经完全释放了，从而等待的其他线程可以继续访问读写锁，获取同步状态，同时此次写线程的修改对后续的线程可见。


> 读锁

读锁为一个可重入的共享锁，它能够被多个线程同时持有，在没有其他写线程访问时，读锁总是或获取成功。

tryAcqurireShared(int arg)尝试获取读同步状态，该方法主要用于获取共享式同步状态，获取成功返回 >= 0的返回结果，否则返回 < 0 的返回结果。
```
protected final int tryAcquireShared(int unused) {
       //当前线程
       Thread current = Thread.currentThread();
       int c = getState();
       //exclusiveCount(c)计算写锁
       //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
       //存在锁降级问题，后续阐述
       if (exclusiveCount(c) != 0 &&
               getExclusiveOwnerThread() != current)
           return -1;
       //读锁
       int r = sharedCount(c);

       /*
        * readerShouldBlock():读锁是否需要等待（公平锁原则）
        * r < MAX_COUNT：持有线程小于最大数（65535）
        * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
        */
       if (!readerShouldBlock() &&
               r < MAX_COUNT &&
               compareAndSetState(c, c + SHARED_UNIT)) {
           /*
            * holdCount部分后面讲解
            */
           if (r == 0) {
               firstReader = current;
               firstReaderHoldCount = 1;
           } else if (firstReader == current) {
               firstReaderHoldCount++;
           } else {
               HoldCounter rh = cachedHoldCounter;
               if (rh == null || rh.tid != getThreadId(current))
                   cachedHoldCounter = rh = readHolds.get();
               else if (rh.count == 0)
                   readHolds.set(rh);
               rh.count++;
           }
           return 1;
       }
       return fullTryAcquireShared(current);
   }
```

读锁获取的过程相对于独占锁而言会稍微复杂下，整个过程如下：
因为存在锁降级情况，如果存在写锁且锁的持有者不是当前线程则直接返回失败，否则继续
依据公平性原则，判断读锁是否需要阻塞，读锁持有线程数小于最大值（65535），且设置锁状态成功，执行以下代码（对于HoldCounter下面再阐述），并返回1。如果不满足改条件，执行fullTryAcquireShared()。

```
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        //锁降级
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
        }
        //读锁需要阻塞
        else if (readerShouldBlock()) {
            //列头为当前线程
            if (firstReader == current) {
            }
            //HoldCounter后面讲解
            else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        //读锁超出最大范围
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        //CAS设置读锁成功
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            //如果是第1次获取“读取锁”，则更新firstReader和firstReaderHoldCount
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            }
            //如果想要获取锁的线程(current)是第1个获取锁(firstReader)的线程,则将firstReaderHoldCount+1
            else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                //更新线程的获取“读取锁”的共享计数
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```


锁降级

上开篇是LZ就阐述了读写锁有一个特性就是锁降级，锁降级就意味着写锁是可以降级为读锁的，但是需要遵循先获取写锁、获取读锁在释放写锁的次序。注意如果当前线程先获取写锁，然后释放写锁，再获取读锁这个过程不能称之为锁降级，锁降级一定要遵循那个次序。 在获取读锁的方法tryAcquireShared(int unused)中，有一段代码就是来判读锁降级的：

```
int c = getState();
      //exclusiveCount(c)计算写锁
      //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
      //存在锁降级问题，后续阐述
      if (exclusiveCount(c) != 0 &&
              getExclusiveOwnerThread() != current)
          return -1;
      //读锁
      int r = sharedCount(c);
```

> 总结

在线程持有读锁的情况下，该线程不能取得写锁(因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有)。

在线程持有写锁的情况下，该线程可以继续获取读锁（获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败）。

仔细想想，这个设计是合理的：因为当线程获取读锁的时候，可能有其他线程同时也在持有读锁，因此不能把获取读锁的线程“升级”为写锁；而对于获得写锁的线程，它一定独占了写锁，因此可以继续让它获取读锁，当它同时获取了写锁和读锁后，还可以先释放写锁继续持有读锁，这样一个写锁就“降级”为了读锁。

综上：
一个线程要想同时持有写锁和读锁，必须先获取写锁再获取读锁；写锁可以“降级”为读锁；读锁不能“升级”为写锁。
