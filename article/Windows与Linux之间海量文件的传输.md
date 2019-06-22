---
title: Windows与Linux之间海量文件的传输与大小写敏感问题
tags: 
---

> mount.cifs 支持通过网络文件系统挂载，不过需要安装cifs-utils，也可通过mount -t cifs挂载，详细的选项可参见man mount.cifs

# 1. 通过Windows共享文件夹

## 1.1 设置windows共享

右键待共享文件夹 -> 属性 -> 共享 -> 高级共享 -> 勾选共享此文件夹

![共享属性界面](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561190068098.png)

这里设置了需要账户和密码才可访问

**==注： #E91E63==** 如果需要从linux往windows下写数据，需要从高级共享内修改用户权限即可

## 1.2 Linux下挂载共享文件夹

``` shell
# mount -t cifs -o username=用户名,password=密码 //网络地址/共享名 挂载点

# 如将上面共享的文件夹挂载到/mnt下(网络地址填共享机器IP)
# mount挂载
mount -t cifs -o username=用户名,password=密码 //网络地址/testDir /mnt
# mount.cifs挂载
mount.cifs -o username=用户名,password=密码 //网络地址/testDir /mnt
# 检查是否挂载成功
df -h
```
![挂载设备](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561191178376.png)

![windows下文件](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561191439659.png)

![linux挂载点下文件](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561191579214.png)

# Linux与Windows大小写问题

**文件路径区分大小写的是由文件系统决定的**

1. 将文件复制到linux机器上/home/disk0目录

``` shell
# 复制文件
cp -R /mnt/* /home/disk0
# 查看图片
ls /home/disk0/pic/2019/20190621/girl.jpg
```
![linux下区分大小写](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561193403426.png)

2. 发布到http也是区分大小写

`ln -s /home/disk0 /var/www/html/disk0`

![小写找不到图片](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561193565519.png)

http://192.168.2.4/disk0/pic/2019/20190621/girl.jpg


## xfs ci版本不区分大小写

1. 新家一块测试盘

![新加测试盘](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561192463497.png)

2. 测试盘比较小直接fdisk分区即可。

![分区](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561192975096.png)

3. 格式化文件系统并挂载到/home/disk1

![格式化并挂载](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561193196913.png)

4. 复制文件到/home/disk1并查看

![linux下大小写不敏感](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561193959009.png)

5. 发布/home/disk1到http

`ln -s /home/disk1 /var/www/html/disk1`

![http中大小写不再敏感](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561194041507.png)