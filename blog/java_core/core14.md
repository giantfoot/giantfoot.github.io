# JAVA 查漏补缺

> synchronized和ReentrantLock

- 在 Java 5 以前，synchronized 是仅有的同步手段，在代码中， synchronized 可以用来修饰方法，也可以使用在特定的代码块儿上，本质上 synchronized 方法等同于把方法全部语句用 synchronized 块包起来。

- ReentrantLock，通常翻译为再入锁，是 Java 5 提供的锁实现，它的语义和 synchronized 基本相同。再入锁通过代码直接调用 lock() 方法获取，代码书写也更加灵活。与此同时，ReentrantLock 提供了很多实用的方法，能够实现很多 synchronized 无法做到的细节控制，比如可以控制 fairness，也就是公平性，或者利用定义条件等。但是，编码中也需要注意，必须要明确调用 unlock() 方法释放，不然就会一直持有该锁。

- synchronized 和 ReentrantLock 的性能不能一概而论，早期版本 synchronized 在很多场景下性能相差较大，在后续版本进行了较多改进，在低竞争场景中表现可能优于 ReentrantLock。

线程安全需要保证几个基本特性：

- 原子性，简单说就是相关操作不会中途被其他线程干扰，一般通过同步机制实现。

- 可见性，是一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到主内存上，volatile 就是负责保证可见性的。
- 有序性，是保证线程内串行语义，避免指令重排等。

代码中使用 synchronized 非常便利，如果用来修饰静态方法，其等同于利用下面代码将方法体囊括进来：
```

synchronized (ClassName.class) {}
```

再来看看 ReentrantLock。

你可能好奇什么是再入？

**它是表示当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功**，这是对锁获取粒度的一个概念，也就是锁的持有是以线程为单位而不是基于调用次数。

再入锁可以设置公平性（fairness），我们可在创建再入锁时选择是否是公平的。
```

ReentrantLock fairLock = new ReentrantLock(true);
```

这里所谓的公平性是指在竞争场景中，当公平性为真时，**会倾向于** 将锁赋予等待时间最久的线程。公平性是**减少** 线程“饥饿”（个别线程长期等待锁，但始终无法获取）情况发生的一个办法。

如果使用 synchronized，我们根本无法进行公平性的选择，**其永远是不公平的**，这也是主流操作系统线程调度的选择。**通用场景中，公平性未必有想象中的那么重要**，Java 默认的调度策略很少会导致 “饥饿”发生。

与此同时，若要保证**公平性则会引入额外开销**，自然会导致一定的吞吐量下降。所以，我建议只有当你的程序确实有公平性需要的时候，才有必要指定它。

为保证锁释放，每一个 lock() 动作，我建议都立即对应一个 try-catch-finally，典型的代码结构如下，这是个良好的习惯。

```

ReentrantLock fairLock = new ReentrantLock(true);// 这里是演示创建公平锁，一般情况不需要。
fairLock.lock();
try {
  // do something
} finally {
   fairLock.unlock();
}
```

如果说 ReentrantLock 是 synchronized 的替代选择，Condition 则是将 wait、notify、notifyAll 等操作转化为相应的对象，将复杂而晦涩的同步操作转变为直观可控的对象行为。
