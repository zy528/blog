---
title: "Oracle、PostGreSql多表关联更新"
date: 2019-05-30T16:32:01+08:00
draft: false
tags: 
  - 数据库
---

## Oracle、PostGreSql多表关联更新

##### oracle
方法1:
```
UPDATE  表2
SET
  表2.C  =  (SELECT  B  FROM  表1  WHERE   表1.A = 表2.A)
WHERE
  EXISTS ( SELECT 1 FROM   表1  WHERE   表1.A = 表2.A)
```
方法2:
```
MERGE INTO 表2 
USING 表1
ON ( 表2.A = 表1.A )    -- 条件是 A 相同
WHEN MATCHED THEN UPDATE SET 表2.C = 表1.B   -- 匹配的时候，更新
```
##### PostGreSql
```
update a
set a.xx = b.xx 
from b 
where a.id = b.id
```
