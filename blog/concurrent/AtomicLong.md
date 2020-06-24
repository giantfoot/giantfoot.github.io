#### AtomicLong

AtomicInteger, AtomicLong和AtomicBoolean这3个基本类型的原子类的原理和用法相似。

在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

而int虽然不是64位，没有原子性的困扰，但为了需要保证线程安全，也提供了AtomicInteger。

```
// 构造函数
AtomicLong()
// 创建值为initialValue的AtomicLong对象
AtomicLong(long initialValue)
// 以原子方式设置当前值为newValue。
final void set(long newValue)
// 获取当前值
final long get()
// 以原子方式将当前值减 1，并返回减1后的值。等价于“--num”
final long decrementAndGet()
// 以原子方式将当前值减 1，并返回减1前的值。等价于“num--”
final long getAndDecrement()
// 以原子方式将当前值加 1，并返回加1后的值。等价于“++num”
final long incrementAndGet()
// 以原子方式将当前值加 1，并返回加1前的值。等价于“num++”
final long getAndIncrement()    
// 以原子方式将delta与当前值相加，并返回相加后的值。
final long addAndGet(long delta)
// 以原子方式将delta添加到当前值，并返回相加前的值。
final long getAndAdd(long delta)
// 如果当前值 == expect，则以原子方式将该值设置为update。成功返回true，否则返回false，并且不修改原值。
final boolean compareAndSet(long expect, long update)
// 以原子方式设置当前值为newValue，并返回旧值。
final long getAndSet(long newValue)
// 返回当前值对应的int值
int intValue()
// 获取当前值对应的long值
long longValue()    
// 以 float 形式返回当前值
float floatValue()    
// 以 double 形式返回当前值
double doubleValue()    
// 最后设置为给定值。延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。
final void lazySet(long newValue)
// 如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。
final boolean weakCompareAndSet(long expect, long update)
```

AtomicLong的代码很简单，下面仅以incrementAndGet()为例，对AtomicLong的原理进行说明。
```
public final long incrementAndGet() {
    for (;;) {
        // 获取AtomicLong当前对应的long值
        long current = get();
        // 将current加1
        long next = current + 1;
        // 通过CAS函数，更新current的值
        if (compareAndSet(current, next))
            return next;
    }
}
// value是AtomicLong对应的long值
private volatile long value;
// 返回AtomicLong对应的long值
public final long get() {
    return value;
}


public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```

incrementAndGet这个函数里面有个死循环,这是乐观锁实现，volatile+CAS实现，更新不成功继续循环更新，效率比每次都加锁(悲观锁)高很多的。如果compareAndSet更新失败，就继续操作，直到成功。
