---
title: Hadoop-HA集群搭建-rehl7.4
tags: hadoop
---


----------

> 无说明需要登录其它机器操作，都是在集群的HD-2-101上执行的命令。
> 所有安装包地址：[百度网盘](https://pan.baidu.com/s/1LQzpKyIsJjlXFpPlAd4N9Q)，提取码：24oy

[toc]

# 1. 基础环境配置

## 1.1 克隆虚拟机

虚拟的安装与静态IP等配置见：[Linux传送门汇总](https://www.cnblogs.com/h-zhang/p/10887002.html)

1. 鼠标点击虚拟机，右键管理 -> 克隆

![点击克隆](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563969543992.png)

2. 点击下一步，克隆自当前状态或快照 -> 选择完整克隆。
3. 修改虚拟机名称，选择存储位置，点击完成。等待克隆完成。

![虚拟机存储选择](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563969756856.png)

4. 依照上面方法克隆所需台虚拟机。

## 1.2 修改静态IP

1. 点击开启所有虚拟机。
2. vmware进入虚拟机打开 ==/etc/sysconfig/network-scripts/ifcfg-ens33 #F44336== 修改静态IP字段 ==IPADDR==。
3. 重启网络服务：`systemctl restart network.service`

注：这里配置三台机器(192.168.2.101;192.168.2.102;192.168.2.103)

## 1.3 本机依赖安装

``` shell
# 先安装本机的依赖，其它没有的依赖后续再安装，目前我电脑现在需要的就这几个o_0
yum install tcl-devel.x86_64 rsync.x86_64 ntp.x86_64 -y
```

## 1.4 配置群改

shell脚本内容：
1. 配置当前机器到其余机器的信任
2. shell脚本修改主机名映射
2. shell分发hosts主机列表映射文件
3. 关闭防火墙
4. 关闭SELinux
5.  ==后续再添加其他功能 #00BCD4== ......

执行步骤：
1. 上传autoconfig.tar.gz到101机器home目录下解压，`tar -zxvf autoconfig.tar.gz -C /home`
2. 修改脚本host.list文件 ==/home/autoconfig/etc/host.list==

![host.list](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563971886473.png)

3. 修改脚本自带hosts文件 ==/home/autoconfig/file/hosts==，会分发到所有机器覆盖/etc/hosts，注意与host.list主机名映射不要出错。

![hosts](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563971973567.png)

4.切换到/home/autoconfig/bin目录执行：sh autoconfig.sh all
5.分发到所有机器执行。

``` shell
cd /home/autoconfig/bin;
sh xsync "/home/autoconfig" "/home";
sh doCommand other "cd /home/autoconfig/bin/; sh autoconfig.sh trust";
```

6. 重启所有机器

``` shell
sh doCommand other "init 0";
init 0;
```

## 1.5 安装依赖

``` shell
cd /home/autoconfig/bin;
sh doCommand all "yum install tcl-devel.x86_64 rsync.x86_64 ntp.x86_64 -y"
```

## 1.6 安装jdk

首先要检查所有机器是否安装java，并卸载

检查：`sh doCommand all "rpm -qa | grep java";`
卸载用:rpm -e --nodeps 要卸载的软件包

1. 所有机器创建目录/opt/cluster

``` shell
sh doCommand all "mkdir -p /opt/cluster";
```

2. jdk上传到101机器/opt/cluster下
3. 解压并分发到其余机器。

``` shell
tar -zxvf /opt/cluster/jdk-8u144-linux-x64.tar.gz;
sh xsync "/opt/cluster/jdk1.8.0_144" "/opt/cluster";
```

4. 创建连接

``` shell
# java版本是jdk1.8.0_144
sh doCommand all "ln -s /opt/cluster/>>==jdk1.8.0_144==<< /opt/cluster/java";
```

5. 添加到环境变量，所有机器/etc/profile追加以下内容
``` shell
#JAVA_HOME
export JAVA_HOME=/opt/cluster/java
export PATH=$PATH:$JAVA_HOME/bin
```

## 1.7 时间同步

注：这里选择HD-2-101为ntpd对时服务器

1. 修改/etc/ntp.conf文件

``` dsconfig
restrict 192.168.2.0 mask 255.255.255.0 nomodify notrap
restrict 127.0.0.1
# 注释掉以下，内网中不能使用外网的
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
# 修改当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步
server 127.127.1.0
fudge 127.127.1.0 stratum 5
```

2. 修改/etc/sysconfig/ntpd文件

``` dsconfig
# 增加内容如下（让硬件时间与系统时间一起同步）
SYNC_HWCLOCK="yes"
```

3. 增加定时任务，增加/etc/cron.d/ntp_crond文件，内容如下

``` dsconfig
*/10 * * * * root /usr/sbin/ntpdate HD-2-101
```
4. 分发文件定时文件

``` shell
sh xsync "/etc/cron.d/ntp_crond" "/etc/cron.d";
```

5. 重启crond服务

``` shell
# 重启
sh doCommand all "systemctl restart crond.service"
```

6. 检查对时服务器机器上状态，对时启动大约需要5分钟。

``` shell
# reach是已经向上层NTP服务器要求更新的次数，是一个八进制，每次改变是poll对应的秒数，等reach大于等于17其它服务器就可对本服务器对时了。
watch ntpq -p
```

![ntpd状态](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564194439010.png)

7. 手动第一次对时

``` shell
# 保证其他机器ntpd不开启
sh doCommand other "systemctl stop ntpd.service;/usr/sbin/ntpdate HD-2-101;"
```

# 2. 集群规划

![集群角色分布](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564195259848.png)


# 3. 配置Zookeeper集群


安装包下载地址：[zookeeper-3.4.14.tar.gz](http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz)

1. 解压zookeeper安装包到/opt/cluster，并解压

``` shell
tar -zxvf /opt/cluster/>>==zookeeper-3.4.14.tar.gz==<< -C /opt/cluster;
```
2. ==所有机器==创建zookeeper数据目录。

``` shell
sh doCommand all "mkdir -p /hdata/zookeeper;";
```

3. 复制conf目录下的zoo_sample.cfg为zoo.cfg。并修改dataDir内容如下，追加server内容：

``` dsconfig
dataDir=/hdata/zookeeper
server.1=HD-2-101:2888:3888
server.2=HD-2-102:2888:3888
server.3=HD-2-103:2888:3888

#server.A=B:C:D。
#A是一个数字，表示这个是第几号服务器；
#B是这个服务器的IP地址；
#C是这个服务器与集群中的Leader服务器交换信息的端口；
#D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选#举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
#集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。
```
4. 复制项目到其他机器上。

``` shell
sh xsync "/opt/cluster/zookeeper-3.4.14" "/opt/cluster";
```

5. 所有机器zookeeper数据目录下创建myid文件，文件中添加与server对应的编号。

``` shell
# 如server.1=B:C:D
echo >>=="1"==<< > /hdata/zookeeper/myid;
```

6. 创建软连接。

``` shell
sh doCommand all "ln -s >>==/opt/cluster/zookeeper-3.4.14==<< /opt/cluster/zookeeper";
```

7. 启动集群。

``` shell
sh doCommand all "source /etc/profile; /opt/cluster/zookeeper/bin/zkServer.sh start";
```

6. 检查状态

``` shell
sh doCommand all "source /etc/profile; /opt/cluster/zookeeper/bin/zkServer.sh status";
```

# 4. HDFS-HA 与YARN-HA 集群配置

## 4.1 修改env.sh配置

1. 切换到/opt/cluster/hadoop/etc/hadoop路径
2. hadoop-env.sh==修改 #F44336====export JAVA_HOME=/opt/cluster/java==
3. yarn-env.sh==增加 #F44336====export JAVA_HOME=/opt/cluster/java==
4. mapred-env.sh==增加 #F44336====export JAVA_HOME=/opt/cluster/java==

## 4.2 修改site.xml配置

1. 上传hadoop_template.tar.gz模版压缩包到/home目录下并解压：`tar -zxvf /home/hadoop_template.tar.gz -C /home`

2. 根据机器配置填写env.sh export 导出的变量值。
3. sh /home/hadoop_template/ha/env.sh运行脚本，自动完成配置。
4. ha模板路径/home/hadoop_template/ha，所有的模板文件配置如下：

### 4.2.1 core-site.xml.template

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- 把两个NameNode的地址组装成一个集群mycluster -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://${HADOOP_CLUSTER_NAME}</value>
    </property>

    <!-- 指定hadoop运行时产生文件的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>${HADOOP_TMP_DIR}</value>
    </property>

    <property>
        <name>ha.zookeeper.quorum</name>
        <value>${HADOOP_ZOOKEEPERS}</value>
    </property>
    <!-- 防止使用start-dfs.sh journalnode未启动NameNode连接不上journalnode无法启动 -->
    <property>
        <name>ipc.client.connect.max.retries</name>
        <value>100</value>
        <description>Indicates the number of retries a client will make to establisha server connection.
        </description>
    </property>
    <property>
        <name>ipc.client.connect.retry.interval</name>
        <value>10000</value>
        <description>Indicates the number of milliseconds a client will wait for before retrying to establish a server connection.
        </description>
    </property>
</configuration>
```

### 4.2.2 hdfs-site.xml.template

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <!-- 完全分布式集群名称 -->
    <property>
        <name>dfs.nameservices</name>
        <value>${HADOOP_CLUSTER_NAME}</value>
    </property>

    <!-- 集群中NameNode节点都有哪些 -->
    <property>
        <name>dfs.ha.namenodes.${HADOOP_CLUSTER_NAME}</name>
        <value>${HADOOP_NAME_NODES}</value>
    </property>

    <!-- nn1的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.${HADOOP_CLUSTER_NAME}.nn1</name>
        <value>${HADOOP_NN1}:9000</value>
    </property>

    <!-- nn2的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.${HADOOP_CLUSTER_NAME}.nn2</name>
        <value>${HADOOP_NN2}:9000</value>
    </property>

    <!-- nn1的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.${HADOOP_CLUSTER_NAME}.nn1</name>
        <value>${HADOOP_NN1}:50070</value>
    </property>

    <!-- nn2的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.${HADOOP_CLUSTER_NAME}.nn2</name>
        <value>${HADOOP_NN2}:50070</value>
    </property>

    <!-- 指定NameNode元数据在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
    <value>${HADOOP_JN}</value>
    </property>

    <!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>

    <!-- 使用隔离机制时需要ssh无秘钥登录-->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>${HADOOP_ISA_PATH}</value>
    </property>

    <!-- 声明journalnode服务器存储目录-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>${HADOOP_JN_DATA_DIR}</value>
    </property>

    <!-- 关闭权限检查-->
    <property>
        <name>dfs.permissions.enable</name>
        <value>false</value>
    </property>

    <!-- 访问代理类：client，mycluster，active配置失败自动切换实现方式-->
    <property>
        <name>dfs.client.failover.proxy.provider.${HADOOP_CLUSTER_NAME}</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>

    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
</configuration>

```

### 4.2.3 yarn-site.xml.template

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!--启用resourcemanager ha-->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
 
    <!--声明两台resourcemanager的地址-->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>${HADOOP_YARN_ID}</value>
    </property>

    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>${HADOOP_YARN_RMS}</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>${HADOOP_YARN_RM1}</value>
    </property>

    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>${HADOOP_YARN_RM2}</value>
    </property>
 
    <!--指定zookeeper集群的地址--> 
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>${HADOOP_ZOOKEEPERS}</value>
    </property>

    <!--启用自动恢复--> 
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>
 
    <!--指定resourcemanager的状态信息存储在zookeeper集群--> 
    <property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
</property>

</configuration>

```

### 4.2.4 mapred-site.xml.template

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

</configuration>
```

### 4.2.5 env.sh脚本

``` shell
#!/bin/bash
# hadoop安装目录
export HADOOP_HOME="/opt/cluster/hadoop-2.7.2"
#
# hadoop集群名称
export HADOOP_CLUSTER_NAME="myhadoop"
# hadoop运行时产生文件的存储目录
export HADOOP_TMP_DIR="/hdata/hadoop"
#
# 集群中所有NameNode节点
export HADOOP_NAME_NODES="nn1,nn2"
# 根据上面列出的NameNode配置所有NameNode节点地址，变量名称如HADOOP_NN1,HADOOP_NN2依次增加
export HADOOP_NN1="HD-2-101"
export HADOOP_NN2="HD-2-102"
# NameNode元数据在JournalNode上的存放位置
export HADOOP_JN="qjournal://HD-2-101:8485;HD-2-102:8485;HD-2-103:8485/myhadoop"
# id_rsa公钥地址
export HADOOP_ISA_PATH="~/.ssh/id_rsa"
# journalnode服务器存储目录
export HADOOP_JN_DATA_DIR="/hdata/hadoop/journal"
# zookeeper机器列表
export HADOOP_ZOOKEEPERS="HD-2-101:2181,HD-2-102:2181,HD-2-103:2181"
# yarn集群id
export HADOOP_YARN_ID="yarn-ha"
# 集群中所有的resourcemanager
export HADOOP_YARN_RMS="rm1,rm2"
# 根据上面列出的resourcemanager配置所有resourcemanager节点地址，变量名称如HADOOP_YARN_RM1,HADOOP_YARN_RM2依次增加
export HADOOP_YARN_RM1="HD-2-101"
export HADOOP_YARN_RM2="HD-2-102"

baseDir=$(cd `dirname $0`; pwd)
for template in `cd ${baseDir}; ls *template`
do
    siteFile=`echo ${template} | gawk -F"." '{print $1"."$2}'`
    envsubst < ${template} > ${HADOOP_HOME}/etc/hadoop/${siteFile}
    echo -e "#### set ${siteFile} succeed"
done

```

# 5. 集群启动

## 5.1 hdfs启动

1. 同步配置到其他机器

``` shell
# 同步
sh xsync "/opt/cluster/hadoop-2.7.2" "/opt/cluster";
# 建立软连接
sh doCommand all "ln -s /opt/cluster/hadoop-2.7.2 /opt/cluster/hadoop;";
```

2. 启动zk集群并初始化在ZK中的状态

``` shell
# 启动zk集群
sh doCommand all "source /etc/profile; /opt/cluster/zookeeper/bin/zkServer.sh start";
# 初始化在ZK中的状态
sh /opt/cluster/hadoop/bin/hdfs zkfc -formatZK
```

3. 启动journalnode

``` shell
sh doCommand all "sh /opt/cluster/hadoop/sbin/hadoop-daemon.sh start journalnode";
```

4. 登录==NameNode1 #E91E63==机器上格式化并启动

``` shell
# 格式化
sh /opt/cluster/hadoop/bin/hdfs namenode -format;
# 启动
sh /opt/cluster/hadoop/sbin/hadoop-daemon.sh start namenode;
```

5. 登录==NameNode2 #E91E63==机器同步nn1元数据，并启动

``` shell
# 同步元数据
sh /opt/cluster/hadoop/bin/hdfs namenode -bootstrapStandby;
# 启动NameNode2
sh /opt/cluster/hadoop/sbin/hadoop-daemon.sh start namenode;
```

6. Web界面显示NameNode信息

![HD-2-101](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564197498335.png)

![HD-2-102](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564197517848.png)


7. 重启所有dfs所有服务，除了zk

``` shell
sh /opt/cluster/hadoop/sbin/stop-dfs.sh
sh /opt/cluster/hadoop/sbin/start-dfs.sh
```

8. 检查所有机器NameNode状态

``` shell
# 检查状态
sh /opt/cluster/hadoop/bin/hdfs haadmin -getServiceState nn1;
sh /opt/cluster/hadoop/bin/hdfs haadmin -getServiceState nn2;
```

## 5.2 yarn启动

1. HD-2-101启动yarn

``` shell
sh /opt/cluster/hadoop/sbin/start-yarn.sh;
```

2. HD-2-103启动ResourceManager

``` shell
sh /opt/cluster/hadoop/sbin/yarn-daemon.sh start resourcemanager;
```

3. 查看ResourceManager服务状态

``` shell
sh /opt/cluster/hadoop/bin/yarn rmadmin -getServiceState rm1;
sh /opt/cluster/hadoop/bin/yarn rmadmin -getServiceState rm2;
```

4. 集群状态

![yarn HD-2-101状态](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564201456464.png)

![yarn HD-2-102状态](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564201502583.png)

# 6. 环境验证

1. 创建文件word.txt，内容如下：

``` txt
export	HADOOP_CLUSTER_NAME	myhadoop
export	HADOOP_TMP_DIR	hdata	hadoop
hdata	export
HADOOP_TMP_DIR	myhadoop	export
```


2. 创建文件到指定路径

``` shell
# 创建路径
/opt/cluster/hadoop/bin/hadoop fs -mkdir -p /mapreduce/test/input/20180702;
# 上传
/opt/cluster/hadoop/bin/hadoop fs -put ./word.txt /mapreduce/test/input/20180702;
```

3. 测试运行wordcount

``` shell
cd /opt/cluster/hadoop;
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /mapreduce/test/input/20180702 /mapreduce/test/output/20180702;
```

![控制台输出](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564203652990.png)

![yarn](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564203681790.png)

![mapreduce结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564203949803.png)