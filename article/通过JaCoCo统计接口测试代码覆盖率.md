---
title: 通过JaCoCo统计接口测试代码覆盖率
tags: jacoco
grammar_cjkRuby: true
---


> 需求：统计微服务接口测试的代码覆盖率
> 1. jacoco的ant与maven方法都是在编译期对单元测试的覆盖率统计
> 2. jacoco的可以开启一个agent服务收集运行过程中的代码执行覆盖率。
> 主要会用到jacoco 的两个功能：agent和cli

[toc]

## 覆盖率收集

### 1. 收集方式

鉴于接口测试是在微服务启动后运行的测试，所以在选用第二种agent的测试，会有两个比较麻烦的地方（列出了自己比较笨拙的解决方法）
1. 源码获取：去git上再拉取一遍。
2. 编译后字节码获取：按照测试环境构建再统计过程中再构建一遍。

统计覆盖率过程基本如：
1. 微服务启动，同时启动agent收集覆盖率
2. 接下来基本都是jenkins任务需要做的
 - git 拉取源码
 - 编译源码
 - 获取jacoco覆盖率统计dump文件
 - 生成报告

### jacoco使用

参考jacoco官方使用文档：[官方文档索引](https://www.jacoco.org/jacoco/trunk/doc/integrations.html)、[agent帮助文档](https://www.jacoco.org/jacoco/trunk/doc/agent.html)、[cli帮助文档](https://www.jacoco.org/jacoco/trunk/doc/cli.html)

- **jacoco官方给出了3种收集覆盖率文件的方式**：file、tcpserver、tcpclient。

在jvm虚拟机启动时加入参数： ==-javaagent:[yourpath/]jacocoagent.jar=[option1]=[value1],[option2]=[value2] #E91E63== ， 实例如下：

``` shell
# Example: 输出到文件，在JVM终止时，执行数据被写入本地文件。接口测试不会考虑这种情况（单测一般是用这种）
# 1. 服务终止影响整个测试环境。
# 2. 有些跨环境的服务，获取微服务所在机器上的文件也比较麻烦。
-javaagent:~/jacoco-0.8.5/lib/jacocoagent.jar=includes=*,output=file,append=true,destfile=~/jacoco-0.8.5/jacoco.exec

# Example: 输出到tcpserver，使用工具连接到JVM，获取dump数据。使用这种方式不需要停止jvm，也直接可以通过网络传输。但是远程连接没有任何身份验证机制，所以生产环境一定要确保只有信任的人可访问jacoco地址。配合jacoco cli获取数据。
-javaagent:~/jacoco-0.8.5/lib/jacocoagent.jar=includes=*,output=tcpserver,append=true,address=127.0.0.1,port=6301

# 列出部分参数，
# includes：插桩的代码类名列表，使用:分割，也可以使用*和?匹配，但是不考虑性能的情况下一般不需要使用。默认为*
# output：用于写入coverage数据的输出方法。有效选项包括：file、tcpserver、tcpclient、none
# append：如果文件已经存在则追加到已存在文件中，如果为false则替换
# destfile：输出exec文件的路径
# address：配合tcpserver来指定对外开放的jacoco访问地址。如果配置的127.0.0.1或localhost则只能本地访问dump数据
# port：配合tcpserver来指定对外开放的jacoco端口。端口不能被占用
```

- **jacoco命令行界面**：命令行界面提供了基本的操作，基本能满足接口覆盖率报告的生成；dump数据与生成报告都使用cli。

``` shell
# Example：获取jacoco server对外开放地址的数据。
# --address jacoco tcpserver地址，网络要通不然啥都白搭。
# --port jacoco tcpserver端口
# --destfile dump数据存储位置
java -jar ${jacoco_home}/lib/jacococli.jar >>==dump==<< --address ${address} --port ${port} --destfile ${destfile}

# Example：使用获取到的exec文件生成覆盖率报告。
# --classfiles 必须指定，源码编译后target目录文件。（这也是jenkins任务在单独拉取源码执行编译的原因）
# --sourcefiles 源码，非必须项。不指定无法查看代码执行详细情况。
# --html html报告生成目录
java -jar ${jacoco_home}/lib/jacococli.jar >>==report==<< ${destfile} --classfiles ${classfiles} --sourcefiles ${sourcefiles} --html reportdir
```

==接下来创建一个项目实验一下==

## Sprint boot测试项目

### 1. 创建项目

![创建Spring boot项目](./images/1578030897269.png)

![项目信息](./images/1578031015244.png)

![Sprint Web](./images/1578031050175.png)

### 2. 工程结构

``` feature
src
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           └── demo/
│   │               ├── JacocoDemoApplication.java
│   │               └── controller/
│   │                   └── CountController.java >>==测试controller==<<
│   └── resources/
│       ├── application.properties
│       ├── static/
│       └── templates/
└── test/
    └── java/
        └── com/
            └── example/
                └── demo/
                    └── JacocoDemoApplicationTests.java
```

### 3. CountController.java

``` java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
public class CountController {

    @RequestMapping(value = "/test1", method = RequestMethod.POST)
    @ResponseBody
    public boolean caseCount(@RequestBody Map<String, Integer> params) {
        if (params.get("count") > 0) {
            return true;
        } else {
            return false;
        }
    }

    @RequestMapping(value = "/test2", method = RequestMethod.POST)
    @ResponseBody
    public boolean caseCount1(@RequestBody Map<String, Integer> params) {
        if (params.get("count") > 0) {
            return true;
        } else {
            return false;
        }
    }
}

```

### 4. 上传代码到github

![创建项目](./images/1578119396431.png)

![仓库地址](./images/1578119495246.png)

``` shell
# 进入项目目录，初始化项目
git init
# 新增修改文件。
git add .
# 按照上一步提示使用命令设置本项目的账户（也可以加--global设置全局）
git config --local user.email '***'
# 提交本次新增内容
git commit -m 'jacoco 代理统计覆盖率demo代码'
# 设置代码库地址
git remote add origin https://github.com/hzhang123/jacoco-demo.git
# 获取项目初始化的README.md文件
git pull --rebase origin master
# 上传到github。 >>==可能会让输入用户名密码==<<
git push -u origin master
```

## 覆盖率统计测试

我这里全部是本地项目，所以都用的本地127.0.0.1

### 1. 启动项目

- 添加参数启动项目

![JVM启动参数加入tcpserver启动收集方式的参数](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579249812837.png)

- postman 发送测试请求

![test1接口<0分支](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579249994106.png)

![test2接口>0分支](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579250018568.png)


### 2. jenkins任务

- 添加流水线任务

![JacocoDemo](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579250291288.png)

- 添加string参数

![参数](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579250433006.png)

- Pipeline script

``` groovy
#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        // 如果scala构建使用sbt，jenkins兼容不太好需要environment中拼接工具地址
        SBT_HOME = tool name: 'sbt1.3.0', type: 'org.jvnet.hudson.plugins.SbtPluginBuilder$SbtInstallation'
        PATH = "${env.SBT_HOME}/bin:${env.PATH}"
        CODE_REPO = "https://github.com/hzhang123/jacoco-demo.git"
    }
    tools {
        // 引入tools中配置的工具
        maven "maven3.6.1"
        jdk 'jdk1.8.0_231'
    }
    stages {
        stage('Clone & Build') {
            steps {
                deleteDir()
                git branch: env.GIT_BRANCH, url: CODE_REPO
                // 如果有Phabricator & Arcanist工具可以配合review与打patch
                // script {
                //     diffs = DIFFS.trim().toUpperCase().split("(\\s+|\\s*,\\s*)")
                //     for (i = 0; i < diffs.length; i++) {
                //         id = diffs[i];
                //         if (id.trim().length() > 0) {
                //             sh "arc patch ${id}"
                //             sh "git checkout ${GIT_BRANCH}"
                //             sh "git merge arcpatch-${id}"
                //         }
                //     }
                // }
                sh "mvn clean package"
            }
        }
        stage('Exec & Report') {
            steps {
                sh '''
                # jacoco home
                # jacoco_home="/Users/growingio/developments/tools/jacoco-0.8.5"
                # ----------------
                # dump tcp端口数据
                # ----------------
                # --address IP地址
                # --port 端口
                # --destfile 保存文件名称
                #
                address=`echo ${server_addr} | awk -F: '{print $1}'`
                port=`echo ${server_addr} | awk -F: '{print $2}'`
                destfile=target/JacocoDemo.exec
                java -jar ${jacoco_home}/lib/jacococli.jar dump --address ${address} --port ${port} --destfile ${destfile}

                # 生成报告
                classfiles="target/scala-2.12/classes"
                sourcefiles="app"
                java -jar ${jacoco_home}/lib/jacococli.jar report ${destfile} --classfiles ${classfiles} --sourcefiles ${sourcefiles} --html report
                '''
            }
        }
        stage('publishHTML & Clean Workspace') {
            steps {
                publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: false, reportDir: "report", reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
                cleanWs()
            }
        }
    }
}
```

