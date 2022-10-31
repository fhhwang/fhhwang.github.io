---
title: Linux网络配置（一）
date: 2022-10-31 10:18:21
tags: 
   - Linux
   - 网络配置
categories: Linux
top_img: /img/post3/top_img.jpg
cover: /img/post3/top_img.jpg
---
## 网络配置

大家平时经常碰到的网络是局域网（LAN），局域网又分为有线局域网和无线局域网（WLAN），其中以太网（Ethernet）是最常见的有线局域网，WIFI是最常见的无线局域网。除此之外，还有拨号网络PPPoE（Point-to-Point Protocol over Ethernet），即以太网上的点对点协议（PPP）。以太网技术虽然具有简单易用，成本低等特点，但是以太网广播网络的属性，使得其通信双方无法相互验证对方的身份，因而通信是不安全的；PPPoE结合了PPP协议通信双方身份验证的功能，在PPP组网结构的基础上，将PPP报文封装成PPPoE报文，从而实现以太网上的点对点通信，使得以太网中的客户端能够连接到远端的宽带接入设备上，实现了传统以太网不能提供的身份验证、加密以及压缩等功能。

本文描述如何在 Linux 系统中配置 ISO 模型的网络层（Network Layer）连接。

### 网络连接

首先确保电脑中装有网卡，排查网络是否连接的详细步骤如下：

1. 检查网卡是列出在和启用，如果没有，则检查网卡驱动程序是否加载；
2. 电脑连接到网络，有线网络（Ethernet）或者无线网络（wireless LAN）；
3. 检查网卡是否有一个 IP 地址；
4. 正确配置路由表；
5. 检查是否可以 ping 通本地 IP 地址（如默认网关）；
6. 检查是否可以 ping 通公网 IP 地址（如 9.9.9.9，该 IP 为一个 DNS 服务器）；
7. 检查是否可以 ping 通一个网络域名（如 archlinux.org）.

首先介绍一些网络管理工具。

#### ping 命令

```bash
ping domain_name/ip_address # ping 后面的测试可以是网站域名或者 IP 地址

# 示例：
ping archlinux.org

# 输出结果
PING archlinux.org (95.217.163.246) 56(84) bytes of data.
64 bytes from archlinux.org (95.217.163.246): icmp_seq=1 ttl=128 time=279 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=128 time=234 ms
64 bytes from archlinux.org (95.217.163.246): icmp_seq=3 ttl=128 time=233 ms
^C
--- archlinux.org ping statistics ---
4 packets transmitted, 3 received, 25% packet loss, time 3090ms
rtt min/avg/max/mdev = 232.841/248.541/278.688/21.323 ms
```

