---
title: PostgreSQL数据库安装
tags: postgresql,linux,9.6.0
grammar_cjkRuby: true
---
2018年01月31日 10时53分13秒


----------

[TOC]

## 编译以及安装

### 源码编译


1） 官网下载源码安装包（本次源码安装包名：postgresql-9.6.0.tar.gz）。
2） 为了安装的管理，Linux机器一般需要先 **==创建 #F44336==** 三个目录：编译目录、数据库安装目录、数据初始化目录。
3） 这儿使用的目录如下：

|编译目录|数据库安装目录|数据存储目录|
| --- | --- |---|
| /usr/local/src/postgresql   |   /usr/local/pgsql9.6.0/  |  /data/pgdata  |
	
4）  将源码安装包上传到编译目录，并使用tar命令解压。

``` shell
tar -xvf postgresql-9.6.0.tar.gz
```
5） 安装postgresql依赖包

``` sehll
# 如果安装报错，再次运行一次
yum install -y perl-ExtUtils-Embed readline-devel zlib-devel pam-devel libxml2-devel libxslt-devel openldap-devel python-devel gcc-c++ openssl-devel cmake
```

**注：** redhat7.4配置yum源步骤见:[传送门](https://www.cnblogs.com/h-zhang/p/11068991.html)

6） 对源码进行编译，源码编译使用root用户即可

``` shell
# 进入解压目录
#对于9.X版本的默认线程安全，所以不用添加线程安全的选项了
#第一步就是使用configure命令
./configure --prefix=/usr/local/pgsql9.6.0 --with-perl --with-python
#第二步使用make，make的版本需要在3.8之上，版本查看：make --version
make
#第三步使用make install安装，需要root权限才能对/usr/local有写权限
make install
```
### 程序安装

1） 安装之后：建立软连接，方便后期升级维护

``` shell
ln -sf /usr/local/pgsql9.6.0/ /usr/local/pgsql
```


2） 设置可执行文件与共享库的路径：
如果是将语句加在.profile或.bash_profile文件中，界面登录是不会生效的。所以可以加在/etc/profile文件中

``` shell
#将postgresql自带命令路径添加到PATH
export PATH=/usr/local/pgsql/bin:$PATH
#设置共享库的路径
export LD_LIBRARY_PATH=/usr/local/pgsql/lib:$LD_LIBRARY_PATH
#设置数据存储的路径
export PGDATA=/data/pgdata/

#创建用户
useradd -U -p 123456 postgres
#如果这个目录不属于postgres用户和组，可以使用chown修改
chown postgres:postgres /data/pgdata/
#将所有的参数设置完成之后，将变量导出使其生效
source /etc/profile
```


3） 将依赖的环境变量导出之后，进行数据库初始化到数据存储目录之中

``` shell
#切换为postgres用户初始化数据库
su - postgres
#数据库的初始化，命令之后不指定PGDATA，默认使用环境变量PGDATA中存储的路径
initdb
```

4） 如果需要安装contrib下的工具可以到之前解压的主目录下的contrib目录运行下面的命令

``` shell
make;
sudo make install;
```

### 数据库的启动和停止

#### 启动数据库

``` shell
# 切换为postgres用户后启动
su - postgres
pg_ctl start -D $PGDATA
#PGDATA是前面导出的数据库的数据目录，也可以直接在命令后面加目录启动
```

#### 关闭数据库


``` shell
# 切换为postgres用户后停止
su - postgres
pg_ctl stop -D $PGDATA [ -m shutdown-mode ]

-m为停止方法：（3个参数）
		smart：等待所有连接中止之后
		fast：快速关闭，断开客户端连接，让已有事务回滚
		immediate：立即退出，下次进入需要修复
```

### 数据库开机自动启动

1） 开机自动启动，需要将解压路径下的contrib/start-scripts/linux文件添加执行权限， **==并且修改该文件中的PGDATA参数为自己的实际的路径 #F44336==**，prefix参数修改为前面数据库的安装目录，前面将数据库的安装目录添加了软连接，这儿修改为软连接的目录就可以了（强烈推荐使用前面软连接的做法，方便管理）

2） 之后将linux文件复制到/etc/init.d/下重命名为postgresql

``` shell
#添加为开机前启动
chkconfig --add postgresql
#对服务的管理命令
service postgresql {start|stop|restart|reload|status}
```

### 远程连接配置

需要配置两个文件，位于数据库数据目录（）
	postgresql.conf
	pg_hba.conf
	
![postgresql.conf][5]

![pg_hba.conf][6]

### 数据库的基本操作


``` sql
--创建用户
create user hzhang with password 'hzhang';
--赋权限
alter user hzhang with createdb;
--修改用户密码
alter user hzhang with password 'hzhang';
--创建一个数据库，数据库拥有者hzhang
create database moon owner hzhang;
```

### 安装过程中的报错

**基本都是安装依赖包的时候没有安装造成的**

报错1：configure: error: no acceptable C compiler found in $PATH

![][1]

解决方法：安装gcc套件
yum install gcc

报错2：configure: error: readline library not found

![][2]

解决方法：安装readline-devel
yum list | grep readline
yum install readline-devel.x86_64

报错3：configure: error: zlib library not found

![][3]

解决方法：安装zlib-devel
yum list | grep zlib
yum install zlib-devel.x86_64

报错4：configure: error: header file <Python.h> is required for Python

![][4]

解决方法
yum install python python-devel





  [1]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561262855478.png
  [2]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561262855484.png
  [3]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561262855485.png
  [4]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561262855485.png
  [5]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561262855486.png
  [6]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561268242459.png
  [7]: ./images/1534295732655.jpg