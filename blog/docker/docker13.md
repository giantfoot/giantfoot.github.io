#### Docker网络

- 每启动一个docker容器，docker就会给容器分配一个ip，只要安装了docker，就会有一个网卡docker0桥接模式，使用的技术是evth-pair技术，启动了容器也会给容器分配一个网卡，这个网卡跟容器内部的网卡是同一对。

容器内部网卡和宿主机外部分配的网卡是成对出现的，veth-pair就是一对虚拟设备接口，成对出现，一端连接着协议，一端彼此相连，正是因为这个特性，veth-pair充当了一个桥梁，都连接在docker网卡，所以两个容器可以相互ping通。

![docker](../pic/docker/docker8.png)

![docker](../pic/docker/docker9.png)

Docker 使用的是Linux的桥接，宿主机中是一个Docker容器的网桥docker0
