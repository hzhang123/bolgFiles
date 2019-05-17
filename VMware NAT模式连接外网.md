---
title: VMware NAT模式连接外网
tags: linux
---


# 配置网络共享
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
