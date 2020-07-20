---
title: shell连接主流数据库 
tags: 
---


## 连接PostgreSQL-psql

psql连接数据库有两种方式
1. 连接本地数据库一般使用命令行参数比如：==psql -U username -d database #E91E63==只用指定用户连接到指定的数据库，这种方式在脚本存在于本地机器的时候比较方便。

2. 连接远程的机器通常使用连接串指定一些参数。连接串分为：keyword/value和URIs

``` shell
# psql keyword/value连接

# example
# host=localhost port=5432 user=myuser dbname=mydb password=mypassword connect_timeout=10
# 指定连接串查询一条记录
psql "host=192.X.X.X port=5432 user=zabbix dbname=zabbix password=123456 connect_timeout=10" -c "select * from users limit 1"
```
![keyword/value连接查询](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561216715052.png)

``` shell
# psql URIs 连接

# The general form for a connection URI is 
# postgres[ql]://[user[:password]@][netloc][:port][/dbname][?param1=value1&...]

# example(多个命令放在-c中容易产生意外的结果，最好使用重复的-c命令或将多个命令提供给psql的标准输入)

psql "postgresql://zabbix:123456@192.X.X.X:5432/zabbix" << EOF
\x
select * from users limit 1;
\q
EOF
```

![URIs 连接查询](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1561217162520.png)