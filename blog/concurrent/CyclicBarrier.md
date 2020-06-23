#### CyclicBarrier

 它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

 通俗点讲就是：让一组线程到达一个屏障时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

 CyclicBarrier的内部是使用重入锁ReentrantLock和Condition。

它有两个构造函数：

- CyclicBarrier(int parties)：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。

- CyclicBarrier(int parties, Runnable barrierAction) ：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

```
CyclicBarrier(int parties)
创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
CyclicBarrier(int parties, Runnable barrierAction)
创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

int await()
在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。
int await(long timeout, TimeUnit unit)
在所有参与者都已经在此屏障上调用 await 方法之前将一直等待,或者超出了指定的等待时间。
int getNumberWaiting()
返回当前在屏障处等待的参与者数目。
int getParties()
返回要求启动此 barrier 的参与者数目。
boolean isBroken()
查询此屏障是否处于损坏状态。
void reset()
将屏障重置为其初始状态。
```
在CyclicBarrier中最重要的方法莫过于await()方法，在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。如下：
```
public int await() throws InterruptedException, BrokenBarrierException {
       try {
           return dowait(false, 0L);//不超时等待
       } catch (TimeoutException toe) {
           throw new Error(toe); // cannot happen
       }
   }


   private int dowait(boolean timed, long nanos)
       throws InterruptedException, BrokenBarrierException,
              TimeoutException {
       final ReentrantLock lock = this.lock;
       // 获取“独占锁(lock)”
       lock.lock();
       try {
           // 保存“当前的generation”
           final Generation g = generation;

           // 若“当前generation已损坏”，则抛出异常。
           if (g.broken)
               throw new BrokenBarrierException();

           // 如果当前线程被中断，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
           if (Thread.interrupted()) {
               breakBarrier();
               throw new InterruptedException();
           }

          // 将“count计数器”-1
          int index = --count;
          // 如果index=0，则意味着“有parties个线程到达barrier”。
          if (index == 0) {  // tripped
              boolean ranAction = false;
              try {
                  // 如果barrierCommand不为null，则执行该动作。
                  final Runnable command = barrierCommand;
                  if (command != null)
                      command.run();
                  ranAction = true;
                  // 唤醒所有等待线程，并更新generation。
                  nextGeneration();
                  return 0;
              } finally {
                  if (!ranAction)
                      breakBarrier();
              }
          }

           // 当前线程一直阻塞，直到“有parties个线程到达barrier” 或 “当前线程被中断” 或 “超时”这3者之一发生，
           // 当前线程才继续执行。
           for (;;) {
               try {
                   // 如果不是“超时等待”，则调用awati()进行等待；否则，调用awaitNanos()进行等待。
                   if (!timed)
                       trip.await();
                   else if (nanos > 0L)
                       nanos = trip.awaitNanos(nanos);
               } catch (InterruptedException ie) {
                   // 如果等待过程中，线程被中断，则执行下面的函数。
                   if (g == generation && ! g.broken) {
                       breakBarrier();
                       throw ie;
                   } else {
                       Thread.currentThread().interrupt();
                   }
               }

               // 如果“当前generation已经损坏”，则抛出异常。
               if (g.broken)
                   throw new BrokenBarrierException();

               // 如果“generation已经换代”，则返回index。
               if (g != generation)
                   return index;

               // 如果是“超时等待”，并且时间已到，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
               if (timed && nanos <= 0L) {
                   breakBarrier();
                   throw new TimeoutException();
               }
           }
       } finally {
           // 释放“独占锁(lock)”
           lock.unlock();
       }
   }
```
其实await()的处理逻辑还是比较简单的：如果该线程不是到达的最后一个线程，则他会一直处于等待状态，除非发生以下情况：
```
最后一个线程到达，即index == 0
超出了指定时间（超时等待）
其他的某个线程中断当前线程
其他的某个线程中断另一个等待的线程
其他的某个线程在等待barrier超时
其他的某个线程在此barrier调用reset()方法。reset()方法用于将屏障重置为初始状态。
```
在上述代码中我们总是可以看到抛出BrokenBarrierException异常，那么什么时候抛出异常呢？

如果一个线程处于等待状态时，如果其他线程调用reset()，或者调用的barrier原本就是被损坏的，则抛出BrokenBarrierException异常。

同时，任何线程在等待时被中断了，则其他所有线程都将抛出BrokenBarrierException异常，并将barrier置于损坏状态。

同时，Generation描述着CyclicBarrier的更显换代。

在CyclicBarrier中，同一批线程属于同一代。当有parties个线程到达barrier，generation就会被更新换代。其中broken标识该当前CyclicBarrier是否已经处于中断状态。

默认barrier是没有损坏的。 当barrier损坏了或者有一个线程中断了，则通过breakBarrier()来终止所有的线程：
```
private void breakBarrier() {
      generation.broken = true;
      count = parties;
      trip.signalAll();
  }
```
在breakBarrier()中除了将broken设置为true，还会调用signalAll将在CyclicBarrier处于等待状态的线程全部唤醒。 当所有线程都已经到达barrier处（index == 0），则会通过nextGeneration()进行更新换地操作，在这个步骤中，做了三件事：唤醒所有线程，重置count，generation。
```
private void nextGeneration() {
        trip.signalAll();
        count = parties;
        generation = new Generation();
    }
```

CyclicBarrier同时也提供了await(long timeout, TimeUnit unit) 方法来做超时控制，内部还是通过调用doawait()实现的。
