<!-- TOC -->

- [Linux网络配置 debian系统](#linux网络配置-debian系统)
    - [配置IP地址](#配置ip地址)
        - [iplink与ifconfig命令对比](#iplink与ifconfig命令对比)
    - [配置默认路由](#配置默认路由)
    - [配置VLAN](#配置vlan)
    - [配置硬件选项](#配置硬件选项)
    - [L4 相关命令](#l4-相关命令)
        - [nestat](#nestat)
        - [linux查看端口占用](#linux查看端口占用)
    - [配置dns](#配置dns)

<!-- /TOC -->


# Linux网络配置 debian系统

Linux网络配置方法简介。

## 配置IP地址

```sh
# 使用ifconfig
ifconfig eth0 192.168.1.3 netmask 255.255.255.0

# 使用用ip命令增加一个IP
ip addr add 192.168.1.4/24 dev eth0

# 使用ifconfig增加网卡别名
ifconfig eth0:0 192.168.1.10
```

这样配置的`IP`地址重启机器后会丢失，所以一般应该把网络配置写入文件中。如`Ubuntu`可以将网卡配置写入`/etc/network/interfaces`（`Redhat`和`CentOS`则需要写入` /etc/sysconfig/network-scripts/ifcfg-eth0`中）：

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 192.168.1.3
    netmask 255.255.255.0
    gateway 192.168.1.1

auto eth1
iface eth1 inet dhcp
```

### iplink与ifconfig命令对比

https://p5r.uk/blog/2010/ifconfig-ip-comparison.html
基本可以理解为：除了物理网卡的增删需要ifconfig，其他的都可以通过ip命令搞定。

## 配置默认路由

```sh
# 使用route命令
route add default gw 192.168.1.1
# 也可以使用ip命令
ip route add default via 192.168.1.1
```

## 配置VLAN

```sh
# 安装并加载内核模块
apt-get install vlan
modprobe 8021q

# 添加vlan
vconfig add eth0 100
ifconfig eth0.100 192.168.100.2 netmask 255.255.255.0

# 删除vlan
vconfig rem eth0.100
```

## 配置硬件选项

```sh
# 改变speed
ethtool -s eth0 speed 1000 duplex full

# 关闭GRO
ethtool -K eth0 gro off

# 开启网卡多队列
ethtool -L eth0 combined 4

# 开启vxlan offload
ethtool -K ens2f0 rx-checksum on
ethtool -K ens2f0 tx-udp_tnl-segmentation on

# 查询网卡统计
ethtool -S eth0
```

## L4 相关命令

### nestat

- http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html
- https://blog.csdn.net/bit_clearoff/article/details/60876695

https://zhidao.baidu.com/question/105305364.html
- local address是本地就是你本机的地址
- foreign address是外部电脑，就是和你电脑有联系的ip地址，也就是remote IP
- 两者加起来就是session
- ssh就是20端口，只是这里显示的是ssh

```
gyw@vultr:~$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 45.77.132.65.v:http-alt *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      1 45.77.132.65.vultr.:ssh 218.92.0.204:52457      FIN_WAIT1
tcp        0     41 45.77.132.65.vultr.:ssh 218.92.0.204:19508      ESTABLISHED
tcp        0    352 45.77.132.65.vultr.:ssh 114.253.99.223:58984    ESTABLISHED
tcp        0      0 45.77.132.65.vultr.:ssh 218.92.0.204:34655      SYN_RECV
tcp        0      0 45.77.132.65.vultr.:ssh 218.92.0.204:50586      SYN_RECV
tcp        0      0 45.77.132.65.vultr.:ssh 218.92.0.204:11826      SYN_RECV
tcp6       0      0 [::]:5000               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 [::]:8888               [::]:*                  LISTEN
udp        0      0 172.17.0.1:ntp          *:*
udp        0      0 45.77.132.65.vultr.:ntp *:*
udp        0      0 localhost:ntp           *:*
udp        0      0 *:ntp                   *:*
udp        0      0 *:bootpc                *:*
udp6       0      0 fe80::6ccd:f8ff:fec:ntp [::]:*
udp6       0      0 fe80::34ba:b7ff:fe8:ntp [::]:*
udp6       0      0 fe80::d060:adff:fe5:ntp [::]:*
udp6       0      0 fe80::42:50ff:fe43::ntp [::]:*
udp6       0      0 fe80::5400:1ff:fedc:ntp [::]:*
udp6       0      0 localhost:ntp           [::]:*
udp6       0      0 [::]:ntp                [::]:*
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    10347437 /run/user/1000/systemd/notify
unix  2      [ ]         DGRAM                    19830    /run/user/0/systemd/notify
unix  2      [ ACC ]     STREAM     LISTENING     10347438 /run/user/1000/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     19831    /run/user/0/systemd/private
unix  2      [ ACC ]     SEQPACKET  LISTENING     9823     /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     9773     /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     9781     /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     9782     /run/systemd/journal/stdout
unix  8      [ ]         DGRAM                    9783     /run/systemd/journal/socket
unix  13     [ ]         DGRAM                    9815     /run/systemd/journal/dev-log
unix  2      [ ACC ]     STREAM     LISTENING     9821     /run/lvm/lvmetad.socket
unix  2      [ ]         DGRAM                    9911     /run/systemd/journal/syslog
unix  2      [ ACC ]     STREAM     LISTENING     185931   /run/containerd/containerd.sock
unix  2      [ ACC ]     STREAM     LISTENING     190317   /var/run/docker/metrics.sock
unix  2      [ ACC ]     STREAM     LISTENING     190535   /run/docker/libnetwork/d7c846a428dd896469c00c31a64ff54035e9afb7a62fb3f2b72c0dfe44b6ebb9.sock
unix  2      [ ACC ]     STREAM     LISTENING     10075    /run/systemd/fsck.progress
unix  2      [ ACC ]     STREAM     LISTENING     12366    /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     12367    /run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     12368    /run/snapd.socket
unix  2      [ ACC ]     STREAM     LISTENING     12369    /run/snapd-snap.socket
unix  2      [ ACC ]     STREAM     LISTENING     12370    /run/uuidd/request
unix  2      [ ACC ]     STREAM     LISTENING     1506742  @/containerd-shim/moby/081f5a2633c96102046497b5c56c6ec2a9402a943bab2593f3d5a776072adc15/shim.sock
unix  2      [ ACC ]     STREAM     LISTENING     12363    /var/lib/lxd/unix.socket
unix  2      [ ACC ]     STREAM     LISTENING     10348906 @/containerd-shim/moby/f861143c120e5fbda684a932c1783290bcd57f7c4ac972c801fff497ecc8112b/shim.sock
unix  2      [ ACC ]     STREAM     LISTENING     14197    @ISCSIADM_ABSTRACT_NAMESPACE
unix  3      [ ]         DGRAM                    9772     /run/systemd/notify
unix  2      [ ACC ]     STREAM     LISTENING     190218   /var/run/docker.sock
unix  2      [ ACC ]     STREAM     LISTENING     2858290  @/containerd-shim/moby/01ffe3c2b61ef52d9413b80aae21d90f59ed4b30fb7f092153e9164cd4fd98f6/shim.sock
unix  3      [ ]         STREAM     CONNECTED     190331   /run/containerd/containerd.sock
unix  3      [ ]         STREAM     CONNECTED     190327
unix  2      [ ]         DGRAM                    14189
unix  2      [ ]         DGRAM                    190311
unix  3      [ ]         STREAM     CONNECTED     17296    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     13256
unix  3      [ ]         STREAM     CONNECTED     13262    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     190328   /run/containerd/containerd.sock
unix  3      [ ]         STREAM     CONNECTED     13187    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     190330
unix  2      [ ]         DGRAM                    12735
unix  2      [ ]         DGRAM                    10347315
unix  3      [ ]         STREAM     CONNECTED     13181
unix  2      [ ]         DGRAM                    19824
unix  3      [ ]         STREAM     CONNECTED     12933
unix  2      [ ]         DGRAM                    9962
unix  3      [ ]         STREAM     CONNECTED     10916    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     12989
unix  3      [ ]         STREAM     CONNECTED     10915
unix  3      [ ]         STREAM     CONNECTED     12938    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     13480    /var/run/dbus/system_bus_socket
unix  3      [ ]         STREAM     CONNECTED     1506751  @/containerd-shim/moby/081f5a2633c96102046497b5c56c6ec2a9402a943bab2593f3d5a776072adc15/shim.sock
unix  3      [ ]         STREAM     CONNECTED     13479
unix  3      [ ]         STREAM     CONNECTED     10348910
unix  2      [ ]         DGRAM                    10347406
unix  2      [ ]         DGRAM                    10347569
unix  3      [ ]         STREAM     CONNECTED     10347390
unix  3      [ ]         STREAM     CONNECTED     2858297
unix  3      [ ]         STREAM     CONNECTED     17309
unix  3      [ ]         STREAM     CONNECTED     10347489
unix  3      [ ]         STREAM     CONNECTED     17310    /var/run/dbus/system_bus_socket
unix  3      [ ]         STREAM     CONNECTED     185857   /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     13774    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     185856
unix  3      [ ]         STREAM     CONNECTED     13773
unix  3      [ ]         STREAM     CONNECTED     13599
unix  2      [ ]         DGRAM                    15855
unix  2      [ ]         DGRAM                    19795
unix  2      [ ]         DGRAM                    2278770
unix  2      [ ]         DGRAM                    16006
unix  3      [ ]         STREAM     CONNECTED     10347490
unix  3      [ ]         STREAM     CONNECTED     190296
unix  3      [ ]         STREAM     CONNECTED     13600    /var/run/dbus/system_bus_socket
unix  3      [ ]         STREAM     CONNECTED     190297   /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     2858305  @/containerd-shim/moby/01ffe3c2b61ef52d9413b80aae21d90f59ed4b30fb7f092153e9164cd4fd98f6/shim.sock
unix  2      [ ]         DGRAM                    17297
unix  3      [ ]         STREAM     CONNECTED     17295
unix  3      [ ]         STREAM     CONNECTED     12988
unix  3      [ ]         STREAM     CONNECTED     19789    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     12645    /run/systemd/journal/stdout
unix  2      [ ]         DGRAM                    15458
unix  3      [ ]         STREAM     CONNECTED     12644
unix  2      [ ]         DGRAM                    10347426
unix  3      [ ]         STREAM     CONNECTED     10347391 /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     19788
unix  3      [ ]         STREAM     CONNECTED     12990    /var/run/dbus/system_bus_socket
unix  2      [ ]         DGRAM                    10930
unix  3      [ ]         DGRAM                    10967
unix  2      [ ]         DGRAM                    12970
unix  3      [ ]         STREAM     CONNECTED     10348912 @/containerd-shim/moby/f861143c120e5fbda684a932c1783290bcd57f7c4ac972c801fff497ecc8112b/shim.sock
unix  3      [ ]         DGRAM                    10966
unix  3      [ ]         STREAM     CONNECTED     13337    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     13336
unix  2      [ ]         DGRAM                    10347533
unix  3      [ ]         STREAM     CONNECTED     12936
unix  3      [ ]         STREAM     CONNECTED     10720    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     10719
unix  3      [ ]         STREAM     CONNECTED     1506746
```

### linux查看端口占用

https://www.cnblogs.com/wangtao1993/p/6144183.html
```
sudo netstat -tunlp |grep 8001
```

## 配置dns

vi /etc/resolv.conf

domain sankuai.info
nameserver 192.168.4.251
nameserver 192.168.6.251