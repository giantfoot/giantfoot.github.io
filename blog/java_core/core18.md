# JAVA 查漏补缺

> JUC

CountDownLatch，允许一个或多个线程等待某些操作完成。

CyclicBarrier，一种辅助性的同步结构，允许多个线程等待到达某个屏障。

Semaphore，Java 版本的信号量实现。

Java 提供了经典信号量（Semaphore）的实现，它通过控制一定数量的允许（permit）的方式，来达到限制通用资源访问的目的。

你可以想象一下这个场景，在车站、机场等出租车时，当很多空出租车就位时，为防止过度拥挤，调度员指挥排队等待坐车的队伍一次进来 5 个人上车，等这 5 个人坐车出发，再放进去下一批，这和 Semaphore 的工作原理有些类似。你可以试试使用 Semaphore 来模拟实现这个调度过程：

- 每释放一个便新加一个：
```
import java.util.concurrent.Semaphore;
public class UsualSemaphoreSample {
  public static void main(String[] args) throws InterruptedException {
      System.out.println("Action...GO!");
      Semaphore semaphore = new Semaphore(5);
      for (int i = 0; i < 10; i++) {
          Thread t = new Thread(new SemaphoreWorker(semaphore));
          t.start();
      }
  }
}
class SemaphoreWorker implements Runnable {
  private String name;
  private Semaphore semaphore;
  public SemaphoreWorker(Semaphore semaphore) {
      this.semaphore = semaphore;
  }
  @Override
  public void run() {
      try {
          log("is waiting for a permit!");
         semaphore.acquire();
          log("acquired a permit!");
          log("executed!");
      } catch (InterruptedException e) {
          e.printStackTrace();
      } finally {
          log("released a permit!");
          semaphore.release();
      }
  }
  private void log(String msg){
      if (name == null) {
          name = Thread.currentThread().getName();
      }
      System.out.println(name + " " + msg);
  }
}
```
- 五个一批，分批次进入和离开
```
import java.util.concurrent.Semaphore;
public class AbnormalSemaphoreSample {
  public static void main(String[] args) throws InterruptedException {
      Semaphore semaphore = new Semaphore(0);
      for (int i = 0; i < 10; i++) {
          Thread t = new Thread(new MyWorker(semaphore));
          t.start();
      }
      System.out.println("Action...GO!");
      semaphore.release(5);
      System.out.println("Wait for permits off");
      while (semaphore.availablePermits()!=0) {
          Thread.sleep(100L);
      }
      System.out.println("Action...GO again!");
      semaphore.release(5);
  }
}
class MyWorker implements Runnable {
  private Semaphore semaphore;
  public MyWorker(Semaphore semaphore) {
      this.semaphore = semaphore;
  }
  @Override
  public void run() {
      try {
          semaphore.acquire();
          System.out.println("Executed!");
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
  }
}
```

**CountDownLatch 和 CyclicBarrier**

CountDownLatch 是不可以重置的，所以无法重用；而 CyclicBarrier 则没有这种限制，可以重用。

CountDownLatch 的基本操作组合是 countDown/await。调用 await 的线程阻塞等待 countDown 足够的次数，**不管你是在一个线程还是多个线程里 countDown，只要次数足够即可**。所以就像 Brain Goetz 说过的，CountDownLatch 操作的是事件。

CyclicBarrier 的基本操作组合，则就是 await，当所有的伙伴（parties）都调用了 await，才会继续进行任务，并自动进行重置。

注意，正常情况下，CyclicBarrier 的重置都是自动发生的，如果我们调用 reset 方法，但还有线程在等待，就会导致等待线程被打扰，**抛出 BrokenBarrierException 异常**。CyclicBarrier 侧重点是线程，而不是调用事件，它的典型应用场景是用来等待并发线程结束。


**CountDownLatch**

这个例子也从侧面体现出了它的局限性，虽然它也能够支持 10 个人排队的情况，但是因为不能重用，如果要支持更多人排队，就不能依赖一个 CountDownLatch 进行了。
```

import java.util.concurrent.CountDownLatch;
public class LatchSample {
  public static void main(String[] args) throws InterruptedException {
      CountDownLatch latch = new CountDownLatch(6);
           for (int i = 0; i < 5; i++) {
                Thread t = new Thread(new FirstBatchWorker(latch));
                t.start();
      }
      for (int i = 0; i < 5; i++) {
              Thread t = new Thread(new SecondBatchWorker(latch));
              t.start();
      }
           // 注意这里也是演示目的的逻辑，并不是推荐的协调方式
      while ( latch.getCount() != 1 ){
              Thread.sleep(100L);
      }
      System.out.println("Wait for first batch finish");
      latch.countDown();
  }
}
class FirstBatchWorker implements Runnable {
  private CountDownLatch latch;
  public FirstBatchWorker(CountDownLatch latch) {
      this.latch = latch;
  }
  @Override
  public void run() {
          System.out.println("First batch executed!");
          latch.countDown();
  }
}
class SecondBatchWorker implements Runnable {
  private CountDownLatch latch;
  public SecondBatchWorker(CountDownLatch latch) {
      this.latch = latch;
  }
  @Override
  public void run() {
      try {
          latch.await();
          System.out.println("Second batch executed!");
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
  }
}

```

**CyclicBarrier**

因为 CyclicBarrier 会自动进行重置，所以这个逻辑其实可以非常自然的支持更多排队人数。
```

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
public class CyclicBarrierSample {
  public static void main(String[] args) throws InterruptedException {
      CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {
          @Override
          public void run() {
              System.out.println("Action...GO again!");
          }
      });
      for (int i = 0; i < 5; i++) {
          Thread t = new Thread(new CyclicWorker(barrier));
          t.start();
      }
  }
  static class CyclicWorker implements Runnable {
      private CyclicBarrier barrier;
      public CyclicWorker(CyclicBarrier barrier) {
          this.barrier = barrier;
      }
      @Override
      public void run() {
          try {
              for (int i=0; i<3 ; i++){
                  System.out.println("Executed!");
                  barrier.await();
              }
          } catch (BrokenBarrierException e) {
              e.printStackTrace();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
}
```


普通无顺序场景选择 HashMap，有顺序场景则可以选择类似 TreeMap 等，但是为什么并发容器里面没有 ConcurrentTreeMap 呢？

> 这是因为 TreeMap 要实现高效的线程安全是非常困难的，它的实现基于复杂的红黑树。为保证访问效率，当我们插入或删除节点时，会移动节点进行平衡操作，这导致在并发场景中难以进行合理粒度的同步。而 SkipList 结构则要相对简单很多，通过层次结构提高访问速度，虽然不够紧凑，空间使用有一定提高（O(nlogn)），但是在增删元素时线程安全的开销要好很多。

CopyOnWriteArraySet 是通过包装了 CopyOnWriteArrayList 来实现的，所以在学习时，我们可以专注于理解一种。

CopyOnWrite 到底是什么意思呢？

它的原理是，任何修改操作，如 add、set、remove，都会拷贝原数组，修改后替换原来的数组，通过这种防御性的方式，实现另类的线程安全。**所以这种数据结构，相对比较适合读多写少的操作，不然修改的开销还是非常明显的。**

```

public boolean add(E e) {
  synchronized (lock) {
      Object[] elements = getArray();
      int len = elements.length;
           // 拷贝
      Object[] newElements = Arrays.copyOf(elements, len + 1);
      newElements[len] = e;
           // 替换
      setArray(newElements);
      return true;
            }
}
final void setArray(Object[] a) {
  array = a;
}
```
