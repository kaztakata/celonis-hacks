---
title: "Determine First Time Right by Pull up function"
date: 2021-05-22T16:31:16+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

As a first example of Pull up function, I will show you First Time Right (FTR) that means process without rework activities ([link](https://dictionary.cambridge.org/dictionary/english/right-first-time)).

To determine whether each case is FTR or not, first I should determine rework activities, then judge if each case has rework activities or not. In this example I determined that rework activity name is started from `Change`. PQL to find such a string pattern is `Change %`, using wildcard `%` after `Change`. Also use keyword `LIKE` with pattern string.

Next step is counting rework activities in each case (`EKPO`, Purchase Order item table in this example) by pull up function. PQL of rework count is below.

```
PU_COUNT(
    EKPO,                                                 -- grouping key
    _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN,                 -- column to be counted
    _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN LIKE 'Change %'  -- filter condition
)
```

As previous explanation of [Understand mechanism of Pull up function](../2021-05-15-understand-mechanism-of-pull-up-function/), by first argument `EKPO`, Activity record is grouped by `EKPO` table key columns. Also by third argument Activity record is filtered by rework activities. In the end rework activities are counted in each case key.

Below screenshot shows that `Change Price` and `Change Quantity` are recognized as rework activities, e.g. PO item `00001` has 2 rework record.

[![image](https://user-images.githubusercontent.com/67397583/119218995-090a1200-bb1e-11eb-9ad7-854206edffee.png)](https://user-images.githubusercontent.com/67397583/119218995-090a1200-bb1e-11eb-9ad7-854206edffee.png)

Finally PQL to calculate FTR is determined as combination of `CASE` statement and previous rework activities count like below. In previous screenshot, PO item `00004` is FTR instead `00001` is not.

```
CASE 
    WHEN PU_COUNT(
        EKPO,
        _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN,
        _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN LIKE 'Change %'        
    ) = 0 THEN 1.0      -- zero rework activity (= FTR)
    ELSE 0.0            -- non-zero rework activity
END
```

Normally you can use single KPI component to summarize FTR ratio (average of FTR cases) as below screenshot. Looking at the OLAP table (I hide Activity column compared with previous one), I can easily verify that FTR ratio is 70% (7 FTR case in total 10 cases). When you build complex PQL logic, I strongly recommend you to split parts as above and verify them by OLAP table.

[![image](https://user-images.githubusercontent.com/67397583/119220916-bafa0c00-bb27-11eb-9a01-1a932dfe350d.png)](https://user-images.githubusercontent.com/67397583/119220916-bafa0c00-bb27-11eb-9a01-1a932dfe350d.png)

Kaz

---

This post's program can be [downloeded here](../../examples/p2p_analysis_20210522.json) then push to your environment by content-cli.

---