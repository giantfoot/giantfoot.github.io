#### fail-fast 解析

由iterator()和listIterator()返回的迭代器是fail-fast的。

在于程序在对list进行迭代时，某个线程对该collection在结构上对其做了修改，这时迭代器就会抛出ConcurrentModificationException异常信息(**不是100%**)。

因此，面对并发的修改，迭代器快速而干净利落地失败，而不是在不确定的情况下冒险。

由elements()返回的Enumerations不是fail-fast的。

需要注意的是，迭代器的fail-fast并不能得到保证，它不能够保证一定出现该错误。

一般来说，fail-fast会尽最大努力抛出ConcurrentModificationException异常。因此，为提高此类操作的正确性而编写一个依赖于此异常的程序是错误的做法，正确做法是：ConcurrentModificationException 应该仅用于检测 bug。

大意为在遍历一个集合时，当集合结构被修改，很有可能 会抛出Concurrent Modification Exception。

为什么说是很有可能呢？

迭代器的remove操作（注意是迭代器的remove方法而不是集合的remove方法）修改集合结构就不会导致这个异常。

看到这里我们就明白了，fail-fast 机制是java容器（Collection和Map都存在fail-fast机制）中的一种错误机制。在遍历一个容器对象时，当容器结构被修改，很有可能会抛出ConcurrentModificationException，产生fail-fast。

**什么时候会出现fail-fast？**

- 单线程环境
> 遍历一个集合过程中，集合结构被修改。（比如使用for循环删除元素）。注意，listIterator.remove()方法修改集合结构不会抛出这个异常。

- 多线程环境
>当一个线程遍历集合过程中，而另一个线程对集合结构进行了修改。

单线程测试
```
import java.util.ListIterator;
    import java.util.Vector;

    public class Test {
    /**
         * 单线程测试
         */
        @org.junit.Test
        public void test() {
            try {
                // 测试迭代器的remove方法修改集合结构会不会触发checkForComodification异常
                ItrRemoveTest();
                System.out.println("----分割线----");
                // 测试集合的remove方法修改集合结构会不会触发checkForComodification异常
                ListRemoveTest();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        // 测试迭代器的remove方法修改集合结构会不会触发checkForComodification异常
        private void ItrRemoveTest() {
            Vector list = new Vector<>();
            list.add("1");
            list.add("2");
            list.add("3");
            ListIterator itr = list.listIterator();
            while (itr.hasNext()) {
                System.out.println(itr.next());
                //迭代器的remove方法修改集合结构，不会发生异常
                itr.remove();
            }
        }

        // 测试集合的remove方法修改集合结构会不会触发checkForComodification异常
        private void ListRemoveTest() {
            Vector list = new Vector<>();
            list.add("1");
            list.add("2");
            list.add("3");
            ListIterator itr = list.listIterator();
            while (itr.hasNext()) {
                System.out.println(itr.next());
                //集合的remove方法修改集合结构，会发生异常
                list.remove("3");
            }
        }
    }
```

多线程测试
```
import java.util.Iterator;
    import java.util.List;
    import java.util.ListIterator;
    import java.util.Vector;

    public class Test {
        private static List<String> list = new Vector<String>();

        /**
         * 多线程情况测试
         */
        @org.junit.Test
        public void test2() {
            list.add("1");
            list.add("2");
            list.add("3");
            // 同时启动两个线程对list进行操作！
            new ErgodicThread().start();
            new ModifyThread().start();
        }

        /**
         * 遍历集合的线程
         */
        private static class ErgodicThread extends Thread {
            public void run() {
                int i = 0;
                while (i < 10) {
                    printAll();
                    i++;
                }
            }
        }

        /**
         * 修改集合的线程
         */
        private static class ModifyThread extends Thread {
            public void run() {
                list.add(String.valueOf("5"));
            }
        }
        /**
         * 遍历集合
         */
        private static void printAll() {
            Iterator iter = list.iterator();
            while (iter.hasNext()) {
                System.out.print((String) iter.next() + ", ");
            }
            System.out.println();
        }
    }
```

**fail-fast实现原理**
```
/**
    * An optimized version of AbstractList.Itr
    */
   private class Itr implements Iterator<E> {
       int expectedModCount = modCount;

       //省略的部分代码

       public void remove() {
           if (lastRet == -1)
               throw new IllegalStateException();
           synchronized (Vector.this) {
               checkForComodification();
               Vector.this.remove(lastRet);
               expectedModCount = modCount;
           }
           cursor = lastRet;
           lastRet = -1;
       }

       @Override
       public void forEachRemaining(Consumer<? super E> action) {
               //省略的部分代码
               checkForComodification();
           }
       }

       final void checkForComodification() {
           if (modCount != expectedModCount)
               throw new ConcurrentModificationException();
       }
   }
```

每次初始化一个迭代器都会执行int expectedModCount = modCount;迭代器遍历的时候通过checkForComodification()方法判断modCount和expectedModCount 是否相等，如果不相等就表示已经有线程修改了集合结构。

使用迭代器的remove()方法修改集合结构不会触发ConcurrentModificationException，现在可以在源码中看出来是为什么。

**在remove()方法的最后会执行expectedModCount = modCount；**这样itr.remove操作后modCount和expectedModCount依然相等，就不会触发ConcurrentModificationException了。
