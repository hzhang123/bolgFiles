---
title: VMware虚拟机开机自启动
tags: linux
grammar_cjkRuby: true
---

2018年05月09日 08时30分18秒


----------
> VMware的命令行语句可以切换到VMware安装目录，使用vmrun.exe --help（windows下）查看。

[TOC]

# 编辑器bat注册文件
## 启动文件

 在系统的某个安静的盘中创建一个vm_start.bat文件，然后使用编辑器打开。写入
 
``` bat
 "D:\VMware\vmrun.exe" start "D:\VMware_workstation\RedHat\Red Hat Enterprise Linux 6 64 位.vmx" nogui
```
**注**：start之前的是VMware安装路径下的启动文件，start到nogui中间的是虚拟机的启动文件。
## 停止文件
再次创建一个vm_stop.bat文件，使用编辑器打开，写入

``` bat
 "D:\VMware\vmrun.exe" stop "D:\VMware_workstation\RedHat\Red Hat Enterprise Linux 6 64 位.vmx"
```

**注**：路径与vm_start.bat中的一样，有soft与hard两种关闭模式（遇到使用soft关闭无法关闭的情况）

## 测试运行文件
双击启动文件：vm_start.bat，如果弹出dos窗口且虚拟机启动则无误
双击停止文件：vm_stop.bat，如果弹出dos窗口且虚拟机停止则无误

# 添加运行脚本
==Windows键 + R== 调出运行界面 -> gpedit.msc -> 用户配置 -> windows设置 -> 鼠标双击脚本(登录/注销) -> 鼠标双击“登录”或“注销”分别添加

![注册注销/登陆界面][1]

 双击“添加” -> 分别添加刚才编辑的脚本(如下面启动脚本的添加)
 
 ![启动脚本添加][2]


  [1]: ./images/1525829227773.jpg
  [2]: ./images/1525829369196.jpg