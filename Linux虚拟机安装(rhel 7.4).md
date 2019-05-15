---
title: Linux虚拟机安装（rhel 7.4）
tags: linux
---


----------

[toc]

# 1. 创建虚拟机

## 1.1. 新建虚拟机

![新建虚拟机](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921624901.png)

![典型虚拟机](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921667693.png)

![稍后安装系统](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921696539.png)

![系统版本选择](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921741998.png)

![虚拟机名称与位置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921825184.png)

**==位置路径尽量不要存在中文 #E91E63==**

![磁盘配额](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557921984071.png)

![选择硬件](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922085236.png)

![ISO镜像选择](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922146639.png)
**==注： #E91E63==**其它硬件如无修改无需改动点击完成即创建完成，DVD选择也可通过如下图所示地址修改。

![硬件设置方法2](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922299100.png)

## 1.2. 启动虚拟机

1. 点击开启虚拟机
2. 如有提示框，关闭或确定即可；选择第一个选项直接安装，白色为选择

![选第一个选项直接安装](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922433274.png)

3. 等待一段时间，进入选择语言界面。

![选择语言](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922533973.png)

4. 本地化、软件、系统设置。如果不选择桌面版，设置一个时区和选择硬盘即可，软件可安装完成之后再安装（这里选择的上海）

![时区和硬盘](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557922912083.png)

5. 点击Begin installation

![root密码与新建用户](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557923077228.png)

![密码weak多点击两遍done既可以设置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557923025969.png)

6. 等待安装完成，点击Reboot。启动之后输入root和之前设置的密码登录

![点击reboot](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557923770613.png)


# 2. 设置虚拟机网络

## 2.1. 子网设置

1. VMware首页点击 -> 编辑 -> 虚拟网络编辑器

2. 设置子网

![设置子网](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924586970.png)

3. dhcp设置起止IP地址

![起止IP地址](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924677854.png)

4. 选中将主机连接到此网络

![主机连接到此网络](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557924710030.png)

## 2.2. 网卡配置文件设置

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
IPADDR=192.168.2.202
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
IPADDR=192.168.228.101
#子网掩码
NETMASK=255.255.255.0
#网关
GATEWAY=192.168.228.1
#---------------------------------------
```

## 2.3. 重启网络服务

``` shell
# network服务重启
systemctl restart network
# 启动之后查看ip和route信息
ip addr show
ip route show
```

![ip和route信息](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925228233.png)

## 2.4. 配置端口转发

1. VMware首页点击 -> 编辑 -> 虚拟网络编辑器
2. 选中NAT模式，点击NAT设置，网关配置为与虚拟机配置网关一致，点击添加。

![NAT设置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925441309.png)

![虚拟机IP下的22端口映射到主机的22端口](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1557925522252.png)

3. 点击确定之后通过xshell ssh本地电脑IP地址或虚拟机IP地址都可登录访问虚拟机。

# 附录：部分配置
修改文件：/etc/ssh/sshd_config
1. 连接过慢
UseDNS yes为UseDNS no
#当客户机通过ssh登录的时候，虚拟机会根据客户机的IP地址进行DNS PTR反向解析出客户机的主机名，然后根据查询出的主机名进行DNS正向查询记录，验证与其原始IP是否一致，这是客户端防止欺骗的措施，但是大部分IP没有配置PTR记录，所以在局域网保证安全的情况下可以关闭。
UseDNS no

2. 断开连接
#ClientAliveInterval指定了服务器端向客户端请求消息的时间间隔, 默认是0, 不发送。设置60表示每分钟发送一次, 然后客户端响应, 这样就保持长连接了。
#ClientAliveCountMax表示服务器发出请求后客户端没有响应的次数达到一定值, 就自动断开。正常情况下, 客户端不会不响应，使用默认值3即可。

**配置完成重启sshd服务：systemctl restart sshd**