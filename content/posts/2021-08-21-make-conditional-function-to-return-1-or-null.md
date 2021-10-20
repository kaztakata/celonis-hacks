---
title: "Make Conditional Function to return 1 or NULL"
date: 2021-08-21T10:42:17+09:00
draft: false
tags: ['PQL','Process Analytics']
---

Previously I posted [Handle NULL efficiently in Aggregation Function](../2021-06-26-handle-null-efficiently-in-aggregation-function/) and discussed how to unite formula of COUNT, SUM, AVG functions. Today I will create boolean function and use it for conditional aggregation.

Imagine you would like to calculate net value of Sales Order (`VBAP.NETWR_CONVERTED`) in O2C process, but there are some conditons to calculate it. First condition is that sales order is active, means rejection reason column (`VBAP.ABGRU`) is not set. Second condition is that Shipping has already done, means 'Ship Goods' activity is included in the case.

Of course it is possible to use component filter to exclude cases, but I do not select this way because I can not manage component filters spreaded to everywhere in the analysis sheet. Instead I will create boolean function to fullfill that condition and return 1 if true, and NULL if false.

Regarding first condition, I will create formula `isActive` like below.

```
REMAP_INTS(ISNULL(VBAP.ABGRU),[0,NULL])
```

I introduce `ISNULL` function that returns 1 if columns value is NULL otherwise returns 0. Then I introduce `REMAP_INTS` against return value of `ISNULL` function. Point is to replace 0 to NULL in this function, then finally I can get 1 if rejection reason is not set (active SO), and NULL if it is set (inactive).

By the way `REMAP_INTS` is easier to read PQL than `CASE WHEN` statement. If I use same function by `CASE WHEN`, PQL is longer and redundant as below.

```
CASE 
  WHEN ISNULL(VBAP.ABGRU) = 1 THEN 1 
  WHEN ISNULL(VBAP.ABGRU) = 0 NULL 
END 
```

Let's go to second condtion named formula `isDelivered` as below, and `CASE WHEN` version too. Apparently first one is simpler.

```
REMAP_INTS(MATCH_ACTIVITIES(_CEL_O2C_ACTIVITIES.ACTIVITY_EN, NODE ['Ship Goods']),[0,NULL])

------

CASE
  WHEN MATCH_ACTIVITIES(_CEL_O2C_ACTIVITIES.ACTIVITY_EN, NODE ['Ship Goods']) = 1 THEN 1
  WHEN MATCH_ACTIVITIES(_CEL_O2C_ACTIVITIES.ACTIVITY_EN, NODE ['Ship Goods']) = 0 THEN NULL
END
```

Regarding this formula I introduced `MATCH_ACTIVITIES` function that returns 1 if each case has activity 'Ship Goods' otherwise returns 0. This function (and similar process functions) is calculated by each case, so it returns result by case key, like PU function against case table. See [Understand mechanism of Pull up function](../2021-05-15-understand-mechanism-of-pull-up-function/) for reference.

Finally I would like to build up conditional net value of each case (named deliveredNetValue) by multiplying three terms. This function returns net value if following two functions return 1, otherwise return NULL.

```
VBAP.NETWR_CONVERTED  * KPI(isActive) * KPI(isDelivered)
```

Using deliveredNetValue I can aggregate COUNT, SUM, AVG of this value as below screen.

[![image](https://user-images.githubusercontent.com/67397583/130308160-d448caa9-7898-47ad-a5bd-69e180f6dd8f.png)](https://user-images.githubusercontent.com/67397583/130308160-d448caa9-7898-47ad-a5bd-69e180f6dd8f.png)

For readability and maintenability perspective I believe conditinal function like above is best practice.

Kaz

---

This post's program can be [downloeded here](../../examples/o2c_analysis_20210821.json) then push to your environment by content-cli.

---

