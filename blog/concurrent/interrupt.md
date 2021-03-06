#### interrupt 线程中断

线程只有在循环调用某段代码才会用while。外部如果达到一定条件，就会调用interrupt方法或设置flag标记为false让线程跳出while循环。

**这才是中断的用处所在。**

如果线程只是顺序执行一段代码，等于是昙花一现，根本就没必要中断（但是try catch必须要，防止sleep等方法执行时中断）。


interrupt()的作用是中断本线程。

本线程中断自己是被允许的；其它线程调用本线程的interrupt()方法时，会通过checkAccess()检查权限。这有可能抛出SecurityException异常。

如果本线程是处于阻塞状态：调用线程的wait(), wait(long)或wait(long, int)会让它进入等待(阻塞)状态，或者调用线程的join(), join(long), join(long, int), sleep(long), sleep(long, int)也会让它进入阻塞状态。若线程在阻塞状态时，调用了它的interrupt()方法，那么它的“中断状态”会被清除并且会收到一个InterruptedException异常。

例如，线程通过wait()进入阻塞状态，此时通过interrupt()中断该线程；调用interrupt()会立即将线程的中断标记设为“true”，但是由于线程处于阻塞状态，所以该“中断标记”会立即被清除为“false”，同时，会产生一个InterruptedException的异常。

如果线程被阻塞在一个Selector选择器(NIO)中，那么通过interrupt()中断它时；线程的中断标记会被设置为true，并且它会立即从选择操作中返回。

如果不属于前面所说的情况，那么通过interrupt()中断线程时，它的中断标记会被设置为“true”。
中断一个“已终止的线程”不会产生任何操作。

> Thread中的stop()和suspend()方法，由于固有的不安全性，已经建议不再使用！

**终止处于“阻塞状态”的线程**

通常，**我们通过“中断”方式终止处于“阻塞状态”的线程。**

当线程由于被调用了sleep(), wait(), join()等方法而进入阻塞状态；若此时调用线程的interrupt()将线程的中断标记设为true。由于处于阻塞状态，中断标记会被清除，同时产生一个InterruptedException异常。将InterruptedException放在适当的为止就能终止线程，形式如下：
```
@Override
public void run() {
    try {
        while (true) {
            // 执行任务...
        }
    } catch (InterruptedException ie) {  
        // 由于产生InterruptedException异常，退出while(true)循环，线程终止！
    }
}
```
在while(true)中不断的执行任务，当线程处于阻塞状态时，调用线程的interrupt()产生InterruptedException中断。中断的捕获在while(true)之外，这样就退出了while(true)循环！

注意：对InterruptedException的捕获务一般放在while(true)循环体的外面，这样，在产生异常时就退出了while(true)循环。否则，InterruptedException在while(true)循环体之内，就需要额外的添加退出处理。

**终止处于“运行状态”的线程**

(01) 通过“中断标记”终止线程。
```
@Override
public void run() {
    while (!isInterrupted()) {
        // 执行任务...
    }
}
```

(02) 通过“额外添加标记”。
```
private volatile boolean flag= true;
protected void stopTask() {
    flag = false;
}

@Override
public void run() {
    while (flag) {
        // 执行任务...
    }
}   
```


通用的终止线程的形式如下：
```
@Override
public void run() {
    try {
        // 1. isInterrupted()保证，只要中断标记为true就终止线程。
        while (!isInterrupted()) {
            // 执行任务...
        }
    } catch (InterruptedException ie) {  
        // 2. InterruptedException异常保证，当InterruptedException异常产生时，线程被终止。
    }
}
```

interrupted() 和 isInterrupted()都能够用于检测对象的“中断标记”。

区别是，interrupted()除了返回中断标记之外，它还会清除中断标记(即将中断标记设为false)；而isInterrupted()仅仅返回中断标记。
