---
title:  （一）mongodb安装
tags: mongodb
grammar_cjkRuby: true
---
2018年03月06日 15时53分29秒


----------
[TOC]

### 安装准备
1、上传安装文件到/usr/local目录之后解压文件，cd /usr/local && tar -zxvf mongodb-linux-x86_64-3.2.9.tgz
2、建立软连接：cd /usr/local && ln -s mongodb-linux-x86_64-3.2.9 mongodb
3、建立数据目录、配置文件、日志文件
mkdir -p /data/mongodata;
mkdir -p /usr/local/mongodb/etc /usr/local/mongodb/log;
touch /usr/local/mongodb/etc/mongodb.conf /usr/local/mongodb/log/mongodb.log;

### 启动mongodb

> 启动mongodb有两种方式

#### 1、命令行参数启动

cd /usr/local/mongodb/bin && ./mongod --dbpath=/data/mongodata --logpath=/usr/local/mongodb/log/mongodb.log --logappend --port=27017 --fork

#### 2、从配置文件启动

cd /usr/local/mongodb/bin && ./mongod --config /usr/local/mongodb/etc/mongodb.conf

配置文件的详细参数见：[==附录一：配置文件详解 #80006e==](#jump1)


### 使用服务启动mongodb

使用系统服务命令启动需要先将服务加入到系统服务中，如何将mongo添加到系统服务见：[==附录二 ：添加mongodb到系统服务 #80006e==](#jump2)

service mongod start|stop|restart

==**注**==：可以将mongodb加入系统路径变量中，这样可以不用输入路径直接启动，代码如下：

echo "export PATH=/usr/local/mongodb/bin:\$PATH" >> ~/.bash_profile && source ~/.bash_profile

之后root用户遍可以在任意目录运行mongodb的命令了

### 参数解释

> 不需要完全记住，知道有这么个东西就行


--dbpath 数据库路径(数据文件)
--logpath 日志文件路径
--master 指定为主机器
--slave 指定为从机器
--source 指定主机器的IP地址
--pologSize 指定日志文件大小不超过64M.因为resync是非常操作量大且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的 oplog大小是空闲磁盘大小的5%)。
--logappend 日志文件末尾添加
--port 启用端口号
--fork 在后台运行
--only 指定只复制哪一个数据库
--slavedelay 指从复制检测的时间间隔
--auth 是否需要验证权限登录(用户名和密码)
--config 配置文件位置

### 用户授权和管理

> 上面的配置文件中并没有打开权限验证，所以现在登录是不需要验证的

1. 前面已经配置了PATH，直接运行命令进入命令行：mongo
2. 连接到admin用户：use admin
3. 创建admin用户：db.createUser({user: "admin",pwd: "123456",roles: [ {role: "userAdminAnyDatabase", db: "admin"} ]});
4. 两个查看当前系统中的用户的方法：show users或者db.system.users.find();
5. 修改配置文件中的参数打开认证登录：auth=true
6. 重启mongo服务：service mongod restart
7. 此时登录之后是不能进行操作了，需要认证：use admin;
8. 认证（返回1，证明认证成功）：db.auth("admin","123456");
9. 认证之后是不能做什么的，需要创建数据库用户与数据库，创建数据库：use mydb;
10. 创建用户：db.createUser({user: "myuser",pwd: "mypwd",roles: [{ role: "readWrite", db: "mydb" }]});
11. 退出mongo之后使用刚刚创建的角色登录（前面的ip可以使任何安装有mongo的ip，这样便可以连接任何一台机器上的mongo）：mongo 127.0.0.1/mydb -u myuser -p mypwd

### 附录一：配置文件详解

————————配置文件————————
<span id="jump1"></span>

----------

``` stylus
#启用日志文件，默认启用

journal=true

#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false

quiet=false

# 日志文件位置

logpath=/usr/local/mongodb/log/mongodb.log

# 以追加方式写入日志

logappend=true

# 是否以守护进程方式运行

fork = true

# 默认27017

port = 27017

# 数据库文件位置

dbpath=/data/mongodata

# 启用定期记录CPU利用率和 I/O 等待

#cpu = true

# 是否以安全认证方式运行，默认是不认证的非安全方式

#auth = true

#noauth = true



# 详细记录输出

#verbose = true

# Inspect all client data for validity on receipt (useful for

# developing drivers)用于开发驱动程序时验证客户端请求

#objcheck = true

# Enable db quota management

# 启用数据库配额管理

#quota = true

# 设置oplog记录等级

# Set oplogging level where n is

#   0=off (default)

#   1=W

#   2=R

#   3=both

#   7=W+some reads

#diaglog=0

# Diagnostic/debugging option 动态调试项

#nocursors = true

# Ignore query hints 忽略查询提示

#nohints = true

# 禁用http界面，默认为localhost：28017

#nohttpinterface = true

# 关闭服务器端脚本，这将极大的限制功能

# Turns off server-side scripting.  This will result in greatly limited

# functionality

#noscripting = true

# 关闭扫描表，任何查询将会是扫描失败

# Turns off table scans.  Any query that would do a table scan fails.

#notablescan = true

# 关闭数据文件预分配

# Disable data file preallocation.

#noprealloc = true

# 为新数据库指定.ns文件的大小，单位:MB

# Specify .ns file size for new databases.

# nssize = 

# Replication Options 复制选项

# in replicated mongo databases, specify the replica set name here

#replSet=setname

# maximum size in megabytes for replication operation log

#oplogSize=1024

# path to a key file storing authentication info for connections

# between replica set members

#指定存储身份验证信息的密钥文件的路径

#keyFile=/path/to/keyfile
```

————————配置文件详解————————

``` text
MongoDB各配置参数详细说明：

1、verbose：

日志信息冗余。默认false。提高内部报告标准输出或记录到logpath配置的日志文件中。要启用verbose或启用verbosity 用vvvv参数，

如：verbose = true



2.vvvv = true

ps：启动verbose冗长信息，它的级别有 vv~vvvvv，v越多级别越高，在日志文件中记录的信息越详细。



3、port：

端口。默认27017，MongoDB的默认服务TCP端口，监听客户端连接。要是端口设置小于1024，比如1021，则需要root权限启动，不能用 mongodb帐号启动，（普通帐号即使是27017也起不来）否则报错：[mongo --port=1021 连接]

ERROR: listen(): bind() failed errno:13 Permission denied for socket: 127.0.0.1:1021

如：port = 27017



4、bind_ip：

绑定地址。默认127.0.0.1，只能通过本地连接。进程绑定和监听来自这个地址上的应用连接。要是需要给其他服务器连接，则需要注释掉这个或则 把IP改成本机地址，如192.168.200.201[其他服务器用 mongo --host=192.168.200.201 连接] ，可以用一个逗号分隔的列表绑定多个IP地址。

如：bind_ip = 127.0.0.1



5、maxConns：

最大连接数。默认值：取决于系统（即的ulimit和文件描述符）限制。MongoDB中不会限制其自身的连接。当设置大于系统的限制，则无效，以系 统限制为准。这对于客户端创建很多“表”，允许连接超时而不关闭“表”的时候很有用。设置该值的高于连接池和总连接数的大小，以防止尖峰时 候的连接。注意：不能设置该值大于20000。

如：maxConns = 100



6、objcheck:

强制验证客户端请求。2.4的默认设置为objcheck成为true，在早期版本objcheck默认为false。因为它强制验证客户端请求，确保客户端绝不插入无 效文件到数据库中。对于嵌套文档的对象，会有一点性能影响。设置noobjcheck 关闭。

如：objcheck = true

7、noobjcheck：

同上，默认关闭false。

如：noobjcheck = false



8、logpath：

指定日志文件，该文件将保存所有的日志记录、诊断信息。除非另有指定，mongod将所有的日志信息输出到标准输出。如果没有指定logappend， 重启则日志会进行覆盖操作。

如：logpath=/var/log/mongodb/mongodb.log



9、logappend：写日志的模式：设置为true为追加。默认是覆盖。如果未指定此设置，启动时MongoDB的将覆盖现有的日志文件。

如：logappend=true



10、syslog：日志输出都发送到主机的syslog系统，而不是标准输出到logpath指定日志文件。syslog和logpath不能一起用，会报错：Cant use both a logpath and syslog

如：syslog  = true



11、pidfilepath：

进程ID，没有指定则启动时候就没有PID文件。默认缺省。

如：pidfilepath = /var/run/mongo.pid



12、keyFile：
指定存储身份验证信息的密钥文件的路径。默认缺省。详情见："
word-spacing: 0px; display: inline; white-space: normal; orphans: 2; widows: 2; font-size-adjust: none; font-stretch: normal; background- color: #ffffff; -webkit-text-size-adjust: auto; -webkit-text-stroke-width: 0px;">Replica Set Security" and “Replica Set Administration.”
如：.keyFile = /srv/mongodb/keyfile


13、nounixsocket：

套接字文件，默认为false，有生成socket文件。当设置为true时，不会生成socket文件。
如：nounixsocket = false

unixSocketPrefix：套接字文件路径，默认/tmp

如：unixSocketPrefix = /tmp



14、fork：

是否后台运行，设置为true 启动 进程在后台运行的守护进程模式。默认false。

如：fork = true



15、auth：

用户认证，默认false。不需要认证。当设置为true时候，进入数据库需要auth验证，当数据库里没有用户，则不需要验证也可以操作。直到创建了第一个用户，之后操作都需要验证。

比如：通过db.addUser('sa','sa')  在admin库下面创建一个超级用户，只能在在admin库下面先认证完毕了：ab.auth('sa','sa') ，才能去别的库操作，不能在其他库验证。这样连接数据库也需要指定库：

1.mongo -usa -psa admin     #sa 帐号连接admin

1.mongo -uaa -paa test      #aa 帐号连接test

如：auth = true



16、noauth：

禁止用户认证，默认true。同上

如：noauth = true



17、cpu：

设置为true会强制mongodb每4s报告cpu利用率和io等待，把日志信息写到标准输出或日志文件。默认为false。

开启日志会出现：

1.Mon Jun 10 10:21:42.241 [snapshotthread] cpu: elapsed:4000  writelock: 0%

如：cpu = true



18、dbpath：

数据存放目录。默认： word-spacing: 0px; white-space: normal; orphans: 2; widows: 2; font-size-adjust: none; font-stretch: normal; background-color: #ffffff; -webkit-text-size-adjust: auto; -webkit-text-stroke-width: 0px;">/data/db/

如：dbpath=/var/lib/mongodb



19、diaglog：

创建一个非常详细的故障排除和各种错误的诊断日志记录。默认0。设置为1，为在dbpath目录里生成一个diaglog.开头的日志文件，他的值如下：

1.Value    Setting
2.0    off. No logging.       #关闭。没有记录。
3.1    Log write operations.  #写操作
4.2    Log read operations.   #读操作
5.3    Log both read and write operations. #读写操作
6.7    Log write and some read operations. #写和一些读操作

设置不等于0，日志会每分钟flush 一次：

1.Mon Jun 10 11:16:17.504 [DataFileSync] flushing diag log
2.Mon Jun 10 11:17:17.442 [DataFileSync] flushing diag log

产生的日志可以用mongosniff 来查看：要是mongosniff[类似于tcpdump的作为一个MongoDB的特定的TCP/ IP网络流量]出现报错和具体用法，请见这里，之前先执行：apt-get install libpcap-dev

1.root@m3:/var/lib/mongodb# mongosniff --source DIAGLOG diaglog.51b542a9

注意：当重新设置成0，会停止写入文件，但mongod还是继续保持打开该文件，即使它不再写入数据文件。如果你想重命名，移动或删除诊断日志，你必须完全关闭mongod实例。

如：diaglog = 3



20、directoryperdb：

设置为true，修改数据目录存储模式，每个数据库的文件存储在DBPATH指定目录的不同的文件夹中。使用此选项，可以配置的MongoDB将数据存储在不同的磁盘设备上，以提高写入吞吐量或磁盘容量。默认为false。
注意：要是在运行一段时间的数据库中，开启该参数，会导致原始的数据都会消失（注释参数则会回来）。因为数据目录都不同了，除非迁移现有的数据文件到directoryperdb产生的数据库目录中，如：
root@m3:/var/lib/mongodb# mv test.* test/
把test数据文件迁移到directoryperdb产生的数据库test目录中。 所以需要在规划好之后确定是否要开启。

01.原始数据结构：
02.journal
03.mongod.lock
04.local.0
05.local.1
06.local.ns
07.test.0
08.test.1
09.test.ns
10.
11.开启 directoryperdb，并把数据文件迁移到相关的数据目录后的结构：
12.
13.journal
14.mongod.lock
15.local/local.0
16.local/local.1
17.local/local.ns
18.test/test.0
19.test/test.1
20.test/test.ns

如：directoryperdb = ture



21、journal：

日志，（redo log，更多的介绍请看这里和这里）
默认值：（在64位系统）true。
默认值：（32位系统）false。
设置为true，启用操作日志，以确保写入持久性和数据的一致性，会在dbpath目录下创建journal目录。
设置为false，以防止日志持久性的情况下，并不需要开销。为了减少磁盘上使用的日志的影响，您可以启用nojournal，并设置为true。
注意：在64位系统上禁用日志必须使用带有nojournal的。

1.#journal=true
2.journal=false

32位OS：

1.Tue Jun 11 12:17:09.628 [initandlisten] ** NOTE: This is a 32 bit MongoDB binary.
2.Tue Jun 11 12:17:09.628 [initandlisten] **       32 bit builds are limited to less than 2GB of data (or less with --journal).

64位OS：

1.Tue Jun 11 12:29:34 [initandlisten] journal dir=/var/lib/mongodb/journal
2.Tue Jun 11 12:29:34 [initandlisten] recover : no journal files present, no recovery needed



22、nojournal:

禁止日志

默认值：（在64位系统）false。
默认值：（32位系统）true。
设置nojournal为true关闭日志，64位，2.0版本后的mongodb默认是启用 journal日志。

如：nojournal=true



23、journalCommitInterval：

刷写提交机制，默认是30ms或则100ms。较低的值，会更消耗磁盘的性能。
此选项接受2和300毫秒之间的值：
如果单块设备提供日志和数据文件，默认的日记提交时间间隔为100毫秒。
如果不同的块设备提供的日志和数据文件，默认的日记提交的时间间隔为30毫秒。

如：journalCommitInterval=100



24、ipv6：

是否支持ipv6，默认false。



25、jsonp：

是否允许JSONP访问通过一个HTTP接口，默认false。



26、nohttpinterface：

是否禁止http接口，即28017 端口开启的服务。默认false，支持。

如：nohttpinterface = false



27、noprealloc：

预分配方式。
默认false：使用预分配方式来保证写入性能的稳定，预分配在后台进行，并且每个预分配的文件都用0进行填充。这会让MongoDB始终保持额外的空间和空余的数据文件，从而避免了数据增长过快而带来的分配磁盘空间引起的阻塞。
设置noprealloc= true来禁用预分配的数据文件，会缩短启动时间，但在正常操作过程中，可能会导致性能显著下降。

如：noprealloc = false



28、noscripting：

是否禁止脚本引擎。默认是false：不禁止。ture：禁止
要是设置成true：运行一些脚本的时候会出现：

1.JavaScript execution failed: group command failed: { "ok" : 0, "errmsg" : "server-side JavaScript execution is disabled" }

1.#noscripting = true     <====> noscripting = false

notablescan：是否禁止表扫描操作。默认false：不禁止，ture：禁止

禁止要是执行表扫描会出现：

1.error: { "$err" : "table scans not allowed:test.emp", "code" : 10111 }

可以动态修改设置：

1.db.adminCommand({setParameter:1, notablescan:false})

1.#notablescan = true  <====> notablescan = false



29、nssize:

命名空间的文件（即NS）的默认大小，默认16M，最大2G。
所有新创建的默认大小命名空间的文件（即NS）。此选项不会影响现有的命名空间的文件的大小。默认值是16M字节，最大大小为2 GB。让小数据库不让浪费太多的磁盘空间，同时让大数据在磁盘上有连续的空间。

1.-rwxrwxrwx 1 mongodb zhoujy  16M  6月 11 14:44 test.0
2.-rwxrwxrwx 1 mongodb zhoujy  32M  6月  1 21:36 test.1
3.-rwxrwxrwx 1 mongodb zhoujy  16M  6月 11 14:44 test.ns
4.drwxr-xr-x 2 root    root   4.0K  6月 10 11:57 _tmp

如：nssize  = 16



30、profile：

数据库分析等级设置。记录一些操作性能到标准输出或则指定的logpath的日志文件中，默认0:关闭。

1.级别 设置
2.0 关。无分析。
3.1 开。仅包括慢操作。
4.2 开。包括所有操作。

控制 Profiling  的开关和级别：2种
第一种是直接在启动参数里直接进行设置或则启动MongoDB时加上–profile=级别，其信息保存在 生成的system.profile 中。

如：profile = 2

第二种是在客户端用db.setProfilingLevel(级别)命令来实时配置，其信息保存在 生成的system.profile 中。

1.[initandlisten] creating profile collection: local.system.profile

1.> db.setProfilingLevel(2)
2.{ "was" : 0, "slowms" : 100, "ok" : 1 }
3.> db.getProfilingStatus()
4.{ "was" : 2, "slowms" : 100 }

默认情况下，mongod的禁用分析。数据库分析可以影响数据库的性能，因为分析器必须记录和处理所有的数据库操作。所以在需要的时候用动态修改就可以了。



31、slowms：

记录profile分析的慢查询的时间，默认是100毫秒。具体同上。

1.slowms  = 200

1.> db.getProfilingStatus()
2.{ "was" : 2, "slowms" : 200 }



32、quota：

配额，默认false。是否开启配置每个数据库的最多文件数的限制。当为true则用quotaFiles来配置最多文件的数量。

如：quota = true



33、quotaFiles：

配额数量。每个数据库的数据文件数量的限制。此选项需要quota为true。默认为8。

如：quotaFiles = 8



34、rest：

默认false，设置为true，使一个简单的 REST API。

如：rest = true

设置为true，开启后，在MongoDB默认会开启一个HTTP协议的端口提供REST的服务（nohttpinterface = false），这个端口是你Server端口加上1000，即28017，默认的HTTP端口是数据库状态页面，（开启后，web页面的Commands 行中的命令都可以点进去）。mongodb自带的REST，不支持 增、删、改，同时也不支持 权限认证。
详细信息见这里和这里。


35、repair：

修复数据库操作，默认是false。
设置为true时，启动后修复所有数据库，设置这个选项最好在命令行上，而不是在配置文件或控制脚本。如：
命令行修复：

1.> db.repairDatabase('xxx')
2.{ "ok" : 1 }
3.> db.repairDatabase()
4.{ "ok" : 1 }

启动时修复：

1.repair = true

1.root@m3:/var/log/mongodb# mongod --repair

启动时修复，需要关闭journal，否则报错:

1.Can't specify both --journal and --repair options.

并且启动时，用控制文件指定参数和配置文件里指定参数的方式进行修复之后，（修复信息见log），需要再禁用repair参数才能启用mongodb。
注意：mongod修复时，需要重写所有的数据库文件。如果在同一个帐号下不能运行修复，则需要运行chown修改数据库文件的权限。

repairpath：修复路径，默认是在dbpath路径下的_tmp 目录。

1.drwxr-xr-x 2 root    root   4.0K  6月 11 20:23 _tmp



36、smallfiles：

是否使用较小的默认文件。默认为false，不使用。
设置为true，使用较小的默认数据文件大小。smallfiles减少数据文件的初始大小，并限制他们到512M，也减少了日志文件的大小，并限制他们到128M。
如果数据库很大，各持有少量的数据，会导致mongodb创建很多文件，会影响性能。

如：smallfiles = true



37、syncdelay：

刷写数据到日志的频率，通过fsync操作数据。默认60秒。

如：syncdelay = 60

默认就可以，不需要设置。不会对日志文件（journal files）有影响

警告：如果设置为0，SYNCDELAY 不会同步到磁盘的内存映射文件。在生产系统上，不要设置这个值。



38、sysinfo：

系统信息，默认false。

设置为true，mongod会诊断系统有关的页面大小，数量的物理页面，可用物理??页面的数量输出到标准输出。

1.Tue Jun 11 21:07:15.031 sysinfo:
2.Tue Jun 11 21:07:15.035   page size: 4096
3.Tue Jun 11 21:07:15.035   _SC_PHYS_PAGES: 256318
4.Tue Jun 11 21:07:15.035   _SC_AVPHYS_PAGES: 19895

当开启sysinfo参数的时候，只会打印上面的信息，不会启动mongodb的程序。所以要关闭该参数，才能开启mongodb。



39、upgrade:

升级。默认为false。
当设置为true，指定DBPATH，升级磁盘上的数据格式的文件到最新版本。会影响数据库操作，更新元数据。大部分情况下，不需要设置该值。

traceExceptions：是否使用内部诊断。默认false。

如：traceExceptions = false



40、quiet：

安静模式。

如：quiet = true



41、setParameter：

2.4的新参数，指定启动选项配置。想设置多个选项则用一个setParameter选项指定，可以setParameter的参数请见这里，详情请见这里
声明setParameter设置在这个文件中，使用下面的格式：

如：setParameter = <parameter>=<value>

如配置文件里设置syncdelay：

1.setParameter = syncdelay= 55,notablescan = true,journalCommitInterval = 50,traceExceptions = true
Replication Options  复制选项



42、replSet：

使用此设置来配置复制副本集。指定一个副本集名称作为参数，所有主机都必须有相同的名称作为同一个副本集。



43、oplogSize：

指定的复制操作日志（OPLOG）的最大大小。mongod创建一个OPLOG的大小基于最大可用空间量。对于64位系统，OPLOG通常是5％的可用磁盘空间。
一旦mongod第一次创建OPLOG，改变oplogSize将不会影响OPLOG的大小。



44、fastsync：

默认为false。在副本集下，设置为true，从一个dbpath里启用从库复制服务，该dbpath的数据库是主库的快照，可用于快速启用同步，否则的mongod将尝试执行初始同步。注意：如果数据不完全同步，mongod指定fastsync开启，secondary或slave与主永久不同步，这可能会导致显着的一致性问题。



45、replIndexPrefetch：

2.2版本出现的新参数，默认是all。可以设置的值有：all, none, and _id_only。只能在副本集（replSet）中使用。默认情况下，secondary副本集的成员将加载所有索引到内存中（从OPLOG之前的操作有关的）。您可以修改此行为，使secondary只会加载_id索引。指定_id_或none，防止mongod的任何索引加载到内存。



46、Master/Slave Replication：主从复制的相关设置

master：默认为false，当设置为true，则配置当前实例作为主实例。

如：master = true



47、slave：

默认为false，当设置为true，则配置当前实例作为从实例。

如：slave = true



48、source：

默认为空，格式为：<host><:port>。用于从实例的复制：设置从的时候指定该选项会让从复制指定主的实例

如：source = 127.0.0.1:30001



49、only：

默认为空，用于从选项，指定一个数据库进行复制。

如：only = abc          #只同步abc集合（库）



50、slavedelay：

设置从库同步主库的延迟时间，用于从设置，默认为0。

如：slavedelay = 60     #延迟60s同步主数据



51、autoresync：

默认为false，用于从设置。是否自动重新同步。设置为true，如果落后主超过10秒，会强制从自动重新同步。如果oplogSize太小，此设置可能有问题。如果OPLOG大小不足以存储主的变化状态和从的状态变化之间的差异，这种情况下强制重新同步是不必要的。当设置autoresync选项设置为false，10分钟内从不会进行大于1次的自动重新同步。
```




### 附录二：添加mongodb到系统服务

<span id="jump2"></span>


----------
MongoDB安装之后，每次都需要运行命令加参数来启动，不便于管理，这里可以将其添加到系统服务，方法如下：
1、创建文件
linux系统服务启动文件都存放在  /etc/init.d/ 下面，service mongod start 这样的命令是通过运行该目录下的脚本实现管理的，所以需要创建一个脚本文件mongod并修改读写权限

cd /etc/init.d/ && touch mongod && chmod 755 mongod 

另外还需要用到一个文件来保存服务pid，service mongod stop 命令就是通过这个文件读取服务pid的,可以建立在MongoDB的安装目录下面，

cd /usr/local/mongodb/ && touch mongod.pid && chmod 755 mongod.pid

2、编写mongod脚本，以下为编写好的脚本示例：

``` shell
#!/bin/bash
# chkconfig: 2345 98 02
# description: mongod service
# 上面的两行必须加上，不然会报错：service mongod does not support chkconfig

# 服务名，会多处调用
prog=mongod

# 状态返回值，functions 函数会调用这个值
RETVAL=0

# 启动命令
mongod=/usr/local/mongodb/bin/mongod
# 启动配置文件
mongodb_conf_file=/usr/local/mongodb/etc/mongodb.conf
# subsys目录下的文件是用于给其他程序判断服务的实例运行状态的
lockfile=/var/lock/subsys/${prog}
# 记录当前mongo运行的pid
pidfile=/usr/local/mongodb/mongod.pid

function mongo_start() {
	if [ -f ${lockfile} ]
	then
		echo "${prog} is starting"	
	else
		echo -ne "Starting ${prog}: \033[1;32m[OK]\033[0m"
		# 启动
		$mongod --config $mongodb_conf_file --fork > /dev/null
		# 写入pid到pidfile
		pgrep ${prog} | sed -n '$p' > ${pidfile}
		RETVAL=$?
		echo
		# 如果pgrep mongo执行查询到mongo的pid则创建锁文件
		[ ${RETVAL} -eq 0 ] &&  touch ${lockfile}
		return ${RETVAL}
	fi
}

function mongo_stop() {
	if [ -f ${lockfile} ]
        then		
		echo -en "Stopping ${prog}: \033[1;32m[OK]\033[0m"
		# 关闭mongo服务
		$mongod --shutdown --config $mongodb_conf_file > /dev/null
		RETVAL=$?
		echo
		[ ${RETVAL} -eq 0 ] &&  /bin/rm -f ${lockfile}
		return ${RETVAL}
	else
		echo "${prog} is stopped"
	fi
}

case $1 in
	start)
		mongo_start
		;;
	stop)
		mongo_stop
		;;
	restart)
		mongo_stop
		mongo_start
		;;
	*)
		echo "Usage:$0 {start | stop | restart}"
		exit 2
		;;
esac

exit $?

```
