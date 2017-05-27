title: mysql中group by和order by同时使用无效的替代方案
author: 赵旭东
tags:
  - mysql
categories:
  - 技术
date: 2017-05-27 15:16:00
---
## 前言
最近一年由于工作需要大部分使用的都是NoSql数据库，对关系型数据库感觉越来越陌生，一个由`group by`和`order by` 引发的血案由此而生。在此做个记录，以备不时之需。

<!--more-->
## 需求
首先，看一下整体的表结构。
![表结构](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/152755344.png?imageslim)
现在查找每个`barCode`中最新的数据。

由于数据太多，不是很好看到效果。我们就拿一个`barCode`为`4565789`的数据做示例。
```sql
SELECT
	barCode,
	priCommodityID,
	createDate
FROM
	tb_history_version
WHERE
	barCode = '4565789'
ORDER BY
	createDate DESC;
```
![示例数据](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/152218972.png?imageslim)
## 试错
由于很久没有写过sql了。所以首先想到了用 group by和order by组合查询。
```sql
SELECT
	barCode,
	priCommodityID,
	createDate
FROM
	tb_history_version
WHERE
	barCode = '4565789'
GROUP BY
	barCode
ORDER BY
	createDate DESC;
```
结果如下：
![错误结果：1](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/153728638.png?imageslim)
可以看到这并不是我们想要的结果，`order by`没有任何效果。
接下来就试一下运用子查询的方式将两者结合。先排序再分组
```sql
SELECT
	*
FROM
	(
		SELECT
			barCode,
			priCommodityID,
			createDate
		FROM
			tb_history_version
		WHERE
			barCode = '4565789'
		ORDER BY
			createDate DESC
	) AS A
GROUP BY
	A.barCode;
```
结果还是令人失望的
![错误结果：2](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/154555875.png?imageslim)
## 解决
上面两种方式试过了，虽然结果让人伤心，但是工作还是要继续。于是就网上找各种资料,看能否用其他方式解决问题。偶然间看到了`group_concat`可以实现分组排序，就拿来试一试
```sql
SELECT
	barCode,
	GROUP_CONCAT(
		priCommodityID
		ORDER BY
			createDate DESC
	) AS priCommodityID,
	GROUP_CONCAT(
		createDate
		ORDER BY
			createDate DESC
	) AS createDate
FROM
	tb_history_version
WHERE
	barCode = '4565789';
```
结果如下
![结果一](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/155933897.png?imageslim)
可以看到顺序没问题了，但是所有数据都被拼接在一起了。需要进一步做截取字符的处理
```sql
SELECT
	barCode,
	SUBSTRING_INDEX(
		group_concat(
			priCommodityID
			ORDER BY
				createDate DESC
		),
		',',
		1
	) AS priCommodityID,
	SUBSTRING_INDEX(
		group_concat(
			createDate
			ORDER BY
				createDate DESC
		),
		',',
		1
	) AS createDate
FROM
	tb_history_version
WHERE
	barCode = '4565789'
GROUP BY
	barCode;
```
![正确结果](http://ooqkdlcps.bkt.clouddn.com/blog/20170527/160406560.png?imageslim)
ok！到这里就发现已经实现我们刚开始的需求了。
## 总结
`group by`和`order by`同时使用是没有效果的,可以使用`group_concat`和`groub by`替代。`group_concat`内可以实现字段排序。
### 参考文章
http://www.cnblogs.com/jjcc/p/5896588.html