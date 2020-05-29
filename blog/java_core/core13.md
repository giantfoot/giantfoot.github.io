# JAVA 查漏补缺

> 设计模式

设计模式是人们为软件开发中相同表征的问题，抽象出的可重复利用的解决方案。在某种程度上，设计模式已经代表了一些特定情况的最佳实践，同时也起到了软件工程师之间沟通的“行话”的作用。理解和掌握典型的设计模式，有利于我们提高沟通、设计的效率和质量。

设计模式可以分为创建型模式、结构型模式和行为型模式。

- 创建型模式，是对对象创建过程的各种问题和解决方案的总结，包括各种工厂模式（Factory、Abstract Factory）、单例模式（Singleton）、构建器模式（Builder）、原型模式（ProtoType）。

- 结构型模式，是针对软件设计结构的总结，关注于类、对象继承、组合方式的实践经验。常见的结构型模式，包括桥接模式（Bridge）、适配器模式（Adapter）、装饰者模式（Decorator）、代理模式（Proxy）、组合模式（Composite）、外观模式（Facade）、享元模式（Flyweight）等。

- 行为型模式，是从类或对象之间交互、职责划分等角度总结的模式。比较常见的行为型模式有策略模式（Strategy）、解释器模式（Interpreter）、命令模式（Command）、观察者模式（Observer）、迭代器模式（Iterator）、模板方法模式（Template Method）、访问者模式（Visitor）。

---

> InputStream 是一个抽象类，标准类库中提供了 FileInputStream、ByteArrayInputStream 等各种不同的子类，分别从不同角度对 InputStream 进行了功能扩展，这是典型的装饰器模式应用案例。

识别**装饰器模式**，可以通过识别类设计特征来进行判断，也就是其类构造函数以相同的抽象类或者接口为输入参数。

例如，BufferedInputStream 经过包装，为输入流过程增加缓存，类似这种装饰器还可以多次嵌套，不断地增加不同层次的功能。
```
public BufferedInputStream(InputStream in)
```

因为装饰器模式本质上是包装同类型实例，我们对目标对象的调用，往往会通过包装类覆盖过的方法，迂回调用被包装的实例，这就可以很自然地实现增加额外逻辑的目的，也就是所谓的“装饰”。

---

创建型模式尤其是**工厂模式**，在我们的代码中随处可见，我举个相对不同的 API 设计实践。比如，JDK 最新版本中 HTTP/2 Client API，下面这个创建 HttpRequest 的过程，就是典型的构建器模式（Builder），通常会被实现成fluent 风格的 API，也有人叫它方法链。

```

HttpRequest request = HttpRequest.newBuilder(new URI(uri))
                     .header(headerAlice, valueAlice)
                     .headers(headerBob, value1Bob,
                      headerCarl, valueCarl,
                      headerBob, value2Bob)
                     .GET()
                     .build();
```
使用构建器模式，可以比较优雅地解决构建复杂对象的麻烦，这里的“复杂”是指类似需要输入的参数组合较多，如果用构造函数，我们往往需要为每一种可能的输入参数组合实现相应的构造函数，一系列复杂的构造函数会让代码阅读性和可维护性变得很差。

---
单例模式

版本一：
```

 public class Singleton {
       private static Singleton instance = new Singleton();
       public static Singleton getInstance() {
          return instance;
       }
    }
```
问题：

**Java 会自动为没有明确声明构造函数的类，定义一个 public 的无参数的构造函数**，所以上面的例子并不能保证额外的对象不被创建出来，别人完全可以直接“new Singleton()”，那我们应该怎么处理呢？

不错，可以为单例定义一个 private 的构造函数（也有建议声明为枚举，这是有争议的，我个人不建议选择相对复杂的枚举，毕竟日常开发不是学术研究）。

这样还有什么改进的余地吗？

添加懒加载

版本二：
```

public class Singleton {
        private static Singleton instance;
        private Singleton() {
        }
        public static Singleton getInstance() {
            if (instance == null) {
            instance = new Singleton();
            }
        return instance;
        }
    }
```
这个实现在单线程环境不存在问题，**但是如果处于并发场景，就需要考虑线程安全，最熟悉的就莫过于“双检锁”**

版本三：
```

public class Singleton {
  private static volatile Singleton singleton = null;
  private Singleton() {
  }

  public static Singleton getSingleton() {
      if (singleton == null) { // 尽量避免重复进入同步块
          synchronized (Singleton.class) { // 同步.class，意味着对同步类方法调用
              if (singleton == null) {
                  singleton = new Singleton();
              }
          }
      }
      return singleton;
  }
}

```

- 这里的 volatile 能够提供可见性，以及保证 getInstance 返回的是初始化完全的对象。

- 在同步之前进行 null 检查，以尽量避免进入相对昂贵的同步块。
- 直接在 class 级别进行同步，保证线程安全的类方法调用。
> 在这段代码中，争论较多的是 volatile 修饰静态变量，当 Singleton 类本身有多个成员变量时，需要保证初始化过程完成后，才能被 get 到。


当然，也有一些人推荐利用内部类持有静态对象的方式实现，其理论依据是对象初始化过程中隐含的初始化锁（有兴趣的话你可以参考jls-12.4.2 中对 LC 的说明），这种和前面的双检锁实现都能保证线程安全，不过语法稍显晦涩，未必有特别的优势。

版本四：
```

public class Singleton {
  private Singleton(){}
  public static Singleton getSingleton(){
      return Holder.singleton;
  }

  private static class Holder {
      private static Singleton singleton = new Singleton();
  }
}
```
