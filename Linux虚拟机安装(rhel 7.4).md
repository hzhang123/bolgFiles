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