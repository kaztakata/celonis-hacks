---
title: "Calculate Multi Dimensional KPIs"
date: 2021-06-12T13:20:53+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis']
---

In the last post [Recognize Record to Calculate KPI](../2021-06-05-recognize-record-to-calculate-kpi/), I showed issue of count duplication when I merged two different dimentional KPIs. Today I will tell how to calculate it correctly.

I will start from changing OLAP table to hide `USER_TYPE` and `ACTIVITY_EN` for calculating KPIs grouped by customer master. And I filtered by customer `K1` as previous post. Finally I added SUM function to both `Rework Time` and `Reminder Time`. Below is the current result, calculation result is still incorrect.

[![image](https://user-images.githubusercontent.com/67397583/121765300-d08daf00-cb84-11eb-8f27-4c7316cd74c6.png)](https://user-images.githubusercontent.com/67397583/121765300-d08daf00-cb84-11eb-8f27-4c7316cd74c6.png)

Next I will modify OLAP table to recalculate `Reminder Time` correctly. As explained in previous post, current value `19,660 min` is decomposed to `(10 min as customer K1) x (1,966 Activity record assigned to customer K1)`. If I would like to get `10 min`, I need to divide summed value by `1,996`, count of activity. Activity count is calculated by `COUNT_TABLE(_CEL_O2C_ACTIVITIES)`, so correctly counted PQL is below.

```
SUM(
    CASE
        WHEN PU_COUNT(
            KNA1,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send 1st Payment Reminder'
        ) > 0 THEN 10
        ELSE 0
    END
) / COUNT_TABLE(_CEL_O2C_ACTIVITIES) -- Activity count per customer
```

Appling above PQL to OLAP table then the result `10 min` is correct.

[![image](https://user-images.githubusercontent.com/67397583/121765924-5c093f00-cb89-11eb-9072-4a0b3e9eceb4.png)](https://user-images.githubusercontent.com/67397583/121765924-5c093f00-cb89-11eb-9072-4a0b3e9eceb4.png)

Next I would like to modify `Total Estimated Time`, single KPI value to sum up all customer's result. Unfortunately just copying PQL of OLAP table as below is not working, especially when clearing filter and calculate all customer's value. The reason is both Reminder time and activity count are independently summed up among all customers, then Reminder time of all customers are divided by activity count of all customers.

```
SUM(
    CASE
        WHEN _CEL_O2C_ACTIVITIES.USER_TYPE = 'A' AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %' THEN 1
        ELSE 0
    END
) + SUM(                                -- Sum up Reminder time of all customers
    CASE
        WHEN PU_COUNT(
            KNA1,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send 1st Payment Reminder'
        ) > 0 THEN 10
        ELSE 0
    END
) / COUNT_TABLE(_CEL_O2C_ACTIVITIES)    -- Activity count of all customers
```

The solution is actually quite simple, I will introduce `GLOBAL` for calculating Riminder time like below. This funciton is getting one global aggregation result, so it is not affected from other part of PQL. In below case, customer level calculation can be processed by passing `SUM` function result to `GLOBAL`.

```
SUM(
    CASE
        WHEN _CEL_O2C_ACTIVITIES.USER_TYPE = 'A' AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %' THEN 1
        ELSE 0
    END
) + GLOBAL(SUM(   -- GLOBAL calculates an aggregation function independently as one group.
    CASE
        WHEN PU_COUNT(
            KNA1,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send 1st Payment Reminder'
        ) > 0 THEN 10
        ELSE 0
    END
))
```
Screenshot below is applying above PQL and getting Total Estimated Time `214,798 min` correctly.

[![image](https://user-images.githubusercontent.com/67397583/121766598-169b4080-cb8e-11eb-8931-bf739fb08b28.png)](https://user-images.githubusercontent.com/67397583/121766598-169b4080-cb8e-11eb-8931-bf739fb08b28.png)

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20210612.json) then push to your environment by content-cli.

---
