#### Enumeration 解析

> Iterator和Enumeration的两者不同

- Iterator有对容器进行修改的方法。而Enumeration只能遍历。

- Iterator支持fail-fast，而Enumeration不支持。
- Iterator比Enumeration覆盖范围广，基本所有容器中都有Iterator迭代器，**而只有Vector、Hashtable有Enumeration**。
- Enumeration在JDK 1.0就已经存在了，而Iterator是JDK2.0新加的接口。

```
import java.util.Enumeration;
  import java.util.Vector;

  public class ItrOfVectorTest {

          @org.junit.Test
          public void test() {
              Vector list = new Vector<>();
              list.add("1");
              list.add("2");
              list.add("3");
              list.add("4");
              Enumeration enu = list.elements();  
              while (enu.hasMoreElements()) {  
                  System.out.println(enu.nextElement());
              }
          }
  }
```
