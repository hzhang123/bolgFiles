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

==安装完成 #f44336==

## （三）maven setting.xml配置

 1. 配置本地仓库，创建目录 ==D:/development/maven/MavenRepository==，然后打开加目录下./conf/setting.xml文件，修改如下配置：

``` xml
<!-- 本地缓存 -->
<localRepository>D:/development/maven/MavenRepository</localRepository>
```

![本地库配置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556104937462.png)

 2. 配置镜像，国外镜像比较慢，配置国内镜像源，[阿里镜像源查询界面](https://maven.aliyun.com/mvn/search)

``` xml
<!-- 阿里库镜像 -->
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
```

![库镜像配置](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556106563244.png)

3. 用户setting.xml配置，创建==D:/development/maven/.m2==目录，windows下创建 ==.m2== 文件夹，键入 ==.m2. #F44336== 回车会自动去掉后面的点；然后将conf下setting.xml复制一份到.m2下。

4. cmd窗口运行==mvn help:system==

![运行截图](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556107574859.png)

最后会打印系统变量信息

#  二、 创建maven项目

## （一）创建项目

1. IDEA中创建新项目，选择Maven，点击==next==。

![创建空白maven项目](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556107703942.png)

2. 输入项目坐标信息，点击==next==。

![项目信息](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556107947190.png)

3. 设置项目信息，点击==Finish==。

![项目信息](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1556108137037.png)

## （二）IDEA中maven设置



