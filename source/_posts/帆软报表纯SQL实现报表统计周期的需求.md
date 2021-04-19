---
title: 帆软报表纯SQL实现报表统计周期的需求
date: 2021-01-28 15:01:52
tags: ["FineReport"]
---

---

如果我有罪，请让法律来惩罚我，而不是让我连续写一个月又臭又长查询慢的 SQL 实现爸爸的报表需求。

---

## 需求

在报表中实现统计周期的功能，可以按照半小时、天、月、季度、半年、年来统计。

## 效果演示

![按季度统计](/images/20210128150909.png)

![按天统计](/images/20210128150929.png)

<!--more-->

## 实现方法

### 帆软

在报表中新建一个下拉选控件用作统计周期的下拉框，控件名定位 dateTimeType，自定义数据字典如下：

| 实际值                | 显示值 |
| :-------------------- | ------ |
| YYYY-MM-DD HH24:MI:SS | 半小时 |
| YYYY-MM-DD HH24       | 小时   |
| YYYY-MM-DD            | 天     |
| YYYY-MM               | 月     |
| YYYY-q                | 季度   |
| YYYY                  | 年     |

---

**注意**

实际情况中，我的基础数据即为半小时一条已加工的数据，所以这里使用`YYYY-MM-DD HH24:MI:SS`。

---

### SQL

```sql
with dateTimeType as (
    select  '${dateTimeType}' from DUAL
),
summary as  (
    select to_char(t.LOGDATE, (select * from dateTimeType)) AS "date1",t.*
    from xxx.xxxxx t
    where 1 = 1
      AND t.xxx >= (select * from starttime)
      AND t.xxx < (select * from endtime)
),
pool as (
    SELECT
           t."date1" AS "x",
           NVL(SUM(t.x + t.x + t.x), 0)  AS "x"
    FROM x t
    group by t."date1"
)
```
