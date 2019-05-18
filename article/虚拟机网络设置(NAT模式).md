---
title: 虚拟机网络设置(NAT模式)
tags: linux
---

----------


[toc]

# 1. 设置虚拟机网络

## 1.1. NAT子网设置

1. VMware首页点击 -> 编辑 -> 虚拟网络编辑器

2. 设置子网

![设置子网](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924586970.png)

3. dhcp设置起止IP地址

![起止IP地址](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924677854.png)

4. 选中将主机连接到此网络

![主机连接到此网络](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924710030.png)

## 1.2. 网卡配置文件设置

打开文件：==/etc/sysconfig/network-scripts/ifcfg-ens33==(文件可能不是ens33)，根据需要编辑如下内容，有备注的基本为必须配置

==注： #E91E63==IP地址属于上一步dhcp 起止IP地址范围内，一般设置*.\*.\*.1为网关。

``` profile
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=yes
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=56029188-f0ab-4f1e-a94b-87b7ccd63a07
DEVICE=ens33
# 静态IP
BOOTPROTO=static
# 开机启动
ONBOOT=yes
# IP地址
IPADDR=192.168.2.3
# 子网掩码
PREFIX=24
# 网关
GATEWAY=192.168.2.1
# DNS
DNS1=114.114.114.114
DNS2=8.8.8.8
```

redhat6.X的一般如下：/etc/sysconfig/network-scripts/ifcfg-eth0

``` profile
#---------------------------------------+
#静态IP配置选项                         |
#---------------------------------------+
#开机启动网卡
ONBOOT=yes
#IP地址静态
BOOTPROTO=static
#IP4地址(以此为例，填写101之后的不可以冲突)
IPADDR=192.168.2.3
#子网掩码
NETMASK=255.255.255.0
#网关
GATEWAY=192.168.2.1
#---------------------------------------
```

## 1.3. 重启网络服务

``` shell
# network服务重启
systemctl restart network
# 启动之后查看ip和route信息
ip addr show
ip route show

# 6.x重启与ip路由信息
service network restart
ifconfig
route
```

![ip和route信息](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925228233.png)

## 1.4. 配置端口转发

1. VMware首页点击 -> 编辑 -> 虚拟网络编辑器
2. 选中NAT模式，点击NAT设置，网关配置为与虚拟机配置网关一致，点击添加。

![NAT设置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925441309.png)

![虚拟机IP下的22端口映射到主机的22端口](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925522252.png)

3. 点击确定之后通过xshell ssh本地电脑IP地址或虚拟机IP地址都可登录访问虚拟机。

# 2. 配置网络共享
1. 打开：==控制面板\网络和 Internet\网络连接 #E91E63==， 鼠标右键当前连接网络的网卡，点击属性。

2. 设置将当前网络共享到虚拟的网卡之上。
![设置Internet共享网卡](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558104378732.png)

3. 右键虚拟网卡VMnet8， 设置IP与网关。

![属性设置前置界面](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558105152025.png)

![属性设置界面](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558105190124.png)

- IP地址：设置与网关同网段的一个地址，但注意不要与==虚拟机IP冲突 #03A9F4==。
- 子网掩码：NAT模式中设置的子网掩码。
- 默认网关：NAT模式设置、虚拟机网关、都是相同的一个网关。
- DNS：设置一个公共DNS即可。

**==注： #E91E63==** 如果配置虚拟网卡的时候显示IP冲突，一定要重启一下网卡，因为可能自动胡乱分配的一个IP。

![随机分配IP](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558105107136.png)

4. 远程连接虚拟机，是否能curl访问通外网与ping其他虚拟机。

![测试网络](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558106651650.png)
