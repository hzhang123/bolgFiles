---
title: （三）mongoDB增、删、改、查
tags: mongodb
grammar_cjkRuby: true
---

2018年03月07日 09时31分40秒


----------
[TOC]

## 插入基础

db.集合名称.insert({"字段名称" : 插入数据});

``` javascript
// 插入一个文档
db.blog.insert({"url" : "www.baidu.com"});
// 插入多个文档
 db.blog.insert([
{"url" : "www.shifeng.com"},
{"url" : "www.xinlang.com"}]);
```

## 查询
### 基础查询find()、findOne()
find()查询返回所有的字段于所有的文档，findOne()会返回一条文档，pretty()会美化输出，但是pretty()不能与findOne()同时使用，因为findOne()是已经美化输出之后的结果。

固定语法:find的后面的括号中指定的是查询条件,如果括号中什么也不指定会返回所有的文档；{}中可以指定多个查询的条件，为and关系，[逻辑运算](#jump1)中有说明。

db.集合名称.find({"字段名称1" : 匹配的数据1,"字段名称2" : 匹配的数据2});

``` javascript
db.blog.find().pretty();
db.blog.find({"url" : "www.baidu.com"});
```
### 指定返回的字段

db.集合名称.find({查询条件},{设置显示的字段});

find的第二个字段指定返回的字段，指定为1（返回），0（不返回）

 - 默认_id会返回，指定_id为1，只会返回_id，指定为0，其余字段全部返回
 - 只有_id能够指定为0，其余字段假设有一个或多个指定为1的，则只会返回指定的字段，其余字段不能指定为0，其余字段一个不指定则全部返回。可以使用[关系查询](#jump2)中的例子

``` javascript
db.blog.find({"url" : "www.baidu.com"},{"_id" : 0});
db.blog.find({"url" : "www.baidu.com"},{"_id" : 0,"url" : 1});
```

<span id="jump2"></span>
### 关系查询

大于（$gt）、小于（$lt）、大于等于（$gte）、小于等于（$lte）、不等于（$ne）、等于（key:value、$eq）

``` javascript
// 插入数据
db.student.insert({"name" : "张三","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区"});
db.student.insert({"name" : "李四","sex" : "男","age" : 20,"score" : 0,"address" : "朝阳区"});
db.student.insert({"name" : "王五","sex" : "女","age" : 21,"score" : 40,"address" : "东城区"});
db.student.insert({"name" : "马六","sex" : "男","age" : 22,"score" : 60,"address" : "西城区"});
db.student.insert({"name" : "孙七","sex" : "女","age" : 20,"score" : 99,"address" : "海淀区"});
db.student.insert({"name" : "赵八","sex" : "男","age" : 19,"score" : 89,"address" : "东城区"});
db.student.insert({"name" : "钱九","sex" : "女","age" : 21,"score" : 100,"address" : "海淀区"});
db.student.insert({"name" : "刘十","sex" : "男","age" : 20,"score" : 50,"address" : "朝阳区"});

db.student.find({"name" : "张三" });
db.student.find({"age" : {"$gt" : 19}}).pretty();
db.student.find({"score" : {"$gt" : 60}}).pretty();
db.student.find({"name" : {"$ne" : "王五"}}).pretty();
```

<span id="jump1"></span>
### 逻辑运算

就三种类型：与（$and）、或（$or）、非（$not、$nor）。
1. and:是最简单的，只需要用（，）
db.student.find({"age" : {"$gte" : 19,"$lte" : 20}}).pretty();
2. or:
db.student.find({"$or" : [{"age" : {"$gt" : 19}},{"score" : {"$gt" : 90}}]}).pretty();
3. not:
db.student.find({"$not" : [{"age" : {"$gt" : 19}},{"score" : {"$gt" : 90}}]}).pretty()

**注：**==逻辑运算or、not都需要跟上数组==

db.student.find({"age" : {"$ne" : 19}}).pretty();


求模：
对20 求模等于0的结果
db.student.find({"age" : {"$mod" : [20,0]}}).pretty()

对20 求模不等于0的结果
db.student.find({"age" : {"$not" : {"$mod" : [20,0]}}}).pretty()
$in:
db.student.find({"name" : {"$in" : ["张三","李四","王五"]}}).pretty()

## 复杂数据的插入用查询
### 插入数组

``` javascript
db.student.insert({"name" : "黄大仙-A","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区","course" : ["语文","数学","英语","音乐","政治"]});
db.student.insert({"name" : "黄大仙-B","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区","course" : ["语文","数学","英语"]});
db.student.insert({"name" : "黄大仙-C","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区","course" : ["语文","数学","音乐","政治"]});
db.student.insert({"name" : "黄大仙-D","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区","course" : ["语文","数学","英语","政治"]});
db.student.insert({"name" : "黄大仙-E","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区","course" : ["语文","音乐","政治"]});
```

### 数组查询

``` javascript
// 查询学习语文与数学的同学
db.student.find({"course" : {"$all" : ["语文","数学"]}}).pretty()

// 既然有数组就可以建立索引（索引下标从0开始，）
// 查询第三个课程为音乐的同学（index=2）
db.student.find({"course.2" : "音乐"}).pretty()

// 查询只参加三门课程的学生
db.student.find({"course" : {"$size" : 3}}).pretty()

// $slice:限制返回的数组内前两门（2）、后两门（-2）、中间的（[1,2]（前一个是跳过的，后一个代表返回几个））
db.student.find({"age" : 19},{"course" : {"$slice" : 2}}).pretty()
```


### 嵌套集合

``` javascript

db.student.insert({"name" : "流国凳-A","sex" : "女","age" : 20,"score" : 89,"address" : "海淀区",
	"course" : ["语文","数学","英语","音乐","政治"],
	"parents" : [{"name" : "流国凳-A（父亲）","age" : 51, "job" : "工人"},
			{"name" : "流国凳-A（母亲）","age" : 47, "job" : "职员"}
			]});
db.student.insert({"name" : "流国凳-B","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区",
	"course" : ["语文","数学","英语","音乐","政治"],
	"parents" : [{"name" : "流国凳-B（父亲）","age" : 50, "job" : "处长"},
			{"name" : "流国凳-B（母亲）","age" : 46, "job" : "局长"}
			]});
db.student.insert({"name" : "流国凳-C","sex" : "女","age" : 20,"score" : 89,"address" : "海淀区",
	"course" : ["语文","数学","英语","音乐","政治"],
	"parents" : [{"name" : "流国凳-C（父亲）","age" : 50, "job" : "工人"},
			{"name" : "流国凳-C（母亲）","age" : 48, "job" : "局长"}
			]});
db.student.insert({"name" : "流国凳-D","sex" : "女","age" : 20,"score" : 89,"address" : "海淀区",
	"course" : ["语文","数学","英语","音乐","政治"],
	"parents" : [{"name" : "流国凳-D（父亲）","age" : 49, "job" : "教师"},
			{"name" : "流国凳-D（母亲）","age" : 46, "job" : "职员"}
			]});
db.student.insert({"name" : "流国凳-E","sex" : "女","age" : 20,"score" : 89,"address" : "海淀区",
	"course" : ["语文","数学","英语","音乐","政治"],
	"parents" : [{"name" : "流国凳-E（父亲）","age" : 48, "job" : "处长"},
			{"name" : "流国凳-E（母亲）","age" : 48, "job" : "职员"}
			]});
```

### 嵌套集合查询

``` javascript
// 查询年龄大于19，且父母有人是局长的人（对嵌套JSON的访问需要用到$elemMatch）
db.student.find(
	{"$and" : 
		[
			{"age" : 
				{"$gte" : 19}
			},
			{"parents" : 
				{"$elemMatch" : 
					{"job" : "处长"}
				}
			}
		]
	}
).pretty()

// 字段判断
// 查找父母信息存在的人的信息
db.student.find(
	{"parents" : 
		{"$exists" : true}
	}
).pretty()
// course不存在的信息
db.student.find(
	{"course" : 
		{"$exists" : false}
	}
).pretty()


// 条件查询（where子句）

// 年龄大于20的
db.student.find({"$where" : "this.age>20"}).pretty()


db.student.find(
	{"$where" : 
		function(){
			return this.age>20;
		}
	}
).pretty()
// 查询大于19，小于21的
db.student.find({
		"$and" : 
			[
				{"$where" : "this.age>19"},
				{"$where" : "this.age<21"}
			]
	}
).pretty();
```


**注：**在mongo中最好不用where，因为对mongo的索引不兼容，会降低查询速度

## 特定类型的查询
### 正则运算

基础语法：{key : 正则标记}
完整语法：{key : {"$regex" : 正则标记 ,"$options" : 选项}}
	"i":忽略字母大小写
	"m":多行查找
	"x":空白字符除了被转义的和在字符集以外的完全忽略
	"s":匹配所有字符（圆点“.”），包括换行内容
需要注意的是如果直接使用javascript那么只能使用i和m，而s和x必须使用$regex

``` javascript
// 查询以‘流’开头的姓名
db.student.find({"name" : /流/}).pretty()
// 模糊查询简化格式
db.student.find({"name" : /a/i}).pretty()
// 完整格式
db.student.find({"name" : {"$regex" : /a/i}}).pretty()

// 查询数组数据
db.student.find({"caurse" : /语?/}).pretty()
```


### 数据排序
排序操作使用sort()可以有两个排序,升序（1），降序（-1）

``` javascript
// 对成绩降序排序：
db.student.find().sort({"score" : -1}).pretty()
// 在排序中会有一种自然排序（按照数据的保存顺序排序）
db.student.find().sort({"$natural" : -1}).pretty()
```


### 分页显示
skip(n)：跨过多少数据行
limit(n)：返回数据行的限制

``` javascript
// 第一页
db.student.find().skip().limit(5).sort({"age" : -1}).pretty()
// 第二页
db.student.find().skip(5).limit(5).sort({"age" : -1}).pretty()
```



## 数据的更新
### 更新基础语法
> 更新操作，共有两类函数save()、update()

**语法：**db.集合.update(更新对象、新的对象数据(更新操作符),upsert,multi);

upsert：更新的数据如果不存在则增加一天新的数据（true为增加，false为不增加）
multi：表示是否只更新满足条件的第一行数据，如果为false则只更新一条，如果为true，全更新。

``` javascript
// 将年龄为19的人的成绩都更改为100分，只更新返回的第一条
db.student.update({"age" : 19},{"$set" : {"score" : 100}},false,false)

// 满足条件的都更新：
db.student.update({"age" : 19},{"$set" : {"score" : 100}},false,true)

// 更新不存在的数据：
db.student.update({"age" : 30},{"$set" : {"name" : "不存在"}},true,false);
db.student.find({"name" : "不存在"});
```

### 复杂更新、删除（修改器）

#### 1、增加已有键的数值值($inc)

$inc：主要是对于一个数字字段(必须为数字字段)，增加某个数字字段的数据内容，如果这个字段不存在则创建它
语法：{"$inc" : {"成员" : "内容"}}
	
``` javascript
db.student.update({"age" : 19},{"$inc" : {"score" : -30,"age" : 1}});
```


#### 2、修改一个字段的值($set)

$set：进行内容的重新设置,如果这个字段不存在则创建它
语法：{"$set" : {"成员" : "新内容"}}

``` javascript
db.student.update({"age" : 20},{"$set" : {"score" : 89}});
```

	
#### 3、删除某一个文档的字段($unset)

$unset：删除某个成员的内容
语法：{"$unset" : {"成员" : 1}}

``` javascript
// 删除张三的成绩与年龄
db.student.update({"name" : "张三"},{"$unset" : {"age" : 1,"score" : 1}});
```

	
#### 4、追加一个内容到数组($push)

$push：相当于是将内容追加到指定的成员之中（基本是数组）,就是一个对数组数据的添加的，如果没有数组就进行一个新的数组的创建
语法：${"$push" : {"成员" : value}}

``` javascript
// 向张三的数据里面追加数组课程数据
db.student.update({"name" : "张三"},{"$push" : {"course" : ["语文","数学"]}});
db.student.find({"name" : "张三"}).pretty();
// 根据内容就可以看到是两个数组

// 向李四追加一门课程数据
db.student.update({"name" : "李四"},{"$push" : {"course" : "语文"}});
db.student.find({"name" : "李四"}).pretty();

// 向黄大仙-B追加一门政治：
db.student.update({"name" : "黄大仙-B"},{"$push" : {"course" : "政治"}});
db.student.find({"name" : "黄大仙-B"}).pretty();
```

#### 5、追加多个内容到数组($pushAll)

$pushAll：类似的可以一次追加多个内容到数组里面
语法：${"$pushAll" : {"成员" : 数组内容}};

``` javascript
// 向王五添加多个课程
db.student.update({"name" : "王五"},{"$pushAll" : {"course" : ["美术","音乐","素描"]}});
db.student.find({"name" : "王五"}).pretty();
```

#### 6、向数组中添加一个不存在的内容($addToSet)
$addToSet：向数组里面添加一个内容，只有内容不存在的情况下才添加
语法：{"$addToSet" : {"成员" : "内容"}}

``` javascript
// 向王五的信息里增加新的内容
db.student.update({"name" : "王五"},{"$addToSet" : {"course" : "跳舞"}});
db.student.update({"name" : "王五"},{"$addToSet" : {"course" : "美术"}});
db.student.find({"name" : "王五"}).pretty();
```

#### 7、删除数组中的第一或最后一个数据($pop)

$pop：删除数组内的数据
语法：{"$pop" : {"成员" : 内容}}，内容如果设置为-1，则是删除第一个课程

``` javascript
// 删除王五的第一个课程
db.student.update({"name" : "王五"},{"$pop" : {"course" : -1}});
db.student.find({"name" : "王五"}).pretty();
	
// 删除王五的最后一个课程
db.student.update({"name" : "王五"},{"$pop" : {"course" : 1}});
db.student.find({"name" : "王五"}).pretty();
```

#### 8、删除数组中的一个指定内容($pull)

$pull：从数组中删除一个指定内容的数据
语法：{"$pull" : {"成员" : 内容}}，进行数据对比，如果没有匹配的数据则什么也不做

``` javascript
// 删除王五的跳舞课程
db.student.update({"name" : "王五"},{"$pull" : {"course" : "跳舞"}});
db.student.find({"name" : "王五"}).pretty();
```

#### 9、删除数组中的多个内容($pullAll)

$pullAll：一次性删除多个内容
语法：{"$pull" : {"成员" : [数据,数据,数据……]}}

``` javascript
// 删除黄大仙-A中的三门课程
db.student.update({"name" : "黄大仙-A"},{"$pullAll" : {"course" : ["语文","数学","英语"]}})
db.student.find({"name" : "黄大仙-A"}).pretty();
```

#### 10、修改字段的key($rename)

$rename：为成员名称重命名
语法：{"$rename" : {旧的成员名称 : 新的成员名称}}

``` javascript
// 将name字段对应为"张三"的文档的"name"key修改为"newname"
db.student.update({"name" : "张三"},{"$rename" : {"name" : "newname"}})
db.student.find({"newname" : "张三"}).pretty()
// 或
db.student.find().sort({"$natural" : -1}).pretty();
```

	
## 索引
MongoDB中存在两种索引：自动创建的索引与手动创建的索引

``` javascript
db.student.drop()
db.student.insert({"name" : "张三","sex" : "女","age" : 19,"score" : 89,"address" : "海淀区"});
db.student.insert({"name" : "李四","sex" : "男","age" : 20,"score" : 0,"address" : "朝阳区"});
db.student.insert({"name" : "王五","sex" : "女","age" : 21,"score" : 40,"address" : "东城区"});
db.student.insert({"name" : "马六","sex" : "男","age" : 22,"score" : 60,"address" : "西城区"});
db.student.insert({"name" : "孙七","sex" : "女","age" : 20,"score" : 99,"address" : "海淀区"});
db.student.insert({"name" : "赵八","sex" : "男","age" : 19,"score" : 89,"address" : "东城区"});
db.student.insert({"name" : "钱九","sex" : "女","age" : 21,"score" : 100,"address" : "海淀区"});
db.student.insert({"name" : "刘十","sex" : "男","age" : 20,"score" : 50,"address" : "朝阳区"});

// 查看当前的索引
db.student.getIndexes();
```


### 索引创建

db.student.ensureIndex({列:1});
在这儿1表示索引按照升序排列，降序为-1

使用上面的方法创建的索引是默认命名的

``` javascript
// 1、使用下面的函数可以对当前索引进行分析：
db.student.find({"age" : 19}).explain();
// 2、对没有索引的score字段进行查询分析
db.student.find({"score" : {"$gt" : 60}}).explain();
// 3、对索引和费索引进行复合查询，可以看出（过滤操作索引不会启用）
db.student.find({"$or" : 
	[
		{"age" : {"$gt" : 19}},
		{"score" :{"$gt" : 60}}
	]
	}).explain();
// 可以强制使用索引
db.student.find({"$or" : 
	[
		{"age" : {"$gt" : 19}},
		{"score" :{"$gt" : 60}}
	]
	}).hint({"age" : -1,"score" : -1 }).explain();
```


### 删除索引

在集合中创建过多的索引会照成性能的下降
	
``` java
// 删除单个索引：
db.student.dropIndex({"age" : -1});
// 删除所有索引：
db.student.dropIndexes();
```
![索引][1]


  [1]: ./images/1520392380733.jpg