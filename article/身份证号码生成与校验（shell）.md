---
title: 身份证号码生成与校验（shell） 
tags: shell
grammar_cjkRuby: true
---


## 规则 

``` shell
（1）15位身份证号生成规则
 A     A     A     A     A     A     Y     Y     M    M    D     D     N    N    S
前6位：地区号
第7、第8位为 年份
第9、第10位为 月份
第11、第12位为 日
后三位为随机码，其中 最后一位 奇数分配给男 偶数分配给女。

（2）18位身份证号码 生成规则
A     A     A     A     A     A     Y     Y     Y     Y     M    M    D     D     N    N    S     C
前6位：地区号
第7、第8位、第9、第10位为年份
第11、第12位为 月份
第13、第14位为 日
第15、第16位、第17位为 随机码，其中最后一位 奇数分配给男 偶数分配给女。
第18位 为校验码

校验码生成规则：
将身份证号码按顺序每位分别乘以以下因子
7     9     10   5     8     4     2     1     6     3     7     9     10   5     8     4     2
然后将其相加求和
对11求模运算
余数和校验码之间的对应规则如下：

余数			0     1     2     3     4     5     6     7     8     9     10
对应的校验码	1     0     X     9     8     7     6     5     4     3     2

```

## 示例

``` shell
#!/bin/bash
#
#
# 使用数据库判断日期格式
#
#
function postgresql_connect(){
	local sql=$1
	psql -h 10.0.8.21 "dbname=zhanghao  user=zhanghao password=zhanghao port=5432" -c "${sql}" -A
}
#
#
# 对身份证前17位乘以对应因子求和之后再取余
# @param areaDateSex($1) 地域代码、日期、性别随机码拼接串
#
#
function multiply_add(){
	areaDateSex=$1
	#因子集合
	local factorCollection=(7 9 10 5 8 4 2 1 6 3 7 9 10 5 8 4 2)
	#求和本地变量
	local sum=""
	#身份证最后一位在效验码集合中的索引
	local lastNumCode=""
	# 将拼接好的前17位与每个因子分别相乘求和
	for i in `seq 0 16`
	do
		local factor=${factorCollection[$i]}
		(( sum += ${areaDateSex:$i:1} * factor ))	
	done
	# 对和取余11，得出最后一位的索引
	(( lastNumCode = sum % 11 ))
	return $lastNumCode

}
#
#
# 生成身份证号码
# @param areaNum($1) 地域代码
# @param dateNum($2) 日期
# @param sex($3) 性别
#
#
function production_identity(){
	#
	#初始化所需变量
	#
	local areaNum=$1
	local dateNum=$2
	local sex=$3
	#效验码集合
	local verificationCode=(1 0 X 9 8 7 6 5 4 3 2)


	if [ ${#areaNum} -eq 6 ]
	then
		local isAreaNum=`echo ${areaNum} | gawk --re-interval '{if($1 ~ /^[1-9][0-9]{5}$/){print "t"}else{print "f"}}'`
	else
		echo "ERROR:地域错误！" && exit 0
	fi
	
	#验证日期	
	[ ${#dateNum} -eq 8 ] && [ `echo ${dateNum} | gawk --re-interval '/^([0-9]+)$/{print "0"}'`  -eq 0 ] && local isDateNum=`postgresql_connect "select isDate('${dateNum}');" | grep -v isdate | grep -v row`

	if [ "${isAreaNum}" = "t" ] && [ "${isDateNum}" = "t" ]
	then
		case $sex in
			男)
				while :
				do
					local sexRandomCode=`echo $RANDOM | cksum | cut -c 1-3`
					if [ $[$sexRandomCode % 2] -ne 0 ]
					then
	
						break
					fi
				done
				;;
			女)
				while :
				do
					local sexRandomCode=`echo $RANDOM | cksum | cut -c 1-3`
					if [ $[$sexRandomCode % 2] -eq 0 ]
					then
						
						break
					fi
				done
				;;
			*)
				echo "ERROR:性别输入错误" && exit 0
				;;
		esac
		#
		#将地域代码、日期、性别随机码拼接
		local areaDateSex=$areaNum$dateNum$sexRandomCode
		#
		multiply_add ${areaDateSex}
		local lastNumCode=$?
		local lastNum=${verificationCode[$lastNumCode]}
		echo "您的身份证号码：$areaDateSex$lastNum"
	else
		echo "ERROR:输入参数有误！" && exit 0
	fi
		
}
#
#
# 验证身份证号码 
# @param identity($1) 传入的身份证号码
#
#
function checkout_identity(){
	local identity=$1
	#效验码集合
	local verificationCode=(1 0 X 9 8 7 6 5 4 3 2)
	if [ `echo "${identity}" | gawk --re-interval '{if($1 ~ /^[1-9][0-9]{5}(18|19|([23][0-9]))[0-9]{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)[0-9]{3}[0-9Xx]$/){print "0"}else{print "1"}}'` -eq 0 ]
	then
		multiply_add ${identity:0:17}
		local lastNumCode=$?
		local lastNum=${verificationCode[$lastNumCode]}
		if [ "${lastNum}" = "${identity:17:1}" ]
		then
			echo "身份证合法"
		else
			echo "ERROR:身份证号码错误！" && exit 0
		fi
	else
		echo "ERROR:身份证号码错误！" && exit 0
	fi

}


if [ -n "${3}" ] 
then
	production_identity $1 $2 $3
elif [ -n "${1}" ] && [ -z "${2}" ]
then
	
	checkout_identity "${1}"
else
	echo "身份证生成：sh identity.sh 6位地域代码 出生日期 性别"
	echo -e "\tsh identity.sh 460000 20171203 男"
	echo "身份证检验：sh identity.sh 身份证号码"
	echo -e "\tsh identity.sh 46000020121103331X"
fi	
```