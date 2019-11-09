---
title: Vagrant+VirtualBox虚拟环境
tags: Vagrant,VirtualBox
grammar_cjkRuby: true
---

[toc]

# 软件安装

安装都比较简单，下载一直点击Next。

VirtualBox安装：[官网主页](https://www.virtualbox.org/)

Vagrant安装：[官网主页](https://www.vagrantup.com/)

# 虚拟机基础配置

## 虚拟机创建

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

3. ssh登录，先试用命令查看默认ssh配置：==vagrant ssh-config==

![默认ssh配置](./images/1573285870469.png)

``` shell
# ssh使用默认的秘钥登录
# -p 2222	指定端口
# vagrant	登录角色
# 127.0.0.1	虚拟机IP
# -i	指定秘钥
ssh -p 2222 vagrant@127.0.0.1 -i /Users/growingio/developments/vagrant_centos/.vagrant/machines/default/virtualbox/private_key
```

5. 常用启停管理命令

**启动**：vagrant up
**停止**：vagrant halt
**暂停**：vagrant suspend
**恢复**：vagrant resume
**重启**：vagrant reload
**销毁**：vagrant destroy


## 共享目录

1. 基础共享目录

``` shell
# 项目虚拟机所在的目录就是默认的共享目录，不过文件的共享需要虚拟机重启
mkdir test
echo "This is a test shared file." > test/file;
```

![创建共享文件](./images/1573287816498.png)

![共享文件](./images/1573287381279.png)

2. 自定义共享目录

需要先安装，否则会报错==mount: unknown filesystem type 'vboxsf'==


``` shell
# 定位到config.vm.synced_folder所在行，编辑信息
# ../data	本机目录
# /vagrant_data	虚拟机目录
# 创建以及权限
config.vm.synced_folder "../data", "/vagrant_data", create:true, owner:"root", group:"root"

# vagrant重启机器
vagrant reload

```

