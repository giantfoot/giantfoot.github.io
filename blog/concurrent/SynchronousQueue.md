#### SynchronousQueue

特性
```
SynchronousQueue没有容量。

与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue。每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。

因为没有容量，所以对应 peek, contains, clear, isEmpty ... 等方法其实是无效的。例如clear是不执行任何操作的，contains始终返回false,peek始终返回null。

SynchronousQueue分为公平和非公平，默认情况下采用非公平性访问策略，当然也可以通过构造函数来设置为公平性访问策略（为true即可）。

若使用 TransferQueue, 则队列中永远会存在一个 dummy node
```

SynchronousQueue非常适合做交换工作，生产者的线程和消费者的线程同步以传递某些信息、事件或者任务。

SynchronousQueue提供了两个构造函数：
```
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable

public SynchronousQueue() {
    this(false);
}

public SynchronousQueue(boolean fair) {
    // 通过 fair 值来决定公平性和非公平性
    // 公平性使用TransferQueue，非公平性采用TransferStack
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```
TransferQueue、TransferStack继承Transferer，Transferer为SynchronousQueue的内部类，它提供了一个方法transfer()，该方法定义了转移数据的规范，如下：
```
    abstract static class Transferer<E> {
        abstract E transfer(E e, boolean timed, long nanos);
    }
```
transfer()方法主要用来完成转移数据的，如果e != null，相当于将一个数据交给消费者，如果e == null，则相当于从一个生产者接收一个消费者交出的数据。

 SynchronousQueue采用队列TransferQueue来实现公平性策略，采用堆栈TransferStack来实现非公平性策略，他们两种都是通过链表实现的，其节点分别为QNode，SNode。

 TransferQueue和TransferStack在SynchronousQueue中扮演着非常重要的作用，SynchronousQueue的put、take操作都是委托这两个类来实现的。

 由于SynchronousQueue的put、take操作都是调用Transfer的transfer()方法，只不过是传递的参数不同而已，put传递的是e参数，所以模式为数据（公平isData = true，非公平mode= DATA），而take操作传递的是null，所以模式为请求（公平isData = false，非公平mode = REQUEST）
 ```
 // put操作
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        if (transferer.transfer(e, false, 0) == null) {
            Thread.interrupted();
            throw new InterruptedException();
        }
    }

    // take操作
    public E take() throws InterruptedException {
        E e = transferer.transfer(null, false, 0);
        if (e != null)
            return e;
        Thread.interrupted();
        throw new InterruptedException();
    }
 ```

 示例：
 ```
 import java.util.Random;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<Integer> queue = new SynchronousQueue<Integer>();
        new Customer(queue).start();
        new Product(queue).start();
    }

    static class Product extends Thread{
        SynchronousQueue<Integer> queue;
        public Product(SynchronousQueue<Integer> queue){
            this.queue = queue;
        }
        @Override
        public void run(){
            while(true){
                int rand = new Random().nextInt(1000);
                System.out.println("生产了一个产品："+rand);
                System.out.println("等待三秒后运送出去...");
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                queue.offer(rand);
                System.out.println("产品生成完成："+rand);
            }
        }
    }
    static class Customer extends Thread{
        SynchronousQueue<Integer> queue;
        public Customer(SynchronousQueue<Integer> queue){
            this.queue = queue;
        }
        @Override
        public void run(){
            while(true){
                try {
                    System.out.println("消费了一个产品:"+queue.take());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("------------------------------------------");
            }
        }
    }
}
生产了一个产品：326
等待三秒后运送出去...
产品生成完成：326
生产了一个产品：291
等待三秒后运送出去...
消费了一个产品:326
------------------------------------------
产品生成完成：291
消费了一个产品:291
------------------------------------------
生产了一个产品：913
等待三秒后运送出去...
产品生成完成：913
消费了一个产品:913
------------------------------------------
生产了一个产品：993
等待三秒后运送出去...
产品生成完成：993
消费了一个产品:993
------------------------------------------
生产了一个产品：295
等待三秒后运送出去...
产品生成完成：295
消费了一个产品:295
------------------------------------------
生产了一个产品：772
等待三秒后运送出去...
产品生成完成：772
消费了一个产品:772
------------------------------------------
生产了一个产品：977
等待三秒后运送出去...
产品生成完成：977
消费了一个产品:977
------------------------------------------
生产了一个产品：182
等待三秒后运送出去...
产品生成完成：182
消费了一个产品:182
------------------------------------------
生产了一个产品：606
等待三秒后运送出去...
产品生成完成：606
消费了一个产品:606
------------------------------------------
生产了一个产品：704
等待三秒后运送出去...
产品生成完成：704
消费了一个产品:704
------------------------------------------
生产了一个产品：194
等待三秒后运送出去...
产品生成完成：194
生产了一个产品：355
等待三秒后运送出去...
消费了一个产品:194
------------------------------------------
产品生成完成：355
消费了一个产品:355
------------------------------------------
生产了一个产品：991
等待三秒后运送出去...
产品生成完成：991
消费了一个产品:991
------------------------------------------
生产了一个产品：958
等待三秒后运送出去...
产品生成完成：958
消费了一个产品:958
------------------------------------------
生产了一个产品：388
等待三秒后运送出去..
```
从结果中可以看出如果已经生产但是还未消费的，那么会阻塞在生产一直等到消费才能生成下一个。

多线程版：
```
import java.util.Random;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        SynchronousQueue<String> queue=new SynchronousQueue();
        // TODO Auto-generated method stub
        for(int i=0;i<5;i++)
            new Thread(new ThreadProducer(queue)).start();
        for(int i=0;i<5;i++)
            new Thread(new ThreadConsumer(queue)).start();
    }
}
class ThreadProducer implements Runnable {
    ThreadProducer(SynchronousQueue<String> queue)
    {
        this.queue=queue;
    }
    SynchronousQueue<String> queue;
    static int cnt=0;
    public void run()
    {
        String name="";
        int val=0;
        Random random =new Random(System.currentTimeMillis());
        for(int i=0;i<2;i++){

            cnt=(cnt+1)&0xFFFFFFFF;

            try{
                val=random.nextInt()%15;
                if(val<5)
                {
                    name="offer name:"+cnt;
                    queue.offer(name);
                }
                else if(val<10)
                {
                    name="put name:"+cnt;
                    queue.put(name);
                }
                else
                {
                    name="offer wait time and name:"+cnt;
                    queue.offer(name, 1000, TimeUnit.MILLISECONDS);
                }
                Thread.sleep(1);
            }catch(InterruptedException e)
            {
                e.printStackTrace();
            }
        }
    }
}

class ThreadConsumer implements Runnable {
    ThreadConsumer(SynchronousQueue<String> queue) {
        this.queue = queue;
    }
    SynchronousQueue<String> queue;

    public void run() {
        String name;
         for(int i=0;i<2;i++){
            try {
                name = queue.take();
                System.out.println("take " + name);
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
