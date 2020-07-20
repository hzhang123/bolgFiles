---
title: jmeter源码环境(IDEA) 
tags: jmeter
---


----------

[toc]

## 1. 本地环境

Windows7
java版本:1.8.0_191
ant版本:1.10.5
git版本:2.19.1.windows.1

## 2. 下载源码

1. 下载压缩包:[地址](https://github.com/apache/jmeter/releases)
2. git获取最新版源码

``` shell
# git clone
git clone https://github.com/apache/jmeter.git
```
## 3. 下载依赖包

1. 切换到源码目录Shift + 鼠标右键:当前目录打开cmd命令行
2. 下载依赖包

``` shell
# 下载依赖包
ant download_jars
ant install
```

2. 修改 ==.classpath==与 ==.project==

``` dos
ren eclipse.classpath .classpath
ren eclipse.project .project
```

## 4. 导入IDEA

**导入工程**

1. 打开File | New | Project from Existing Sources...
2. 选择源码目录
3. 选择==Eclipse #F44336==
4. 根据需要配置(一般默认即可)，最后选择jdk版本，然后点击Finsh

**配置**

1. 打开File | Project Structure | Modules选择右侧jmeter工程与Source tab
2. 选择src目录，右键protocol鼠标右键标记为sources
3. 点击Apply

![protocol](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1562851762633.png)

## 5. 运行

1. Ctrl + N快捷键查询NewDriver(jmeter入口类)
2. 点击运行(会报错，运行报错之后再配置)
3. 点击Edit Configurations
4. VM options增加源码目录参数

``` shell
#-Djmeter.home=源码目录
-Djmeter.home=D:\development\jmeter-5_0
```

5. 再次运行

![Edit Configurations](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1562851981590.png)

![VM options](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1562852157567.png)

