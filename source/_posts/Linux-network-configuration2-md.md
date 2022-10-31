---
title: Linux网络配置（二）
date: 2022-10-31 10:53:00
tags: 
   - Linux
   - 网络配置
categories: Linux
top_img: /img/post3/top_img.jpg
cover: /img/post3/top_img.jpg
---
## 检查网卡

查看网卡是否存在以及相应模块是否加载：

```bash
# lspci 命令用于显示 Linux 系统上的设备和驱动程序
lspci -k

# 输出结果

## Ethernet controller 为有线网卡（以太网卡）
02:05.0 Ethernet controller: Intel Corporation 82545EM Gigabit Ethernet Controller (Copper) (rev 01)
        DeviceName: Ethernet1
        Subsystem: VMware PRO/1000 MT Single Port Adapter
        Kernel driver in use: e1000
        Kernel modules: e1000
        
## Network controller 为无线网卡（WIFI网卡）
06:00.0 Network controller: Intel Corporation WiFi Link 5100
	Subsystem: Intel Corporation WiFi Link 5100 AGN
	Kernel driver in use: iwlwifi
	Kernel modules: iwlwifi
	
# dmesg 命令是用于显示内核的相关信息，用该命令查看相关驱动是否成功加载，module_name 为上述命令输出的模块名
dmesg | grep module_name 

# 输出结果
[   13.726383] e1000: ens37 NIC Link is Up 1000 Mbps Full Duplex
```

如果网卡存在，驱动没有正确加载，则需要先安装相应的驱动。

无线网卡可以通过 USB 接入系统（例如在虚拟机中，想要直接通过无线网卡连接 WiFi，可通过 USB 外接无线网卡的形式实现），此时查看可用 lsusb 命令查看无线网卡信息：

```bash
lsusb -v

# 输出结果

# Wireless Adapter 为无线网卡
Bus 001 Device 002: ID 7392:7811 Edimax Technology Co., Ltd EW-7811Un 802.11n Wireless Adapter [Realtek RTL8188CUS]
```

## 连接网络

### 有线网络

#### 以太网

激活以太网卡，插入网线，基本就可以连接到网络了，一般不需要选择网络和输入密码，如果网络未连接，则检查网卡，网络通路等。

#### PPPoE

PPPoE 是一种拨号上网（ADSL），连接这种类型的网络需要输入用户名与密码进行认证。对于 DEB 包发行版，安装 pppoeconf 包；对于 RPM 包发行版，安装 rp-pppoe 包。

以 rp-pppoe 包为例展示设置网络的过程：

```bash
pppoe-setup    # 配置PPPoE连接

# 输入用户名：
# 输入以太网卡代号：（根据实际网卡名配置）
# 配置：若长时间连线，连线会被自动中断：（选no）
# 配置主DNS服务器：（无需配置或者114.114.114.114）
# 配置次DNS服务器：（无需配置或者8.8.8.8）
# 两次输入账户密码以确认（宽带密码，输入没有不显示“*”，如果输错，会提示是否重新输入，选择y）
# 配置普通账户是否有网络连接权限（y或者n）
# 配置防火墙（没有特殊需求选0不配置）
# 配置是否开机自动拨号连接（yes或no）
# 确认刚填写的配置信息（y或n）

pppoe-connect [configuration_file_path] # 建立连接
pppoe-start [configuration_file_path]   # 启动软件进行连接，当出现“Connected”就表示连接成功了
pppoe-stop [configuration_file_path]    # 关闭连接
pppoe-status     # 查看网络状态
```

pppoe-setup 生成的配置文件是 /etc/ppp/pppoe.conf，可以修改该文件调整参数，用户名与密码存储在 /etc/ppp/chap-secrets 文件中。

### 无线网络

#### 普通无线网络

目前很少有开放 WIFI，即连接 WIFI 不需要输入密码，大部分 WIFI 网络都需要进行密码验证。连接无线网络可以使用网络管理软件或者以下工具：

|    Software    |    Package     | WEXT | nl80211 | WEP  | WPA/WPA2 |
| :------------: | :------------: | :--: | :-----: | :--: | :------: |
| wireless_tools | wireless_tools | Yes  |   No    | Yes  |    No    |
|       iw       |       iw       |  No  |   Yes   | Yes  |    No    |
| wpa_supplicant | wpa_supplicant | Yes  |   Yes   |  No  |   Yes    |
|      iwd       |      iwd       |  No  |   Yes   |  No  |   Yes    |

##### wireless_tools

Wireless Extension (WE)是一组通用的 API，能在用户空间对通用 Wireless LANs 进行配置和统计，Wireless Tools (WT)就是用来操作Wireless Extensions的工具集。常用命令为 iwconfig iwlist.

```bash
iwlist wlan0 scan   # 扫描可用的 WIFI 接入点
iwconfig wlan0      # 查看连接状态
iwconfig wlan0 essid your_essid   # 连接一个开放的 WIFI
iwconfig wlan0 essid your_essid key s:your_key  # 连接一个 WEP 加密的 WIFI，密码为 ASCII 格式
```

#####  iw

