title: 修改mongodb字段名字
author: 赵旭东
tags:
  - mongoDB
  - ''
categories:
  - 技术
date: 2017-05-03 11:12:00
---
最近项目在做重构，一些mongodb数据库中集合里面的field名字需要进行重命名。如下图：
![mark](http://ooqkdlcps.bkt.clouddn.com/blog/20170503/123821777.png?imageslim)
将"order"改为"ordered"。

执行以下语句即可：
```db.tb_category.update({},{$rename : {"order" : "ordered"}}, false, true);
```

结果展示：
![mark](http://ooqkdlcps.bkt.clouddn.com/blog/20170503/124323943.png?imageslim)