ping 命令包含在 [iputils](https://github.com/iputils/iputils)包中，在一些精简的系统中，要使用 ping 命令首先需要安装 iputils 包。使用 Ctrl+c 即可终止输出，从输出结果中可以看出测试结果，需要注意 ping 命令测试是运行的是网络层中 ICMP 协议， 当代理软件运行在应用层时，不能用该命令测试网络的连通性。

#### net-tools

net-tools 包含了一系列 Linux 网络的基础程序，其中最常见的有 arp ifconfig netstat route.

##### arp 

ARP（Address Resolution Protocol，地址解析协议）是用来将IP地址解析为MAC地址的协议。 主机或三层网络设备上会维护一张ARP表，用于存储IP地址和MAC地址的映射关系，一般ARP表项包括动态ARP表项和静态ARP表项。

arp 命令用于操作内核 ARP 缓存，一般很少需要手动操作 ARP 表项。

```bash
arp -v                   # 显示 arp 缓冲区内容
arp -s IP MAC-ADDRESS    # 添加静态 arp 映射　
arp -d IP                # 删除 arp 缓存条目
```

##### ifconfig (network interfaces configuring)

ifconfig 是配置网卡的主要命令，其功能是用于显示或设置网络设备参数信息，在Windows系统中与之类似的命令叫做 ipconfig，使用 ifconfig 命令配置网络设备的参数信息临时生效，当服务器重启，配置过的参数会自动失效，如果需要永久改变，则需要写入配置文件中。

```bash
# 语法格式：ifconfig [网卡设备] [参数]
ifconfig        # 显示网卡信息
ifconfig -a     # 显示的是系统所有的网络接口，不管是激活的还是未激活的
ifconfig network_interface up/down    # 对指定的网卡设备进行启动或关闭操作
ifconfig network_interface 192.168.10.20 netmask 255.255.255.0  # 临时修改IP地址
```

注意一下网卡的命名规则，默认情况下，udev 分配网卡名字，以太网前缀 en，WIFI 前缀 wl. 

##### netstat

netstat是控制台命令,是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。Netstat用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。

```bash
netstat -ltup
# -l 只显示监听端口
# -t 列出 tcp 端口
# -u 列出 udp 端口
# -p 输出中显示 PID 和进程名称
```

实际使用中可以用 grep 命令过滤输出。

##### route

route 命令是用于操作基于内核 IP 路由表。

```bash
route -n   # 显示当前路由
oute add/del -net ip_addr netmask ip_addr dev net_interface  # 添加网/删除网关
route add/del default gw ip_addr  # 添加/删除设置默认网关
```

#### traceroute

traceroute 用于探测数据包从源到目的经过路由的 IP，使用该命令前需要安装 traceroute 包。traceroute 的原理是试图以最小的 TTL（存活时间）发出探测包来跟踪数据包到达目标主机所经过的网关，然后监听一个来自网关 ICMP 的应答。

```bash
 # 示例：
 traceroute archlinux.org
```

#### iproute2

iproute2 的出现旨在从功能上取代 net-tools，net-tools 通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。抛开性能而言，iproute2的用户接口比net-tools显得更加直观。

iproute2 的核心命令是 ip 命令，它与 net-tools 中命令的对应关系如下：

| net-tools command | iproute2 commands   |
| :---------------- | :------------------ |
| arp               | ip neighbor         |
| ifconfig          | ip address, ip link |
| netstat           | ss                  |
| route             | ip route            |

ip 命令用法示例：

```bash
ip link show                      # 显示出所有可用网络接口的列表
ip addr [show dev eth1]           # 查看某个指定网络接口的IPv4地址
ip link set up/down eth1          # 激活或停用某个指定的网络接口
ip addr add 10.0.0.1/24 [broadcast 10.0.0.255] dev eth1  # 可以使用 iproute2 给同一个接口分配多个 IP 地址
ip addr del 10.0.0.1/24 dev eth1  # 移除 IP 地址
ip link set dev eth1 address 08:00:27:75:2a:67  # 更改网络接口的MAC地址
```

#### DHCP

通常情况下我们不需要手动配合静态 IP，只需要去动态获取 IP 即可。一个网络中的 DHCP 服务器给客户端提供了动态 IP 地址、子网掩码、默认网关，以及可选的 DNS 服务。

在客户端安装 DHCP 的客户端即可使用该服务。Linux 上常用的 DHCP 客户端是 dhcpcd，如下表所示：

| Client | Package |               Note                |                 Systemd units                  |
| :----: | :-----: | :-------------------------------: | :--------------------------------------------: |
| dhcpcd | dhcpcd  | DHCP, DHCPv6, ZeroConf, static IP | `dhcpcd.service`, `dhcpcd@*interface*.service` |

很多网络管理软件都内置了 DHCP 服务。

## 网络管理软件

介绍几种网络管理软件，这些网络管理软件用于管理网络连接。使用这些软件很大程度上降低了网络连接的复杂性。常用的网络管理软件如下：

| Network manager  |     GUI      |     CLI tools     | PPP support |     DHCP client      |                        systemd units                         |
| :--------------: | :----------: | :---------------: | :---------: | :------------------: | :----------------------------------------------------------: |
|      netctl      | 2 unofficial | netctl, wifi-menu |     Yes     |  dhcpcd or dhclient  | `netctl-ifplugd@*interface*.service`, `netctl-auto@*interface*.service` |
|  NetworkManager  |     Yes      |   nmcli, nmtui    |     Yes     | internal or dhclient |                   `NetworkManager.service`                   |
| systemd-networkd |      No      |    networkctl     |     No      |       internal       |    `systemd-networkd.service`, `systemd-resolved.service`    |

### NetworkManager

[NetworkManager](https://networkmanager.dev/) 是一个为系统提供检测和配置功能以便自动连接到网络的程序。支持 PPPoE 拨号，集成了 DHCP 服务。nmcli 是配置网络的命令行工具，nmtui 提供一个图形化文本界面来与NetworkManager交互用于配置网络。

## 本机地址

### Hostname

在网络中，hostname 是一个分配给主机的一个域名，通常是一个主机的本地名字加上可选的域名，用点（.）分隔，例如 en.wikipedia.org（主机名：en，域名：wikipedia.org），这种主机名通过本机 hosts 文件或者 DNS 服务器解析为 IP 地址。一个主机可以拥有多个主机名。

可以通过修改 /etc/hostname 文件配置主机名，该文件包含单独的一行 *myhostname*，或者通过命令更改：

```bash
hostnamectl set-hostname myhostname  # 等同于修改 /etc/hostname
hostname myhostname   # 临时更改，重启失效
```

要使你的设备能在局域网中被识别，有以下两种方法：

1. 在局域网上编辑每一台主机的 /etc/hosts 文件，加上你的主机；
2. 配置一个 DNS 服务器，解析域名。

### Hosts

Hosts 文件位于 /etc/hosts，以表的形式存储了主机名和 IP 地址的映射关系。Hosts文件是大多数系统都存在的一个小型主机表。Hosts文件中包含了本地网络重要的主机名和地址信息，查询Hosts文件得到的结果比通过查询 DNS 得到的结果优先级更高。

### Localhost

localhost 是一个在计算机网络中用于表示“此计算机”的主机名，它被用于通过本地回环网络接口，来访问本机运行的服务，并且将会绕过任何物理网络接口硬件。运用本地环回机制，便可在主机上运行网络服务，期间不须安装实体网络接口卡，也无须将该服务开放予主机所在网络。例如，在设置好本地安装的网站后，可通过http://localhost这一网址，来访问本地网站。localhost这个主机名称一般会解析为 IPv4 本地回环地址 127.0.0.1 和 IPv6 本地回环地址 [::1].

为了解析本机主机名，可在 /etc/hosts 中添加以下信息：

```bash
127.0.0.1        localhost
::1              localhost
127.0.1.1        myhostname
```

如果本机有一个永久的 IP 地址，则可配置：

```bash
127.0.0.1        localhost
::1              localhost
203.0.113.45     host1.fqdomain.example host1
```

一般情况下本机有三块网卡：

1. lo：回环网卡（Loopback Adapter），是一块虚拟网卡；
2. eth0：有线以太网卡；
3. wl0：WIFI无线网卡。

localhost、127.0.0.1、0.0.0.0和本机IP的区别：

1. localhost 是一个指向本机的域名，通过http://localhost这一网址，来访问本地网站，localhost 指向的 IP 地址是可以配置的。不联网，不使用网卡，不受防火墙和网卡限制，本机访问。
2. 127.0.0.1 是一个回环地址（Loop back address），通常分配给 loopback 接口。凡是以 127 开头的 IP 地址，都是回环地址，都指向 lo 网卡。发送给 127 开头的 IP 地址的数据包会被发送的主机自己接收，根本传不出去，外部设备也无法通过回环地址访问到本机。不联网，网卡传输，受防火墙和网卡限制，本机访问。
3. 0.0.0.0是不能被 ping 通的，0.0.0.0称为“unspecified”，即未指定（即无效的，无意义的）地址。DHCP客户端还未获取到ip的时候规定使用0.0.0.0作“源地址”；在服务器中，0.0.0.0并不是一个真实的的IP地址，它表示本机中所有的IPV4地址。监听0.0.0.0的端口，就是监听本机中所有IP的端口。
4. 本机IP通常仅指在同一个局域网内，能同时被外部设备访问和本机访问的那些IP地址（可能不止一个）。像127.0.0.1这种一般是不被当作本机IP的。本机IP是与具体的网络接口绑定的，比如以太网卡、无线网卡或者PPP/PPPoE拨号网络的虚拟网卡，想要正常工作都要绑定一个地址，否则其他设备就不知道如何访问它。联网，网卡传输，受防火墙和网卡限制，本机或外部访问。

## 参考文献

[1] [Network_configuration](https://wiki.archlinux.org/title/Network_configuration)
[2] [什么是PPPoE？PPPoE解决了哪些问题？](https://info.support.huawei.com/info-finder/encyclopedia/zh/PPPoE.html)
[3] [localhost、127.0.0.1和0.0.0.0和本机IP的区别](https://www.cnblogs.com/absoluteli/p/13958072.html)

