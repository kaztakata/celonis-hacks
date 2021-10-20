---
title: "Recognize Record to Calculate KPI"
date: 2021-06-05T17:34:46+09:00
draft: false
tags: ['PQL','Process Analytics']
---

Using Pull up function, you can calculate various kind of KPIs. I would like to tell in this post is taking care the record (dimensions) to calculate each KPI especially when you unite multiple KPIs to same component.

In this example, I will use Order to Cash process again and first I would like to estiamate time of rework. Definition of rework Activity is same as previous post, and one more condition is it happens by manual user (`USER_TYPE = 'A'`). If rework Activity is operated by manual user, this rework is estimated 1 minute for example. PQL of total estimated rework time is as below.

```
SUM(
    CASE
        WHEN _CEL_O2C_ACTIVITIES.USER_TYPE = 'A' AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %' THEN 1 -- 1 minute
        ELSE 0
    END
)
```

Next I would like to estimate time of writing reminder to customers. Reminder can be sent once per customer, even this customer have multiple order that should be paid. PQL of total reminder time as below. By using Pull up function, Activity record is grouped by customer master table `KNA1`. If `Send 1st Payment Reminder` activity is happened to specific cusotmer, it is estimated as 10 minutes.

```
SUM(
    CASE
        WHEN PU_COUNT(
            KNA1,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send 1st Payment Reminder'
        ) > 0 THEN 10 -- 10 minutes
        ELSE 0
    END
)
```

Finally I would like calculate sum of rework time and reminder time as total estimated time. When I just add two PQLs as below, the result `47,771,418 min` is apparently incorrect (manually calculating result is `214,798 min`). Why ?

[![image](https://user-images.githubusercontent.com/67397583/120887993-f53bd100-c630-11eb-9b41-e56b519aeaaa.png)](https://user-images.githubusercontent.com/67397583/120887993-f53bd100-c630-11eb-9b41-e56b519aeaaa.png)

```
SUM(
    CASE
        WHEN _CEL_O2C_ACTIVITIES.USER_TYPE = 'A' AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %' THEN 1
        ELSE 0
    END
) + SUM(
    CASE
        WHEN PU_COUNT(
            KNA1,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send 1st Payment Reminder'
        ) > 0 THEN 10
        ELSE 0
    END
)
```

In the next screenshot below, I added OLAP table to verify combination of both calculation. To vefiry the resutl easily, I filtered by one specific USER_TYPE / actvitiy / customer. Important thing is there are two Activity record that has same customer code, and reminder time `10 min` is set twice. Due to Pull up function grouped by customer, all activity record with this customer has `10 min`. Finally, as a result of `SUM` function, two `1 min` record and two `10 min` record are summed up then total estimated time is `22 min`.

[![image](https://user-images.githubusercontent.com/67397583/120888302-8790a480-c632-11eb-811d-6b4f8b66705f.png)](https://user-images.githubusercontent.com/67397583/120888302-8790a480-c632-11eb-811d-6b4f8b66705f.png)

In the original definition, rework time is calculated by activity record, instead reminder time is calculated by customer master record. Now customer master record is joined with activity record, so duplicated reminder time are counted unexpectedly due to multiple activity record.

In the next post, I will tell how to calculate sum of two KPIs (total estimated time) correctly.

Kaz

---

This post's program can be [downloeded here](../../examples/o2c_analysis_20210605.json) then push to your environment by content-cli.

---