---
title: Vagrant+VirtualBox虚拟环境
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 软件安装

安装都比较简单，下载一直点击Next。

VirtualBox安装：[官网主页](https://www.virtualbox.org/)

Vagrant安装：[官网主页](https://www.vagrantup.com/)

# 虚拟机

查看命令与子命令帮助文档：
==vagrant -h==
==vagrant COMMAND -h==
box子命令：==vagrant box <subcommand> -h==

1. 到VirtualBox上找到一个自己需要的虚拟机，这里使用==centos/7==

![centos/7](./images/1573190736287.png)

2. 初始化并启动虚拟机
``` shell
# 创建初始化目录
mkdir -p vagrant_centos;
# 添加镜像，并输入选择自己的虚拟平台，这里选择3，virtualbox
vagrant box add centos/7;
# 进入vagrant目录，查看可用box并初始化启动
cd vagrant_centos;
vagrant box list;
vagrant init centos/7;
vagrant up
# 命令行直接登录虚拟机
vagrant ssh
```

![初始化并启动虚拟机](./images/1573278645453.png)

![登录虚拟机输出信息并退出](./images/1573283018164.png)