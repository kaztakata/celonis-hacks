---
title: "Handle NULL efficiently in Aggregation Function"
date: 2021-06-26T09:37:59+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

I looked at many PQLs that can be simplified if they know about NULL handling well. Today I would like to tell how to handle NULL efficiently in Aggregation Functions (`COUNT`,`SUM`,`AVG` etc.).

Today I would like to use O2C process to explain my case. I determined KPI `Send Invoice within a day after Ship Goods`, because sales company will Ship Goods then should Send Invoice immediately.

Same as previous posts, first I would like to create OLAP table to look into the cases. As below screenshot, I determined `_CASE_KEY` (sales order item), `Customer` for classifying cases later, timestamp of `Ship Goods` and `Send Inovice` activities at first. Timestamps are extracted by `PU_FIRST( VBAP, _CEL_O2C_ACTIVITIES.EVENTTIME, _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Ship Goods')` etc. Timestamp columns are `NULL` (displayed as `-`) if there is no record fullfills Pull up filter condition, that means no specific activity in that case.

[![image](https://user-images.githubusercontent.com/67397583/123500299-11eb8780-d678-11eb-8476-fce5f2c635ef.png)](https://user-images.githubusercontent.com/67397583/123500299-11eb8780-d678-11eb-8476-fce5f2c635ef.png)

Next column `days between Ship and Invoice` is time difference between two activities, as below PQL (This PQL is frequently used so I registered saved formula `days_between_ship_and_invoice`). Screenshot shows that if either of timestamps are `NULL` then time difference is also `NULL`. In other word, time difference value can only be calculated if both timestamps exists.

```
-- days_between_ship_and_invoice
DATEDIFF(
    dd,            -- calculate time difference as day
    PU_FIRST(
        VBAP,
        _CEL_O2C_ACTIVITIES.EVENTTIME,
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Ship Goods'
    ),
    PU_FIRST(
        VBAP,
        _CEL_O2C_ACTIVITIES.EVENTTIME,
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send Invoice'
    )
)
```

The last column `Within a day` (it is also registered as `within_a_day` formula) is judging if time difference is less than 1 day by PQL `CASE WHEN KPI(days_between_ship_and_invoice) < 1 THEN 1 ELSE 0 END`. If `days_between_ship_and_invoice` is `NULL`, then comparing result between `days_between_ship_and_invoice` and `1` is also `NULL`.

To summarize above discussion, `within_a_day` formula returns

- `1` if both activities exists and time difference is less than 1 day
- `0` if both actiivties exists and time difference is more than 1 day
- `NULL` if either activity is not found

Next step is to group `Within a day` column (dimension) by Customer using aggregation functions.

- Invoice Count : `COUNT(KPI(within_a_day))` ignores `NULL` record and count the rest of record.
- Within a day Count : `SUM(KPI(within_a_day))` ignores `NULL` record and sum up the rest of record.
- Within a day Ratio : `AVG(KPI(within_a_day))` is same as `SUM(KPI(within_a_day)) / COUNT(KPI(within_a_day))`, ignoring `NULL` record count from denominator (`COUNT`) and calculate avarage of the rest of record.

Especially for calculating ratio (`AVG`) you need to take care denominator (`COUNT`). Below screenshot shows example of one Customer that have 16 record including 6 `NULL` record, so denominator of ratio is 10.

[![image](https://user-images.githubusercontent.com/67397583/123502094-8debcc80-d684-11eb-91b1-20b6ae8be3eb.png)](https://user-images.githubusercontent.com/67397583/123502094-8debcc80-d684-11eb-91b1-20b6ae8be3eb.png)

In this series of calculation I assume that I can ignore the case that does not have `Send Invoice`. If I think differently that no `Send Invoice` means forgetting to send invoice, of cource it takes more than 1 day and include Ratio calculation.

To implement this considering NULL handling, I will minimumly change `days_between_ship_and_invoice` formula to set default value (today) instead of `NULL` as below PQL. Below screenshot is the result of this PQL change, changing denominator from 10 to 13.

```
DATEDIFF(
    dd,
    PU_FIRST(
        VBAP,
        _CEL_O2C_ACTIVITIES.EVENTTIME,
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Ship Goods'
    ),
    COALESCE(       -- set default value if PU_FIRST result is NULL
        PU_FIRST(
            VBAP, 
            _CEL_O2C_ACTIVITIES.EVENTTIME, 
            _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send Invoice'),
        TODAY()     -- default value is today
    )
)
```

[![image](https://user-images.githubusercontent.com/67397583/123502482-bc1edb80-d687-11eb-8a33-04af851e1746.png)](https://user-images.githubusercontent.com/67397583/123502482-bc1edb80-d687-11eb-8a33-04af851e1746.png)

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20210626.json) then push to your environment by content-cli.

---

