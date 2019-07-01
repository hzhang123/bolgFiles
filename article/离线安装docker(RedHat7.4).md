---
title: 离线安装docker(RedHat7.4)
tags: docker
---


----------

[toc]

## 1. 下载地址

官网下载地址:[下载](https://download.docker.com/linux/static/stable/x86_64/)
官网文档地址:[文档](https://docs.docker.com/install/linux/docker-ce/binaries/)

## 2. 解压并注册为service

1. 下载安装

``` shell
# 下载
tarball="docker-18.09.7.tgz"
wget -c https://download.docker.com/linux/static/stable/x86_64/${tarball}
# 解压
tar -zxvf ${tarball}
# 复制到/usr/bin
cp docker/* /usr/bin
```

2. 添加到service: ==vim /etc/systemd/system/dockerd.service==

``` dsconfig
[Unit]

Description=Docker Application Container Engine

Documentation=https://docs.docker.com

After=network-online.target firewalld.service

Wants=network-online.target

[Service]

Type=notify

# the default is not to use systemd for cgroups because the delegate issues still

# exists and systemd currently does not support the cgroup feature set required

# for containers run by docker

ExecStart=/usr/bin/dockerd

ExecReload=/bin/kill -s HUP $MAINPID

# Having non-zero Limit*s causes performance problems due to accounting overhead

# in the kernel. We recommend using cgroups to do container-local accounting.

LimitNOFILE=infinity

LimitNPROC=infinity

LimitCORE=infinity

# Uncomment TasksMax if your systemd version supports it.

# Only systemd 226 and above support this version.

#TasksMax=infinity

TimeoutStartSec=0

# set delegate yes so that systemd does not reset the cgroups of docker containers

Delegate=yes

# kill only the docker process, not all processes in the cgroup

KillMode=process

# restart the docker process if it exits prematurely

Restart=on-failure

StartLimitBurst=3

StartLimitInterval=60s

 

[Install]

WantedBy=multi-user.target
```

3. 启动服务

``` shell
#添加文件权限并启动docker
chmod +x /etc/systemd/system/dockerd.service
#重载unit配置文件
systemctl daemon-reload
#启动dockerd
systemctl start dockerd        
#设置开机自启
systemctl enable dockerd.service
```

4. 检查状态

``` shell
# 状态
systemctl status dockerd
# 版本
docker -v
```