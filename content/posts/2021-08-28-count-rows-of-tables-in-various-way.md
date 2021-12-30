---
title: "Count rows of Tables in various way"
date: 2021-08-28T11:11:45+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

Counting rows of tables is frequently used so we are not aware about how to do it. But sometimes I got stuck to do it so I would like to deep dive this topic today.

For this demo case I use P2P data model and three tables, header `EKKO` (key columns are `MANDT`/`EBELN`), item `EKPO` (`MANDT`/`EBELN`/`EBELP`) and activity. Also I used sum of item Net value. Below screen is the result of this demo.

[![image](https://user-images.githubusercontent.com/67397583/131204467-0fa76a98-929c-40a3-87e1-707c20613f04.png)](https://user-images.githubusercontent.com/67397583/131204467-0fa76a98-929c-40a3-87e1-707c20613f04.png)

First test is using `COUNT` function in OLAP table. Counting result of `EKKO`/`EKPO` column is not good, but why ? Joining three tables affected to the result due to row duplication. In the end counting result of three tables are same, that is identical to count of Activity.

Second test is using `COUNT_TABLE` funciton. In this case counting result of `EKKO`/`EKPO` column is good, even I joined three tables. I guess `COUNT_TABLE` delete duplicated record internally, so we are easy to use `COUNT_TABLE`. But last column that is sum of Net value is duplicated. Why ?

Even `COUNT_TABLE` column is correctly working, but this OLAP table is actually used three tables. Compared with current result, I also created OLAP table without fourth column (`COUNT_TABLE` of Activity table). Result of Net value is correct in second case, so my guess is verified. Again, even using `COUNT_TABLE` target table is joined in PQL, so other columns are affected by counting table. You may read my previous post [Calculate Multi Dimensional KPIs](../2021-06-12-calculate-multi-dimentional-kpis/) for detail of row duplication.

Third test is using `COUNT DISTINCT` function that calculates the number of distinct elements per group. So I concatenated table primary keys, `EKKO.MANDT||EKKO.EBELN` for header count, and `EKPO.MANDT||EKPO.EBELN||EKPO.EBELP` for item count. Counting result of three tables are good. Why ? Because I explicitly count distinct primary keys even rows of header and item tables are duplicated.

Result of last column (Net value) is duplicated too, but it is predicted. Fourth column that is counting Activity column (_CASE_KEY) is affected to duplication of Net value. To prevent it, counting Activity without joining Activity table. To do so I can use Pull up function `SUM(PU_COUNT(EKPO,_CEL_P2P_ACTIVITIES_EN._CASE_KEY))` that is initially count rows of Activity table by each item, then sum up the result of each item.

In the end either `COUNT_TABLE` or `COUNT DISTINCT` is possible to correctly count rows of tables, but especially for `COUNT_TABLE` please be careful that target table is internally joined and that may affect to result of other columns.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210828.json) then push to your environment by content-cli.

---
