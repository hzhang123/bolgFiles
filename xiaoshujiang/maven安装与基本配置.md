---
title: maven安装与基本配置 
tags: 
---


> 依赖：java环境，[JDK安装](111)

[toc]

# 一、 maven安装
## （一）下载maven

[下载地址](http://maven.apache.org/download.cgi)

![下载界面](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556102538826.png)

## （二）安装与环境变量设置

 1.  将下载后的文件解压到指定目录，==这里解压到D:\development\maven==

![解压后目录结构](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556103547787.png)

 2.  添加环境变量 M2_HOME：==D:\development\maven\apache-maven-3.6.0==
 3.  path变量中追加 ==%M2_HOME%\bin\;==
 4.  打开cmd窗口，输入 ==mvn -version==  验证

![验证结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556103810306.png)

==安装完成 #F44336==