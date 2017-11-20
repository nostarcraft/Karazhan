环境：­

#### VMware workstation 5.5.2­

#### Guest OS:RHEL5.1­


所有的网卡都走bridge,其中1台RHEL只开eth0,另1台开eth0,eth1,并做bonding­


bonding过程如下：­

修改/etc/sysconfig/network-scripts/下的eth0和eth1­

```
vi ifcfg-eth0­

DEVICE=eth0­

BOOTPROTO=none­

#HWADDR=00:0C:29:BB:57:4A­

ONBOOT=yes­

#NETMASK=255.255.255.0­

#IPADDR=192.168.2.160­

TYPE=Ethernet­

USERCTL=no­

IPV6INIT=no­

PEERDNS=no­

MASTER=bond0­

SLAVE=yes­
```
```
vi ifcfg-eth1­

DEVICE=eth1­

BOOTPROTO=none­

#HWADDR=00:0C:29:BB:57:54­

ONBOOT=yes­

#NETMASK=255.255.255.0­

#IPADDR=192.168.2.161­

TYPE=Ethernet­

USERCTL=no­

IPV6INIT=no­

PEERDNS=no­

MASTER=bond0­

SLAVE=yes­
```

添加bond0­
```
vi ifcfg-bond0­

DEVICE=bond0­

BOOTPROTO=none­

#HWADDR=00:0C:29:BB:57:4A­

ONBOOT=yes­

NETMASK=255.255.255.0­

IPADDR=192.168.2.165­

#TYPE=Ethernet­

USERCTL=no­

IPV6INIT=no­

#FREEDNS=yes­
```

在/etc/modprobe.conf里加入两行­
```
alias bond0 bonding­

options bond0 miimon=100 mode=0­
```

#其中miimon为2个网口互相检测的时间，mode=0为round robin，如果改成1那就是active-backup­

#如果想修改里面的值但不重起系统，可以用modprobe -r bonding,然后modprobe -v bonding来重载这个模块­


修改/etc/rc.d/rc.local,加入一行­
```
/sbin/ifenslave bond0 eth0 eth1­
```

到此，bonding做完，可以用service network status和cat /proc/net/bonding/bond0来查看其状态­


>测试一：­

>在此系统上与另1台RHEL(192.168.2.111)互ping­

>1.如果，只ifconfig eth1 down,双方都能ping通­

>2.如果ifconfig eth0 down而保持eth1 up，双方都ping不通­


通过watch -n 1 "/sbin/ifconfig |grep bytes"查看原因，发现eth1的RX并没有收包­


>测试二：­

>把第二台RHEL也配成bonding(mode=0),两台互ping,发现2个系统都是eth1的RX没有收包­


猜测：可能是我宿主系统上只有1个网卡，所以才造成了这种情况­


为了证实这一点，开第三台RHEL,同样eth0(192.168.2.140)eth1(192.168.2.141),都做bridge­

发现不管是ping 140还是141,或同时ping,都能通,但通过watch发现仅仅是eth0在收发包，而eth1的RX和TX基本没动静­

然后所有的虚拟网卡改走NAT，情况依旧­

2008.11
