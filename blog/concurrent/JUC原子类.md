#### JUC原子类

根据修改的数据类型，可以将JUC包中的原子操作类可以分为4类。

1. 基本类型: AtomicInteger, AtomicLong, AtomicBoolean ;
2. 数组类型: AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray ;
3. 引用类型: AtomicReference, AtomicStampedRerence, AtomicMarkableReference ;
4. 对象的属性修改类型: AtomicIntegerFieldUpdater, AtomicLongFieldUpdater, AtomicReferenceFieldUpdater 。

这些类存在的目的是对相应的数据进行原子操作。所谓原子操作，是指操作过程不会被中断，保证数据操作是以原子方式进行的。

> ABA

为了解决ABA问题，AtomicStampedReference是利用版本戳的形式记录了**每次改变以后的版本号**，这样的话就不会存在ABA问题了，在这里我借鉴一下别人举得例子
```
import java.util.concurrent.atomic.AtomicStampedReference;
public class AtomicStampedReferenceTest {
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference(0, 0);
    public static void main(String[] args) throws InterruptedException {
        final int stamp = atomicStampedReference.getStamp();
        final Integer reference = atomicStampedReference.getReference();
        System.out.println(reference+"============"+stamp);
        Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
                System.out.println(reference + "-" + stamp + "-"
                + atomicStampedReference.compareAndSet(reference, reference + 10, stamp, stamp + 1));
        }
        });

        Thread t2 = new Thread(new Runnable() {
        @Override
        public void run() {
                Integer reference = atomicStampedReference.getReference();
                System.out.println(reference + "-" + stamp + "-"
                + atomicStampedReference.compareAndSet(reference, reference + 10, stamp, stamp + 1));
        }
        });
        t1.start();
        t1.join();
        t2.start();
        t2.join();

        System.out.println(atomicStampedReference.getReference());
        System.out.println(atomicStampedReference.getStamp());
    }
}

0============0
0-0-true
10-0-false
10
1
```
AtomicStampedReference只要时间戳变量每次改变增加1，就可以知道总共改变了几次

AtomicMarkableReference跟AtomicStampedReference差不多.

AtomicStampedReference是使用pair的int stamp作为计数器使用，AtomicMarkableReference的pair使用的是boolean mark。

AtomicStampedReference可能关心的是动过几次，AtomicMarkableReference关心的是有没有被人动过，方法都比较简单.
