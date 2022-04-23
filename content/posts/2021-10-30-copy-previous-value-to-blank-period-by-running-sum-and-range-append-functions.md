---
title: "Copy Previous Value to Blank Period by RUNNING_SUM and RANGE_APPEND functions"
date: 2021-10-30T09:17:48+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis']
---

At [Investigate Workload Trend of Cropped Subprocess](../2021-10-16-investigate-workload-trend-of-cropped-subprocess) I showed trend of activity count, and at that time I used `RANGE_APPEND` to fill zero count in trend graph. Today I would like to use different aggregation `RUNNING_SUM` and fill value to blank period. 

Imagine you would like to check weekly trend of credit amount regarding some customer. Credit amount is increased by the amount of net value when 'Receive Order' happened, and it is decreased when 'Clear Invoice' happened. There are no activities to change credit amount between 'Receive Order' and 'Clear Invoice' period, but you would like to see credit amount at that period too. Below image is the output of today's post.

[![image](https://user-images.githubusercontent.com/67397583/139514193-7fcb6fb5-e9cb-4a60-9864-6f3da0cb5c7f.png)](https://user-images.githubusercontent.com/67397583/139514193-7fcb6fb5-e9cb-4a60-9864-6f3da0cb5c7f.png)

As first step, I create formula `deltaNetValue` to calculate delta credit amount at next step. When 'Clear Invoice' I multiply net value by -1  to decrease credit amount.

```
CASE
  WHEN _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Receive Order' THEN VBAP.NETWR_CONVERTED
  WHEN _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Clear Invoice' THEN -1 * VBAP.NETWR_CONVERTED
END
```

Second step is to create OLAP table at left side of screen and verify delta calculation. Dimension column is week of each activities, filling blank period by `RANGE_APPEND` as `RANGE_APPEND(ROUND_WEEK(_CEL_O2C_ACTIVITIES.EVENTTIME),'7D')`. I use step size `7D` to fill blank week (7 days).

Next I create KPI `SUM(KPI(deltaNetValue))` to sum up all delta net value belonging each week. You can see first `24,294` value at `2016-07-11` week, and there are no value few weeks afterward.

To sum up all previous delta value, I create new KPI Credit Amount as `RUNNING_SUM(SUM(KPI(deltaNetValue)))`. By creating blank period by `RANGE_APPEND`, I generate running sum value of blank period that is same as previous week's one. For example week `2016-07-25` is same value `24,294` as `2016-07-11`.

Third step is to copy OLAP table and convert it to Area Chart (Area Chart is good to express cumulated total value). Removing unnecessary Delta column then I can create trend graph of credit amount as right side of the screen. You can easily see that credit amount is not changed for 6 weeks from `2016-07-11` to `2016-08-22` and start moving afterward.

This kind of graph is adapted not only credit amount but inventory history. In both cases I can also introduce threshold and grasp safety amount.

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20211030.json) then push to your environment by content-cli.

---

