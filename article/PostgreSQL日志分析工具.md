---
title:  PostgreSQL日志分析工具
tags: postgresql,linux
---

----------

> PostgreSQL日志审计可以配合 pgbench、jmeter...测试工具制定测试计划测试性能，由于日志审计比较影响性能，在不需要问题排查或测试的时候可以关闭。


[toc]


## 1. pgBadger安装

pgBadger:[主页](https://pgbadger.darold.net/)
pgBadger:[下载地址](https://github.com/darold/pgbadger/releases)
Text-CSV_XS-1.39:[下载地址](https://metacpan.org/release/Text-CSV_XS)

### 环境

- Red Hat Enterprise Linux Server release 7.4
- PostgreSQL 9.6.0

### 安装包版本

分析csv格式日志需要Text-CSV_XS

- Text-CSV_XS-1.39
- pgbadger-10.3

### 安装

1. 上传tar包到任意目录

2. 切换到tar包目录执行安装

``` shell
plugs=`echo -e "Text-CSV_XS-1.39\npgbadger-10.3"`;
baseDir=`pwd`; 
for plug in ${plugs}; 
do 
    tar zxf ${plug}.*; 
	cd ${baseDir}/${plug}; 
	perl Makefile.PL && make && make install; 
    cd ${baseDir};
done;
```

## 2. PostgreSQL日志审计配置

postgresql.conf增加配置后重启数据库

``` dsconfig
# 指定csv格式日志
log_destination = 'csvlog'
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%w.log'
log_file_mode = 0640
log_truncate_on_rotation = off
log_rotation_age = 1d
log_line_prefix = '%t [%r]-[%p]: %l user=%u,db=%d'
log_lock_waits = off
log_checkpoints = off
log_connections = off
log_disconnections = off
log_duration = off
log_min_duration_statement = 0
```

## 3. 日志生成

### 访问数据库

``` shell
# 初始化pgbenth依赖表
pgbench -U postgres -d postgres -i -s 5
# 5客户端压力10秒
pgbench -U postgres -d postgres -v -c 5 -T 10
```

### 生成日志

``` shell
# 开启httpd服务
systemctl start httpd.service
# 创建日志输出目录
mkdir -p /var/www/html/pgbadger/;
# 查找一下pgbadger命令所在目录
which pgbadger
# 指定命令路径 -> 指定日志路径(名称根据log_filename参数来的) -> 指定httpd的服务为输出目录: 增量生成日志
/usr/local/bin/pgbadger -I -q /data/pgdata/pg_log/postgresql-*.csv -O /var/www/html/pgbadger/
```

### 定时任务

``` shell
# /etc/cron.d目录下创建文件 pgbadger_cron 并添加如下内容
# 凌晨2点补全目录
0 2 * * * root /usr/bin/mkdir -p /var/www/html/pgbadger/
# 凌晨2点10分分析前天统计日志
10 2 * * * root /usr/local/bin/pgbadger -I -q /data/pgdata/pg_log/postgresql-`date -d "now -1 days" "+%w"`.csv -O /var/www/html/pgbadger/


# 重启crond
systemctl restart crond.service
```

### 结果示例

![增量日志索引页](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561381325439.png)


![概览页](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561436364317.png)


![SQL统计](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561436422758.png)

等等....