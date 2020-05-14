# 深度COPY

任何变成语言中，其实都有浅拷贝和深拷贝的概念，Java 中也不例外。

首先需要明白，浅拷贝和深拷贝都是针对一个已有对象的操作。那先来看看浅拷贝和深拷贝的概念。

在 Java 中，除了基本数据类型（元类型）之外，还存在 类的实例对象 这个引用数据类型。而一般使用 『 = 』号做赋值操作的时候。对于基本数据类型，实际上是拷贝的它的值，但是对于对象而言，其实赋值的只是这个对象的引用，将原对象的引用传递过去，他们实际上还是指向的同一个对象。

而浅拷贝和深拷贝就是在这个基础之上做的区分，如果在拷贝这个对象的时候，只对基本数据类型进行了拷贝，**而对引用数据类型只是进行了引用的传递**，而没有真实的创建一个新的对象，则认为是浅拷贝。反之，**在对引用数据类型进行拷贝的时候，创建了一个新的对象，并且复制其内的成员变量，则认为是深拷贝**。

所以到现在，就应该了解了，所谓的浅拷贝和深拷贝，只是在拷贝对象的时候，对 类的实例对象 这种引用数据类型的不同操作而已。


在 Java 中，所有的 Class 都继承自 Object ，而在 Object 上，存在一个 clone() 方法，它被声明为了 protected ，所以我们可以在其子类中，使用它。

而无论是浅拷贝还是深拷贝，都需要实现 clone() 方法，来完成操作。

它的实现非常的简单，它限制所有调用 clone() 方法的对象，都必须实现 Cloneable 接口,对此我们就没必要深究了，只需要知道它可以 clone() 一个对象得到一个新的对象实例即可。而反观 **Cloneable 接口**，可以看到它其实什么方法都不需要实现。对他可以简单的理解只是一个标记，是开发者允许这个对象被拷贝。


clone() 方法实则是真的创建了一个新的对象。

**但这只是一次浅拷贝的操作**,因为新对象里面的成员变量还是指向老对象，除非让成员变量的类也实现Cloneable 接口并重写clone方法。

最近需要用到比较两个对象属性的变化，其中一个是oldObj,另外一个是newObj，oldObj是newObj的前一个状态，所以需要在newObj的某个状态时，复制一个一样的对象，由于JAVA不支持深层拷贝，因此专门写了一个方法。

总结：

> 如果一个对象内部只有基本数据类型，那用 clone() 方法获取到的就是这个对象的深拷贝，而如果其内部还有引用数据类型，那用 clone() 方法就是一次浅拷贝的操作。

方法实现很简单，提供三种方式：

一种是序列化成数据流，前提是所有对象（对象中包含的对象...）都需要继承Serializable接口，如果都继承了那很容易，如果没有继承，而且也不打算修改所有类，可以用第二种方式。

第二种是将对象序列化为json，通过json来实现拷贝，这种方式需要用到net.sf.json.JSONObject。

第三种是重写所有子对象的clone方法。

```
import net.sf.json.JSONObject;

import java.io.*;

class deepCopy {
    /**
     * 深层拷贝
     *
     * @param <T>
     * @param obj
     * @return
     * @throws Exception
     */
    static <T> T copy(T obj) throws Exception {
        //是否实现了序列化接口，即使该类实现了，他拥有的对象未必也有...
        if(Serializable.class.isAssignableFrom(obj.getClass())){
            //如果子类没有继承该接口，这一步会报错
            try {
                return copyImplSerializable(obj);
            } catch (Exception e) {
                //这里不处理，会运行到下面的尝试json
            }
        }
        //如果序列化失败，尝试json序列化方式
        if(hasJson()){
            try {
                return copyByJson(obj);
            } catch (Exception e) {
                //这里不处理，下面返回null
            }
        }
        return null;
    }

    /**
     * 深层拷贝 - 需要类继承序列化接口
     * @param <T>
     * @param obj
     * @return
     * @throws Exception
     */
    @SuppressWarnings("unchecked")
    private static <T> T copyImplSerializable(T obj) throws Exception {
        ByteArrayOutputStream baos = null;
        ObjectOutputStream oos = null;

        ByteArrayInputStream bais = null;
        ObjectInputStream ois = null;

        Object o = null;
        //如果子类没有继承该接口，这一步会报错
        try {
            baos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(baos);
            oos.writeObject(obj);
            bais = new ByteArrayInputStream(baos.toByteArray());
            ois = new ObjectInputStream(bais);

            o = ois.readObject();
            return (T) o;
        } catch (Exception e) {
            throw new Exception("对象中包含没有继承序列化的对象");
        } finally{
            try {
                assert baos != null;
                baos.close();
                assert oos != null;
                oos.close();
                assert bais != null;
                bais.close();
                assert ois != null;
                ois.close();
            } catch (Exception e2) {
                //这里报错不需要处理
            }
        }
    }

    /**
     * 是否可以使用json
     * @return
     */
    private static boolean hasJson(){
        try {
            Class.forName("net.sf.json.JSONObject");
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 深层拷贝 - 需要net.sf.json.JSONObject
     * @param <T>
     * @param obj
     * @return
     * @throws Exception
     */
    @SuppressWarnings("unchecked")
    private static <T> T copyByJson(T obj) throws Exception {
        return (T) JSONObject.toBean(JSONObject.fromObject(obj),obj.getClass());
    }
}

```
