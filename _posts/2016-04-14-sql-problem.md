---
layout: post
title:  "SQL问题记录"
date:   2016-05-06
desc: "SQL问题记录"
keywords: "mysql sql in or"
categories: [Database]
tags: [sql]
icon: icon-mysql
---

# SQL问题记录
## in 和 or

**测试sql**

in或or在参数只有4个时 使用了索引，超过4个使用全表扫描

```
select * from t_goods_feature where category_id in (1,2,3,4,5,6,7,8)
```
```
select * from t_goods_feature where category_id =1 or category_id=2 or category_id =3 or category_id =4
or category_id =5 or category_id =6 or category_id =7 or category_id =8
```




