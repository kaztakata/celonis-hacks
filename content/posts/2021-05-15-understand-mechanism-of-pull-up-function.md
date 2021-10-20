---
title: "Understand mechanism of Pull up function"
date: 2021-05-15T10:00:14+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

Pull up aggregation function, called PU function, like `PU_COUNT`, `PU_SUM`, `PU_AVG`, `PU_MAX`, `PU_MIN` looks similar to Standard aggregation function like `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`. In Celonis, it is important to understand that output of PU function is not KPI but dimension. You can see [Understand Difference between Dimension and KPI](../2021-05-01-understand-difference-between-dimension-and-kpi/).

If you know about SQL, you can imagine that PU function is similar to window function in SQL. As other functions, PU functions add dynamic column in grouping table. Output of PU function is representative column value in that group.

Below screenshot is the example of `PU_MAX` function againt purchasing tables, Vendor `LFA1`, PO Document `EKKO`, PO item `EKPO`, also each PO item has Net Order Value `NETWR`. Column `Max Net Value by Doc` representing max value in PO Document has PQL `PU_MAX(EKKO,EKPO.NETWR)`.

[![image](https://user-images.githubusercontent.com/67397583/118344725-c74ff900-b56a-11eb-9641-1de8a36ea4c6.png)](https://user-images.githubusercontent.com/67397583/118344725-c74ff900-b56a-11eb-9641-1de8a36ea4c6.png)

Looking at screenshot, PO Document `0000005763` has max value `3,744.14` in item `00009`, then all other items in this PO also has representative max value `3,744.14`.

When using `PU_MAX` function, 10 items are kept and add one more column representing max value. On the other hand, you can imagine when using standard `MAX` function, 10 items are aggregated to 1 record and show max value.

In above PQL, by table name `EKKO` in first argument, PO item record is partitioned by join key between `EKKO` and `EKPO`. So that below three PQLs are equivalent (`DOMAIN_TABLE` function creates dynamic table of argument columns).

```
PU_MAX(EKKO,EKPO.NETWR) -- using table
PU_MAX(DOMAIN_TABLE(EKKO.MANDT,EKKO.EBELN),EKPO.NETWR) -- using PO Doc columns
PU_MAX(DOMAIN_TABLE(EKPO.MANDT,EKPO.EBELN),EKPO.NETWR) -- using PO item columns
```

Back to screenshot above, column `Max Net Value by Vendor` has PQL `PU_MAX(LFA1,EKPO.NETWR)`. Vendor `0000002901` have 2 PO Document and 20 PO items, then max value `9,213.75` in the second PO is representing max value in this vendor.

Also grouping by any columns in above 3 tables is possible like below.

```
PU_MAX(DOMAIN_TABLE(EKKO.MANDT,EKKO.BUKRS),EKPO.NETWR) -- grouped by column in PO Document 
PU_MAX(DOMAIN_TABLE(EKPO.MANDT,EKPO.MATNR),EKPO.NETWR) -- grouped by column in PO item.
```

In the next post, I will show you how to use output of PU function.

Kaz

---

This post's program can be [downloeded here](../../examples/p2p_analysis_20210515.json) then push to your environment by content-cli.

---