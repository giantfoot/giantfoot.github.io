### Hadoop学习笔记二

**网络配置二**

vi   /etc/sysconfig/network-scripts/ifcfg-ens33

```

TYPE="Ethernet"   # 网络类型为以太网
BOOTPROTO="static"  # 手动分配ip
NAME="ens33"  # 网卡设备名，设备名一定要跟文件名一致
DEVICE="ens33"  # 网卡设备名，设备名一定要跟文件名一致
ONBOOT="yes"  # 该网卡是否随网络服务启动
IPADDR="192.168.220.101"  # 该网卡ip地址就是你要配置的固定IP，如果你要用xshell等工具连接，220这个网段最好和你自己的电脑网段一致，否则有可能用xshell连接失败
GATEWAY="192.168.220.2"   # 网关
NETMASK="255.255.255.0"   # 子网掩码
DNS1="8.8.8.8"    # DNS，8.8.8.8为Google提供的免费DNS服务器的IP地址
```
/etc/sysconfig/network文件里增加如下配置
```

NETWORKING=yes # 网络是否工作，此处一定不能为no
```

配置公共DNS服务(可选)

在/etc/resolv.conf文件里增加如下配置
```
nameserver 8.8.8.8
```

关闭防火墙
```
systemctl status firewalld #检查防火墙状态
systemctl stop firewalld # 临时关闭防火墙
systemctl disable firewalld # 禁止开机启动
```
重启网络服务
```
service network restart
```

```
192.168.80.100 hadoop100
192.168.80.101 hadoop101
192.168.80.102 hadoop102
192.168.80.103 hadoop103
192.168.80.104 hadoop104
192.168.80.105 hadoop105
192.168.80.106 hadoop106
192.168.80.107 hadoop107
192.168.80.108 hadoop108
```

```
IPADDR=192.168.80.100
GATEWAY=192.168.80.2
DNS1=192.168.80.2

```


Nat网络概述
![Nat网络概述](../pic/hadoop/nat网络.png)
nat网关
![nat网关](../pic/hadoop/nat网关.png)
IP文件配置
![IP文件配置](../pic/hadoop/ip文件配置.png)
IP文件配置说明
![IP文件配置说明](../pic/hadoop/IP文件配置说明.png)
设置ip重启服务
![设置ip重启服务](../pic/hadoop/设置ip重启服务.png)
修改主机名
![修改主机名](../pic/hadoop/修改主机名.png)
