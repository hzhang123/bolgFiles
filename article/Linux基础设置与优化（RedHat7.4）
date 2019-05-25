---
title: Linux基础设置与优化（RedHat7.4） 
tags: linux
---


----------

[toc]

# 1. 基本设置

``` shell
# 关闭防火墙
systemctl stop firewalld
# 关闭SELinux
sed -ri 's/^SELINUX=.*$/SELINUX=disabled/g' /etc/selinux/config

# WARNING! The remote SSH server rejected X11 forwarding request解决
sed -ri 's/.*X11Forwarding.*/X11Forwarding yes/g; s/.*UseLogin.*/UseLogin no/g' /etc/ssh/sshd_config
# 安装xorg-x11-xauth
yum install xorg-x11-xauth -y
cd ~; touch .Xauthority;
# 重启sshd
systemctl restart sshd

# 重启机器，然后检查状态
reboot

# 检查防火墙
systemctl status firewalld
# 检查SElinux
sestatus -v
```

# 2. hosts