iw 是一种新的基于 nl80211 的用于无线设备的 CLI 配置实用程序，iw 取代了采用无线扩展接口的旧工具iwconfig。

```bash
iw dev  # 显示网卡
iw dev wlan0 scan   # 扫描可用的 WIFI 接入点
iw dev wlan0 link   # 查看连接状态
iw dev wlan0 connect your_essid  # 连接一个开放的 WIFI
iw dev wlan0 connect your_essid key d:0:your_key  # 连接一个 WEP 加密的 WIFI，密码为 ASCII 格式，d：default，0：表示第0个密码
```

##### wpa_supplicant

iwconfig 和 iw 只能连接采用 WEP 加密方式的 WIFI。WPA（Wi-Fi Protected Access），意即“Wi-Fi访问保护”，是一种由Wi-Fi联盟制订与发布，用来保护无线网络（Wi-Fi）访问安全的技术标准。前一代有线等效加密（Wired Equivalent Privacy, WEP）系统中，被发现若干严重的弱点，因此Wi-Fi联盟推出WPA、WPA2与WPA3系列来加强无线网络安全。wpa_supplicant是一个开源项目，已经被移植到Linux，Windows以及很多嵌入式系统上。它是WPA的应用层认证客户端，负责完成认证相关的登录、加密等工作。

使用 wpa_cli 命令配置网络连接，首先需要创建一个配置文件 /etc/wpa_supplicant/wpa_supplicant.conf，内容如下：

```bash
ctrl_interface=/run/wpa_supplicant
update_config=1
```

然后开启 wpa_supplicant：

```bash
wpa_supplicant -B -i interface -c /etc/wpa_supplicant/wpa_supplicant.conf
```

然后运行 wpa_cli：

```bash
wpa_cli

> scan            # 使用 scan 扫描网络
<3>CTRL-EVENT-SCAN-RESULTS
> scan_results    # 使用 scan_results显示扫描结果
bssid / frequency / signal level / flags / ssid
00:00:00:00:00:00 2462 -49 [WPA2-PSK-CCMP][ESS] MYSSID
11:11:11:11:11:11 2437 -64 [WPA2-PSK-CCMP][ESS] ANOTHERSSID
> add_network     # 添加网络
0
> set_network 0 ssid "MYSSID"     # 选择网络
> set_network 0 psk "passphrase"  # 输入密码，如果没有密码使用：set_network 0 key_mgmt NONE
> enable_network 0                # 开启连接
> save_config      # 保存
OK
> quit    # 退出
```

##### iwd

[iwd](https://iwd.wiki.kernel.org/) (iNet wireless daemon，iNet 无线守护程序) 是由英特尔（Intel）为 Linux 编写的一个无线网络守护程序。该项目的核心目标是不依赖任何外部库，而是最大程度地利用 Linux 内核提供的功能来优化资源利用。

```bash
iwctl   # 进入交互式提示符
> help  # 列出所有可用的命令
> device list         # 列出所有 WiFi 设备
> station wlan0 scan  # 扫描网络
> station wlan0 get-networks    # 列出所有可用的网络
> station wlan0 connect SSID    # 连接到一个网络，如果要求输入网络密码，将会提示用户输入
```

此外，连接操作可以应用成命令行参数的形式：

```bash
iwctl --passphrase your_key station wlan0 connect SSID
```

#### 使用网页认证的无线网络

现在有很多 WIFI 使用了网页认证（[Captive Portal](https://link.zhihu.com/?target=https%3A//www.wikiwand.com/en/Captive_portal)），Captive portal，又名强制网络门户、强制主页，是在授予新连接至 WIFI 或接受最终用户许可协议/可接受使用策略的着陆页或登录页。强制门户应用于方方面面的移动宽带服务中（如有线连接、计费Wi-Fi及家庭热点），同时也可提供对企业或家庭有线网络（公寓、酒店和商业中心的网络）的访问权限。

这种网络通过 Web + DHCP 认证方式解决无线用户接入问题，常见于无线校园网中。通常主机连接上无线网络后，DHCP服务器就会给主机分配一个 IP 地址，如果用户没有认证登录，在浏览器访问的任何 IP 地址都会被重定向到 WEB 认证页面。Windows和安卓系统在连接到网络后一般会自动跳出登录界面；而在linux中，当连接上此类 WIFI 时可能并不会弹出网页认证的界面，解决方法如下：

1. [NetworkManager/Captive portals](https://wiki.archlinux.org/title/NetworkManager#Captive_portals)
2. [captive-browser-git](https://github.com/FiloSottile/captive-browser)

## 参考文献

[1] [Network configuration/Ethernet](https://wiki.archlinux.org/title/Network_configuration/Ethernet)
[2] [Network configuration/Wireless](https://wiki.archlinux.org/title/Network_configuration/Wireless)
[3] [在Linux操作系统下的PPPoE拨号上网](https://blog.csdn.net/qq_26733603/article/details/109682400)
[4] [强制门户](https://zh.m.wikipedia.org/zh-hans/%E5%BC%BA%E5%88%B6%E9%97%A8%E6%88%B7)