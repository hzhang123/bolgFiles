---
title: 历史命令与实时记录(redhat6.8)
tags: linux,shell
grammar_cjkRuby: true
---
2018年02月13日 10时58分53秒


----------

[TOC]

> 默认情况下，我们在命令行指定的命令，在我们退出当前用户之后，内存中存储的历史命令会记录到家目录的.history文件中，日志的格式以及这种记录的方法都是根据一系列的参数决定的，我们可以修改这些参数，定制日志记录。

## 参数

### HISTTIMEFORMAT

决定历史记录的格式，是否加时间。

![未指定样式之前][1]

![修改默认样式][2]

我们为本次脚本定制一个更加直观的样式：

``` powershell
# 后面的%F %T是c函数strftime的格式化，可以通过man strftime查看
export HISTTIMEFORMAT="[%F %T][$USER][`who am i 2> /dev/null | gawk '{printf $NF}' | sed -e 's/[()]//g'`]"
```

![定制样式效果][3]
 
### HISTSIZE

控制内存中的历史命令的条数
如：系统的默认1000条，当退出系统的时候会将内存中的历史命令写到文件中

### HISTFILESIZE

文件中存储的历史的条数，如果想禁用写多少条，可以使用HISTFILESIZE=0来禁止写入

### HISTFILE

默认历史记录会写到用户的家目录的.bash_history文件中，我们可以使用这个变量来修改命令被写入的位置

### HISTCONTROL

使用这个变量来控制历史命令的去重

``` powershell
export HISTCONTROL=ignoredups # （去除连续的重复指令）
export HISTCONTROL=erasedups # （去除所有的重复命令）
	
# 命令：history -c清除所有的历史命令
```

### HISIGNORE

在存储的时候忽略某些指令，如果写ls，只会忽略ls，而不会忽略ls -l

`例子：export HISIGNORE="pwd:ls:history"`

## 实时记录参数(PROMPT_COMMAND)

上面的记录方式会有一个限制的地方，那就是我们必须要的等到用户退出的时候才能将历史命令写到文件中去，如果有人history -c  命令就被清空了， 下面提供一种实时写入的方法。

配置PROMPT_COMMAND参数

``` powershell
export PROMPT_COMMAND='{ date "+%F %T ##### $(who am i | gawk "{print $NF}") #### $(history 1|{ read x cmd;echo "$cmd"; })"; } >> /tmp/history.txt'
# 在PROMPT_COMMAND中虽然可以添加时间与获得ip但是由于单引号与双引号的问题，特别的麻烦，拆分为下面的

export HISTTIMEFORMAT="[$USER][`who am i 2> /dev/null | gawk '{printf $NF}' | sed -e 's/[()]//g'`]"

export PROMPT_COMMAND='{ date "+%F %T ##### $(history 1 | { read x cmd;echo "$cmd"; })"; } >> /tmp/history.txt'
```
## 实例脚本

最后附上一个实例脚本，只要放到初始化文件中保证能够刷到环境变量之中即可。

注：==下面脚本在设置忽略命令之后，由于缓冲区保存有一个命令，每次执行命令时会触发写入，此时便会重复写入缓冲区的这个命令==:wink: 

``` powershell
#设置历史文件条数
export HISTSIZE=2000
#设置过滤连续重复指令
export HISTCONTROL=ignoredups
#设置忽略指令
export HISIGNORE="pwd:ls:history:cd"
#设置history格式
export HISTTIMEFORMAT="[%F %T][$USER][`who am i 2> /dev/null | gawk '{printf $NF}' | sed -e 's/[()]//g'`]"

#创建用户日志目录与用户日志文件
##################################################
#
#声明日志目录变量与日志文件变量（不导出为全局变量，子shell会报错）
export mlogdir=/tmp/history/${USER}
export tlogfile=/tmp/history/${USER}/history.$(date +%F).log
#
#判断目录的是否存在、权限和文件是否存在、权限
#
if [ -d "${mlogdir}" ]
then
        if [ -f "${tlogfile}" ]
        then
                if [ -w "${tlogfile}" ]
                then
                        echo -n
                else
                        chmod a+w ${tlogfile}
                fi
        else
                touch ${tlogfile}
                chmod a+w ${tlogfile}
        fi
else
        mkdir -p ${mlogdir}
        touch ${tlogfile}
        chmod a+w ${tlogfile}
fi
#记录shell执行的每一条命令
export PROMPT_COMMAND='{ echo "##### $(history 1 | { read x cmd;echo "$cmd"; })"; } >> ${tlogfile}'
```


  [1]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1553320926949.png
  [2]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1553320926963.png
  [3]: https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1553320926990.png
  [4]: ./images/1518491797015.jpg