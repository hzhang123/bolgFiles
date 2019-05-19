---
title: RedHat7.4 yum配置 
tags: linux
---


----------

[toc]

# 1. yum配置

## 1.1 本地yum源配置

1. 设置使用ISO镜像软件：虚拟机 -> 设置

![本地ISO选择](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558270325530.png)

2. 此时设置本地ISO之后，在Linux挂载的文件为/dev/sr0

![sr0设备](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558270400301.png)

3. 将sr0挂载到/mnt/cdrom路径，如果此路径不存在，需要先创建。

``` shell
# 创建路径
mkdir -p /mnt/cdrom
# 一次性挂载
mount /dev/sr0 /mnt/cdrom
```

![挂载镜像](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558270680355.png)

4. ==永久挂载 #E91E63==，打开文件：/etc/fstab，并增加一行

![fstab文件](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558270888663.png)

从左向右依次为：

- 挂载的文件系统名称或UUID或LABEL
- 挂载点
- 文件系统
- 挂载选项：ro、noatime、async等
- dump选项，一般默认0
- fsck选项，一般默认0

5. 保存之后，命令行输入mount -a自动挂载fstab文件中的挂载项，之后每次重启会自动挂载。

6. 切换到 /etc/yum.repos.d/目录，如果存在文件全部备份，并创建一个yum文件.repo结尾，这里创建local.yum.repo，内容如下

``` shell
[local_yum]    # 括号中的名称为仓库源名称，通常为字母和数字，必须填写
name=local     # 对yum的描述，可写可不写
baseurl=file:///mnt/cdrom    # baseurl表示声明yum可以管理并使用的rpm包路径，必须填写
enabled=1            # enabled 表示当前仓库是否开启：1为开启，0为关闭，此项不写默认为开启
gpgcheck=0           # gpgcheck 表示安装rpm包时，是否基于公私钥对匹配包的安全信息：1表示开启， 0表示关闭，此项不写默认为验证
```

7. 运行清理命令并查询当前本地包
`yum clean all;yum list | wc -l;`

![清理与本地包](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558271691438.png)

## 1.2 配置网络yum源为CentOS源

==注： #E91E63==如果存在本地yum源最好先安装wget，方便下载rpm包到虚拟机下，也可以使用windows下载然后上传到linux内

1. 查找已经安装的yum依赖包

``` shell
# 查找已经安装的yum依赖包
rpm -qa | grep yum
```

2. 卸载安装的yum依赖包

``` shell
# 直接卸载已经安装的yum依赖包，不检查依赖
rpm -qa | grep yum | xargs rpm -e --nodeps
```

![卸载依赖包](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558272014137.png)

3. 下载依赖包，创建目录并下载对应文件：

``` shell
mkdir -p /tmp/yum; cd $_;
# 下载安装包
yum_list="yum-utils-1.1.31-50.el7.noarch.rpm 
yum-updateonboot-1.1.31-50.el7.noarch.rpm
yum-plugin-fastestmirror-1.1.31-50.el7.noarch.rpm
yum-metadata-parser-1.1.4-10.el7.x86_64.rpm 
yum-3.4.3-161.el7.centos.noarch.rpm 
python-kitchen-1.1.1-5.el7.noarch.rpm
python-chardet-2.2.1-1.el7_1.noarch.rpm";
for i in ${yum_list}; do wget http://mirrors.163.com/centos/7/os/x86_64/Packages/${i}; done;
```

4. 安装yum依赖包，单个安装可能会依赖报错，全部安装：rpm -ivh \* ，可能提示还会存在其他依赖，如果提示根据关键词到http://mirrors.163.com/centos/7/os或http://mirrors.aliyun.com/centos/7/os网址下载。

![依赖安装](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558274806365.png)

5. 配置/etc/yum.repos.d/下文件，与本地yum配置方法一致

``` shell
[base]
name= yum repo
baseurl=http://mirrors.aliyun.com/centos/7/os/$basearch/
enabled=1
gpgcheck=0
```

6. 配置完成之后查看一下rpm列表

![yum列表](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1558275243725.png)

7. 尝试安装一下vim：yum install vim -y