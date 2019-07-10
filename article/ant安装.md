---
title: ant安装(Windows)
tags: ant
---

> 下载之前参考一下官网的ant与java版本依赖
> 
----------



[toc]

## 1. 下载地址

[ant官网](http://ant.apache.org/)
[所有版本](https://www.apache.org/dist/ant/binaries/)

## 2. 解压与配置

java版本:1.8.0_191
ant版本:1.10.5

1. 解压apache-ant-1.10.5-bin.zip 
2. 新建变量ANT_HOME:==ant解压目录 #F44336==
3. Path环境变量追加
==;%ANT_HOME%\bin #F44336==
4. classpath环境变量追加:
==;%ANT_HOME%\lib #F44336==

**版本验证**: ==ant -version==