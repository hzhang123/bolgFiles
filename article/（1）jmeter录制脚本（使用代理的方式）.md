---
title: （1）jmeter录制脚本（使用代理的方式） 
tags: jmeter
grammar_cjkRuby: true
---
2018年07月09日 17时27分24秒

> 很多APP使用badboy是无法录制的，这种情况下需要使用chrome或Firefox，如果能联网使用chrome的插件BlazeMeter录制导出会更方便，但是在不能联网的情况下，BlazeMeter无法导出脚本。这儿还可以选择使用代理的方法进行录制。

[toc]

代理的方式使用chrome与Firefox录制都是一样的，只不过配置代理的界面不同而已，下面使用Firefox演示，同时给出chrome的代理配置方式。

### 1. jmeter配置

 1. 添加线程组（这儿使用来保存录制脚本的）：测试计划 -> 鼠标右键 -> Threads -> 线程组（tearUp、tearDown与线程组的区别自行查阅），这儿将线程组的名字改为starsTest，线程先使用默认配置

![添加线程组][1]

 2. 添加HTTP请求：线程组(starsTest) -> 点击鼠标右键 -> 添加 -> sampler -> HTTP请求
 
![添加HTTP请求][2]

 3. 配置HTTP请求
 
 - 配置服务器名称或IP：待录制的机器IP
 - 端口号：录制界面使用的端口号
 - implementation与协议：协议的版本与协议名称

![配置HTTP请求][3]

 4. 添加HTTP代理服务器：点击工作台 -> 鼠标右键 -> 非测试原件 -> HTTP代理服务器

![添加HTTP代理服务器][4]

5. HTTP代理服务器配置
 - 代理端口：配置要监听的本地的端口，这儿使用8088
 - 目标控制器：前面添加的线程组，要将代码保存到这个线程组下
 - 包含模式与排除模式可以将录制过程中的.css、.png文件做包含于排除等定制，排除一部分影响代码阅读的，如果是压力测试，为了贴合实际情况，一般不排除这些文件。
 - 启动按钮：在所有配置完成，需要来时进行录制的时候进行启动。==配置代理之后会启动然后开始录制，如果只配置了代理，不启动这儿的HTTP代理服务器浏览器是无法上网的==

![HTTP代理服务器配置][5]

![排除文件][6]

### 2. 代理配置
1. Firefox的代理配置：选项 -> 网络代理 -> 设置 -> 选中手动设置代理，IP地址填写localhost或127.0.0.1，端口为前面HTTP代理服务器配置的8088。

![Firefox浏览器代理配置][7]

2. chrome的代理配置：设置 -> 显示高级设置 -> 更改服务器代理设置 -> Internet属性 -> 连接  -> 局域网设置 -> 代理服务器 -> 选中为LAN使用代理服务器，IP地址填写localhost或127.0.0.1，端口为前面HTTP代理服务器配置的8088。

![chrome浏览器代理配置][8]

### 3. 开始录制
1. 启动代理服务

![启动代理][9]

2. 使用Firefox代开录制的网页，进行操作

![录制到的代码][10]

3. 添加监听器结果树：运行一遍，可以看到回访成功

![回放结果][11]

注：在录制完成之后需要参考==代理配置==中将浏览器设置为不使用代理，不然在不开启代理配置的情况下浏览器是无法上网的。

![不使用代理][12]


  [1]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112696.png
  [2]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112697.png
  [3]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112698.png
  [4]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112661.png
  [5]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112698.png
  [6]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112476.png
  [7]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112803.png
  [8]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112888.png
  [9]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112805.png
  [10]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112975.png
  [11]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112980.png
  [12]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563797112695.png