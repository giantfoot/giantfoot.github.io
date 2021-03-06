# JAVA 查漏补缺


> Java  **概述**

Jva最重要的两个特征：
- 书写一次，导出运行，强大的 **跨平台** 能力。

- **自动垃圾回收**，通过垃圾收集器自动回收内存，程序员不用过分关心内存分配和回收。

JRE（Java Runtime Environment）或者 JDK（Java Development Kit）。

- JRE:也就是 Java **运行环境**，包含了 JVM 和 Java 类库，以及一些模块等。

- JDK 可以看作是 JRE 的一个超集，提供了更多工具，比如编译器、各种诊断工具jmap,jstack,jconsole等。

对于“Java 是解释执行”这句话，这个说法不太准确。应该说是半解释半编译，
- 我们开发的 Java 的源代码，首先通过 Javac 编译成为字节码（bytecode），然后，在运行时，通过 Java 虚拟机（JVM）内嵌的**解释器**将字节码转换成为最终的机器码。

- 但是常见的 JVM，比如我们大多数情况使用的 Oracle JDK 提供的 Hotspot JVM，都提供了 JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行了。


**在Java中解释执行和编译执行的区别**

解释执行：将编译好的字节码一行一行地翻译为机器码执行。

编译执行：以方法为单位，将字节码一次性翻译为机器码后执行。

**前者的优势在于不用等待，后者则在实际运行当中效率更高。**

为了满足不同的场景，HotSpot虚拟机内置了多个即时编译器：C1,C2（两个即时编译器）与Graal。Graal 是Java10正式引入的实验性即时编译器，在此暂不叙述。

- C1：即**Client编译器**，面向对启动性能有要求的客户端GUI程序，采用的优化手段比较简单，因此编译的时间较短。

- C2：即**Server编译器**，面向对性能峰值有要求的服务端程序，采用的优化手段复杂，因此编译时间长，但是在运行过程中性能更好。


从Java7开始，HotSpot虚拟机默认采用 **分层编译** 的方式：

- 热点方法首先被C1编译器编译，而后 热点方法中的热点再进一步被C2编译（理解为二次编译，根据前面的运行计算出更优的编译优化）。

- 为了不干扰程序的正常运行，JIT编译时放在额外的线程中执行的，HotSpot根据实际CPU的资源，以 1:2的比例分配给C1和C2线程数。

- 在计算机资源充足的情况，**字节码的解释运行和编译运行时可以同时进行**，**编译执行完后的机器码会在下次调用该方法时启动，已替换原本的解释执行**（意思就是已经翻译出效率更高的机器码，自然替换原来的相对低效率执行的方法）。


**Java 分为编译期和运行时。**

这里说的 Java 的编译和 C/C++ 是有着不同的意义的，Javac 的编译，编译 Java 源码生成“.class”文件里面实际是字节码，而不是可以直接执行的机器码。Java 通过字节码和 Java 虚拟机（JVM）这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“一次编译，到处执行”的基础。
**多了一个步骤，由JVM把字节码翻译成机器码，比如windows,linux，这也是JAVA能够跨平台的关键**



在运行时，JVM 会通过类加载器（Class-Loader）加载字节码，解释或者编译执行。就像我前面提到的，主流 Java 版本中，如 JDK 8 实际是解释和编译混合的一种模式，即所谓的混合模式（-Xmixed）。通常运行在 server 模式的 JVM，**会进行上万次调用以收集足够的信息进行高效的编译**，client 模式这个门限是 1500 次。Oracle Hotspot JVM 内置了两个不同的 JIT compiler，C1 对应前面说的 client 模式，适用于对于启动速度敏感的应用，比如普通 Java 桌面应用；C2 对应 server 模式，它的优化是为长时间运行的服务器端应用设计的。默认是采用所谓的分层编译（TieredCompilation）。

除了我们日常最常见的 Java 使用模式，其实还有一种新的编译方式，即所谓的 **AOT（Ahead-of-Time Compilation），直接将字节码编译成机器代码**，这样就避免了 JIT 预热等各方面的开销，比如 Oracle JDK 9 就引入了实验性的 AOT 特性，并且增加了新的 jaotc 工具。利用下面的命令把某个类或者某个模块编译成为 AOT 库。**Oracle JDK 支持分层编译和 AOT 协作使用，这两者并不是二选一的关系。**


**广义概念**：

解释执行和编译执行的区别？

计算机并不能直接地接受和执行用高级语言编写的源程序，源程序在输入计算机时，通过"翻译程序"翻译成机器语言形式的目标程序，计算机才能识别和执行。这种"翻译"通常有两种方式，即编译方式和解释方式。

编译方式是指利用事先编好的一个称为编译程序的机器语言程序，作为系统软件存放在计算机内，当用户将高级语言编写的源程序输入计算机后，编译程序便把源程序整个地翻译成用机器语言表示的与之等价的目标程序，然后计算机再执行该目标程序，以完成源程序要处理的运算并取得结果。

解释方式是指源程序进入计算机后，解释程序边扫描边解释，逐句输入逐句翻译，计算机一句句执行，并不产生目标程序。


**类比一下，一个是同声传译，一个是放录音**
