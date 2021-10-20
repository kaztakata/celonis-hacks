---
title: "Use Pull up function as Subquery"
date: 2021-05-29T13:04:06+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

In the last post [Determine First Time Right by Pull up function](../2021-05-22-determine-first-time-right-by-pull-up-function/), I broke down PQL of FTR step by step. Today I will explain more complex KPI in similar way. By the way you may know Subquery that enables to pass result of SQL to parts of another SQL. PQL can also do similar things by Pull up function.

In this post, for example I would like to calculate Average rework count after Delivery in Order to Cash process, and definition of rework is same as previous post. I imagine that rework operation after Delivery operation causes more time loss than rework before Delivery, so that I would like to visualize this KPI.

First step is to determine when Delivery is started in each case key (sales order item), so I introduced Pull up function `PU_FIRST` as below. It returns first record by grouping key, with filter condition as option. Below function returns EVENTTIME column of first record of `Generate Delivery Document` activity by each sales order item.

```
PU_FIRST(
    VBAP,                                                           -- grouping key (sales order item)
    _CEL_O2C_ACTIVITIES.EVENTTIME,                                  -- returning column
    _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Generate Delivery Document', -- filter condition
    ORDER BY _CEL_O2C_ACTIVITIES.EVENTTIME                          -- sorting column
)
```

Second step is to determine rework activities after Delivery. It is decomposed to rework definition and comparison of timestamp. PQL using `CASE` statement is below. Point is using result of `PU_FIRST` as comparator of `>=`. One more consideration is using `COALESCE` to set default value if Delivery is not happened yet.

```
CASE 
    WHEN 1=1                                                    -- always true, just for formatting reason
        AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %'     -- rework definition 
        AND _CEL_O2C_ACTIVITIES.EVENTTIME >= COALESCE(PU_FIRST( -- after Delivery timestamp (result of PU_FIRST)
            VBAP,
            _CEL_O2C_ACTIVITIES.EVENTTIME,
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Generate Delivery Document',
            ORDER BY _CEL_O2C_ACTIVITIES.EVENTTIME
        ),{d '9999-12-31'})                                     -- default value if Delivery not found
    THEN 1
    ELSE 0
END
```

Below screenshot is the result of applying above two PQLs to sample case `1217437`. `Generate Delivery Document` is the third record so timestamp of third record is set to `Delivery EVENTTIME` column. After Delivery there are three activity record started from `Change` so these record have `1` in `Rework after Delivery` column.

[![image](https://user-images.githubusercontent.com/67397583/120057779-ca75d980-c080-11eb-80ab-7fcdb6e15080.png)](https://user-images.githubusercontent.com/67397583/120057779-ca75d980-c080-11eb-80ab-7fcdb6e15080.png)

The last step is counting Rework activity by `PU_COUNT` function, below is the PQL. I feel `PU_FIRST` function is similar to SQL Subquery applying to `WHERE` clause.

```
    PU_COUNT(
        VBAP,                                                       -- grouping key (sales order item)
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN,                            -- counting column
        1=1                                                         -- filter condition same as above CASE condition
            AND _CEL_O2C_ACTIVITIES.ACTIVITY_EN LIKE 'Change %'
            AND _CEL_O2C_ACTIVITIES.EVENTTIME >= COALESCE(PU_FIRST(
                VBAP,
                _CEL_O2C_ACTIVITIES.EVENTTIME,
                _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Generate Delivery Document',
                ORDER BY _CEL_O2C_ACTIVITIES.EVENTTIME
            ),{d '9999-12-31'})                         
    )
```

Final result screenshot is as below (hiding other columns except for case key). Above sample case `1217437` has 3 in rework count column. Single KPI value `0.03` is the result of `AVG` function of rework count.

[![image](https://user-images.githubusercontent.com/67397583/120058113-7fa99100-c083-11eb-8fd3-b5a3578b39fb.png)](https://user-images.githubusercontent.com/67397583/120058113-7fa99100-c083-11eb-8fd3-b5a3578b39fb.png)

Kaz

---

This post's program can be [downloeded here](../../examples/o2c_analysis_20210529.json) then push to your environment by content-cli.

---