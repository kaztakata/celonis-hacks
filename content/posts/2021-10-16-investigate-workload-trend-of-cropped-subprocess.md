---
title: "Investigate Workload Trend of Cropped Subprocess"
date: 2021-10-16T10:12:18+09:00
draft: false
tags: ['PQL','Process Analytics','Process Explorer']
---

Celonis Activity record is good to investigate user workload, so you may already implement analysis to do this. Today I would like to crop activities to minimum subprocess then investigate workload against subprocess.

Today's output image is below. I set filter button of company code, also set three buttons to point out activities to crop subprocess pass through these activities. Right side Process Explorer is to check subprocess, in this case it is starting from 'Approve Credit Check' until 'Cancel Order' via 'Deny Credit Check'. Left side bar charts are to count activities in this subprocess by Hour, Day, and Month.

[![image](https://user-images.githubusercontent.com/67397583/137567852-34a663d2-21f1-41ec-bde4-7f68276c787c.png)](https://user-images.githubusercontent.com/67397583/137567852-34a663d2-21f1-41ec-bde4-7f68276c787c.png)

First I would like to determine subprocess. Cropping to subprocess between two activities is done by `CALC_CROP_TO_NULL` function, so I create OR condition of two cropped subprocess by `COALESCE` function as below.

```sql
-- cropTrinityActivity
COALESCE(
  CALC_CROP_TO_NULL(
    FIRST_OCCURRENCE [{p1}] TO LAST_OCCURRENCE [{p2}],
    _CEL_O2C_ACTIVITIES.ACTIVITY_EN
  ),
  CALC_CROP_TO_NULL(
    FIRST_OCCURRENCE [{p2}] TO LAST_OCCURRENCE [{p3}],
    _CEL_O2C_ACTIVITIES.ACTIVITY_EN
  )
)
```

It converts Activity name to NULL except for cropped area. So dimension of Process Explorer is determined as `KPI(cropTrinityActivity, <%=cropFromActivity%>, <%=cropViaActivity%>, <%=cropToActivity%>)` to see cropped subprocess (see [Customize Process Explorer](../2021-05-08-customize-process-explorer/) for detail).

Regarding bar charts, time scale dimension (x axis) is extracted from timestamp column only when Activity name is not NULL. Y axis is just counting non NULL actiivty name by `COUNT` aggregation.

By the way, if you look at x axis precisely, there are blank (zero) area e.g. 8:00 and 5 to 7 day. Normally Celonis do not show blank dimension value but I use `RANGE_APPEND` to fill blank value to solve this. Below is the PQL of Hours case, generating 0 to 23 consecutive values even if there are no case from timestamp. Showing zero value in visual graph is good to understand trend which part is happened and not.

```
RANGE_APPEND(
  CASE 
    WHEN ISNULL(KPI(cropTrinityActivity,<%=cropFromActivity%>,<%=cropViaActivity%>,<%=cropToActivity%>)) = 0 
      THEN HOURS(_CEL_O2C_ACTIVITIES.EVENTTIME) 
  END,
  1,
  0,
  23
)
```

Regarding Month, filling blank month by `RANGE_APPEND` too. In case I do not determine upper and lower limit so these are automatically determined from timestamp value.

```
RANGE_APPEND(
  CASE 
    WHEN ISNULL(KPI(cropTrinityActivity,<%=cropFromActivity%>,<%=cropViaActivity%>,<%=cropToActivity%>)) = 0 
      THEN ROUND_MONTH(_CEL_O2C_ACTIVITIES.EVENTTIME) 
  END,
  '1M'
)
```

In this sample case, subprocess of “Approve Credit but Deny later then cancel order” is happend per half year, and at second half of that month. Due to blank value I feel it is easy to understand time series trend.

Kaz

---

This post's program can be [downloeded here](../../examples/o2c_analysis_20211016.json) then push to your environment by content-cli.

---
