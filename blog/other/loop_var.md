# JAVA核心技术总结


> Java  **循环里的变量**

放在循环内部定义，确实会多次申请栈帧的内存空间（java中一个线程对应一个Java栈，每次方法调用会向栈中压入一个新帧，帧中存储着参数、局部变量、中间运算结果等等数据，方法调用结束后当前的帧就被弹出了。

参考文章：http://boy00fly.iteye.com/blog/1096637）

为什么会多占用栈空间内？

原因很简单，每次循环都要重复定义局部变量，而如果放在循环体外，每次只要移动引用指向的堆内存地址即可，不必重新申请内存空间了。
其实话说回来，这样对性能的影响其实可以忽略不计，没什么大的问题，


放在里面，虽然会在栈内每次都分配一块内存，但是由于栈内存的分配非常之快，仅次于寄存器，所以，可以忽略不计。


循环内的话，每次循环内部的局部变量在每次进for循环的时候都要重新定义一遍变量，也就是执行申请内存空间，变量压入堆栈的过程。
循环外定义的话，for循环一直用的是同一块内存空间，效率比较高，但是变量的作用域就大了，耗内存

其实内外都可以的，总之就是空间和时间的权衡，看实际情况了，局部变量的数据类型、大小什么的都有关
