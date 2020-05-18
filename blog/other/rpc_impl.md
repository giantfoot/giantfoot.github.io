# 实现简单地RPC框架

说起RPC，就不能不提到分布式，这个促使RPC诞生的领域。

假设你有一个计算器接口，Calculator，以及它的实现类CalculatorImpl，那么在系统还是单体应用时，你要调用Calculator的add方法来执行一个加运算，直接new一个CalculatorImpl，然后调用add方法就行了，这其实就是非常普通的本地函数调用，因为在同一个地址空间，或者说在同一块内存，所以通过方法栈和参数栈就可以实现。

现在，基于高性能和高可靠等因素的考虑，你决定将系统改造为分布式应用，将很多可以共享的功能都单独拎出来，比如上面说到的计算器，你单独把它放到一个服务里头，让别的服务去调用它。

这下问题来了，服务A里头并没有CalculatorImpl这个类，那它要怎样调用服务B的CalculatorImpl的add方法呢？

有同学会说，可以模仿B/S架构的调用方式呀，在B服务暴露一个Restful接口，然后A服务通过调用这个Restful接口来间接调用CalculatorImpl的add方法。

很好，这已经很接近RPC了，不过如果是这样，那每次调用时，是不是都需要写一串发起http请求的代码呢？比如httpClient.sendRequest...之类的，能不能像本地调用一样，去发起远程调用，**让使用者感知不到远程调用的过程呢？**


这时候，有同学就会说，用代理模式呀！而且最好是结合Spring IoC一起使用，通过Spring注入calculator对象，注入时，如果扫描到对象加了@Reference注解，那么就给它生成一个代理对象，将这个代理对象放进容器中。**而这个代理对象的内部，就是通过httpClient来实现RPC远程过程调用的。**

可能上面这段描述比较抽象，不过这就是很多RPC框架要解决的问题和解决的思路，比如阿里的Dubbo。

总结一下，RPC要解决的两个问题：

1. 解决分布式系统中，服务之间的调用问题。
2. 远程调用时，要能够像本地调用一样方便，让调用者感知不到远程调用的逻辑。


如何实现一个RPC
实际情况下，**RPC很少用到http协议来进行数据传输，毕竟我只是想传输一下数据而已，何必动用到一个文本传输的应用层协议呢，我为什么不直接使用二进制传输？** 比如直接用Java的Socket协议进行传输？

实现思路：

假设有两个服务A和B，A要调用B的某个方法

1. Service A的应用层代码中，调用了Calculator的一个实现类的add方法，希望执行一个加法运算；

2. 这个Calculator实现类，内部并不是直接实现计算器的加减乘除逻辑，而是通过远程调用Service B的RPC接口，来获取运算结果，因此称之为Stub；
3. Stub怎么和Service B建立远程通讯呢？这时候就要用到远程通讯工具了，也就是图中的Run-time Library，这个工具将帮你实现远程通讯的功能，比如Java的Socket，就是这样一个库，当然，你也可以用基于Http协议的HttpClient，或者其他通讯工具类，都可以，RPC并没有规定说你要用何种协议进行通讯；
4. Stub通过调用通讯工具提供的方法，和Service B建立起了通讯，然后将请求数据发给Service B。需要注意的是，由于底层的网络通讯是基于二进制格式的，因此这里Stub传给通讯工具类的数据也必须是二进制，比如calculator.add(1,2)，你必须把参数值1和2放到一个Request对象里头（这个Request对象当然不只这些信息，还包括要调用哪个服务的哪个RPC接口等其他信息），然后序列化为二进制，再传给通讯工具类，这一点也将在下面的代码实现中体现；
5. 二进制的数据传到Service B这一边了，Service B当然也有自己的通讯工具，通过这个通讯工具接收二进制的请求；
6. 既然数据是二进制的，那么自然要进行反序列化了，将二进制的数据反序列化为请求对象，然后将这个请求对象交给Service B的Stub处理；
7. 和之前的Service A的Stub一样，这里的Stub也同样是个“假玩意”，它所负责的，只是去解析请求对象，知道调用方要调的是哪个RPC接口，传进来的参数又是什么，**然后再把这些参数传给对应的RPC接口，也就是Calculator的实际实现类去执行。很明显，如果是Java，那这里肯定用到了反射。**
8. RPC接口执行完毕，返回执行结果，现在轮到Service B要把数据发给Service A了，怎么发？一样的道理，一样的流程，只是现在Service B变成了Client，Service A变成了Server而已：Service B反序列化执行结果->传输给Service A->Service A反序列化执行结果 -> 将结果返回给Application，完毕。


具体实现：

**服务A有服务B的接口包，但没有具体实现，需要自己实现调用的接口，在实现类里面远程调用服务B的接口**

服务A调用
```
public class ComsumerApp {
    public static void main(String[] args) {
        Calculator calculator = new CalculatorRemoteImpl();
        int result = calculator.add(1, 2);
    }
}
```

通过一个CalculatorRemoteImpl，我们把RPC的逻辑封装进去了，客户端调用时感知不到远程调用的麻烦。下面再来看看CalculatorRemoteImpl，代码有些多，但是其实就是把上面的2、3、4几个步骤用代码实现了而已，CalculatorRemoteImpl：

```
public class CalculatorRemoteImpl implements Calculator {
    public int add(int a, int b) {
        List<String> addressList = lookupProviders("Calculator.add");
        String address = chooseTarget(addressList);
        try {
            Socket socket = new Socket(address, PORT);

            // 将请求序列化
            CalculateRpcRequest calculateRpcRequest = generateRequest(a, b);
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());

            // 将请求发给服务提供方
            objectOutputStream.writeObject(calculateRpcRequest);

            // 将响应体反序列化
            ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
            Object response = objectInputStream.readObject();

            if (response instanceof Integer) {
                return (Integer) response;
            } else {
                throw new InternalError();
            }

        } catch (Exception e) {
            log.error("fail", e);
            throw new InternalError();
        }
    }
}
```

服务端B：
```
public class ProviderApp {
    private Calculator calculator = new CalculatorImpl();

    public static void main(String[] args) throws IOException {
        new ProviderApp().run();
    }

    private void run() throws IOException {
        ServerSocket listener = new ServerSocket(9090);
        try {
            while (true) {
                Socket socket = listener.accept();
                try {
                    // 将请求反序列化
                    ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
                    Object object = objectInputStream.readObject();

                    log.info("request is {}", object);

                    // 调用服务
                    int result = 0;
                    if (object instanceof CalculateRpcRequest) {
                        CalculateRpcRequest calculateRpcRequest = (CalculateRpcRequest) object;
                        if ("add".equals(calculateRpcRequest.getMethod())) {
                            result = calculator.add(calculateRpcRequest.getA(), calculateRpcRequest.getB());
                        } else {
                            throw new UnsupportedOperationException();
                        }
                    }

                    // 返回结果
                    ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
                    objectOutputStream.writeObject(new Integer(result));
                } catch (Exception e) {
                    log.error("fail", e);
                } finally {
                    socket.close();
                }
            }
        } finally {
            listener.close();
        }
    }

}
```
