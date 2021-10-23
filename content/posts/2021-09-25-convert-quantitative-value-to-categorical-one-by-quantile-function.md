---
title: "Convert Quantitative value to Categorical one by Quantile Function"
date: 2021-09-25T09:42:52+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

Last week I posted [Integrate Button Dropdown Entries to one Formula](../2021-09-18-integrate-button-dropdown-entries-to-one-formula/) and integrated multiple dimensions. At that time I used string column that enable you to categorize each case, that is called categorical variable. In contrast quantitative variable such as order quantity, net value is not normally possible to use for dimension. If you would like to use these column as dimension, you need to convert its value to categorical value. Today I would like to show you how to convert it by quantile function.

Today's output image is as below. I copied last week's analysis then added one more entry 'Net Value Range' to drilldown dimension. This number column is distributed from zero to millions, but these values are categorized to four ranges and used as dimension.

[![image](https://user-images.githubusercontent.com/67397583/134751598-813fae95-5d56-4c6f-a781-babaa67ac98a.png)](https://user-images.githubusercontent.com/67397583/134751598-813fae95-5d56-4c6f-a781-babaa67ac98a.png)

To divide four categories, first I use `MEDIAN` funciton to divide to upper half cases and lower ones, then use `QUANTILE` function to determine boundory of 25% / 75% cases. Finally I compare each case with three boundories and categorize it to four ranges.

By the way Celonis can not use `MEDIAN` or `QUANTILE` as comparator because these aggregation function can be used once (most outer function in PQL). Instead I use Pull Up function that can be used as comparator, because output of Pull Up function behaves as standard column (see [Understand mechanism of Pull up function](../2021-05-15-understand-mechanism-of-pull-up-function/)).

One more points is how to determine the target table of Pull up function. Now I would like to see all cases globally and calculate median and quantiles, so I should set `CONSTANT()` function to target table.

Finally extended formula to categorize net value is below. As output value I also use `PU_MIN` and `PU_MAX`, but it can be expresses by `PU_QUANTILE` with parameter 0% and 100% (`PU_MEDIAN` is 50% too). To distinguish boundary value easily, I rounded it to integer. So it is different from output of boxplot, and case number of four categories are slightly different.

```
--drilldownDimension

CASE
  -- ... same as last week
  WHEN {p1} = 'Net Value Range' THEN 
    CASE
      WHEN EKPO."Net Value(NETWR_EUR)" < ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.25))
        THEN '1.('||ROUND(PU_MIN(CONSTANT(),EKPO."Net Value(NETWR_EUR)"))||' - '||ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.25))||')'
      WHEN EKPO."Net Value(NETWR_EUR)" < ROUND(PU_MEDIAN(CONSTANT(),EKPO."Net Value(NETWR_EUR)"))
        THEN '2.('||ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.25))||' - '||ROUND(PU_MEDIAN(CONSTANT(),EKPO."Net Value(NETWR_EUR)"))||')'
      WHEN EKPO."Net Value(NETWR_EUR)" < ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.75)) 
        THEN '3.('||ROUND(PU_MEDIAN(CONSTANT(),EKPO."Net Value(NETWR_EUR)"))||' - '||ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.75))||')'
      ELSE '4.('||ROUND(PU_QUANTILE(CONSTANT(),EKPO."Net Value(NETWR_EUR)",0.75))||' - '||ROUND(PU_MAX(CONSTANT(),EKPO."Net Value(NETWR_EUR)"))||')'
    END
END 
```

Finally you should take care that result of Pull up function is not changed by filter, so it is not possible to dinamically change boundary by filtering like box plot.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210925.json) then push to your environment by content-cli.

---


