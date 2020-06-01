# JAVA 查漏补缺

> 死锁

```

public class DeadLockSample extends Thread {
  private String first;
  private String second;
  public DeadLockSample(String name, String first, String second) {
      super(name);
      this.first = first;
      this.second = second;
  }

  public  void run() {
      synchronized (first) {
          System.out.println(this.getName() + " obtained: " + first);
          try {
              Thread.sleep(1000L);
              synchronized (second) {
                  System.out.println(this.getName() + " obtained: " + second);
              }
          } catch (InterruptedException e) {
              // Do nothing
          }
      }
  }
  public static void main(String[] args) throws InterruptedException {
      String lockA = "lockA";
      String lockB = "lockB";
      DeadLockSample t1 = new DeadLockSample("Thread1", lockA, lockB);
      DeadLockSample t2 = new DeadLockSample("Thread2", lockB, lockA);
      t1.start();
      t2.start();
      t1.join();
      t2.join();
  }
}
```

**join**

- 在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，**主线程往往将于子线程之前结束**，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

- join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

查找死锁

- 首先，可以使用 jps 或者系统的 ps 命令、任务管理器等工具，确定进程 ID。

- 其次，调用 jstack 获取线程栈：
```
${JAVA_HOME}\bin\jstack your_pid
```
