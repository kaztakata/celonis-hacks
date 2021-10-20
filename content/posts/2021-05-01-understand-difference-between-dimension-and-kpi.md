---
title: "Understand Difference between Dimension and KPI"
date: 2021-05-01T15:25:08+09:00
draft: false
tags: ['PQL','Process Analytics']
---

If you are beginner of creating Celonis Analysis, I believe you will first create OLAP table to understand how Celonis PQL works with data model. Points are,

- Dimensions are grouping key to calculate KPI, normally they are string (character) columns
- KPIs are calculated figures based on specific numeric columns, or count of rows of any columns, it must be numeric value in either cases
- Columns used as dimensions and KPIs are inplicitly joined based on data model

Below screenshot is the quite simple example to show Dimension, KPI, and data model. Dimension is vendor code `LFA1.LIFNR` that is group key to calculate KPI `COUNT(EKKO.MANDT)` that is count of rows of Purchase order table (`MANDT` can be changed to any columns in `EKKO`). I just set up columns in different tables then you can see two tables (vendor master `LFA1` and purchase order `EKKO`) are joined automatically.

[![image](https://user-images.githubusercontent.com/67397583/116774171-b6e14e00-aa95-11eb-8506-9196c09ace4b.png)](https://user-images.githubusercontent.com/67397583/116774171-b6e14e00-aa95-11eb-8506-9196c09ace4b.png)

Next screenshot is the proof of count KPI. I hide KPI then add one more dimension, purchase order number `EKKO.EBELN`. You can see first vendor code `0000000002` that has 4 rows of different purchase order number. Dimension is grouping key of KPI so `0000000002` that is seen in 4 rows are merged to 1 row if I set count KPI, instead when I do not display KPI then 4 rows are displayed as they are.

[![image](https://user-images.githubusercontent.com/67397583/116774364-1ee46400-aa97-11eb-8048-6ae6b4ebf111.png)](https://user-images.githubusercontent.com/67397583/116774364-1ee46400-aa97-11eb-8048-6ae6b4ebf111.png)

When I use functions to operate columns in data model, the idea is the same. In the next screenshot I will add one more KPI using deletion flag `EKKO.LOEKZ` that is originally string column and it is converted to numeric value by `CASE WHEN ... THEN 1.0 ELSE 0.0 END` function judging this column is NULL or not, finally use `SUM` to sum up numeric value by vendor. You can see vendor `0000001000` has 1528 active PO on the other hand PO count is 1530 (means there are 2 deleted PO).

[![image](https://user-images.githubusercontent.com/67397583/116775716-6d95fc00-aa9f-11eb-8e1e-d9f8f010fed5.png)](https://user-images.githubusercontent.com/67397583/116775716-6d95fc00-aa9f-11eb-8e1e-d9f8f010fed5.png)

Similar to previous case, I hide two KPIs and add dimensions of `EKKO.LOEKZ` and column of judging resust whether PO is active or not. It is important that total count of rows (rows of joining `LFA1` and `EKKO`) is not changed after applying function to `EKKO.LOEKZ`. You can imagine one more numeric column `Active PO` is added to `EKKO` dynamically, so that each Purchase order has `Active PO` value (active is 1 and inactive is 0). When I write KPI that include data model columns, firstly result of function is expaneded to `EKKO` dynamically, then sum up value by dimension.

[![image](https://user-images.githubusercontent.com/67397583/116775990-39233f80-aaa1-11eb-92e4-294221dc6078.png)](https://user-images.githubusercontent.com/67397583/116775990-39233f80-aaa1-11eb-92e4-294221dc6078.png)

Even when you use other component (charts and single KPI component), the idea is the same. So I recommend you to start from OLAP table to visualize each step of KPI calculation and test your PQL. Also this idea is applicable to all data model tables including activity tables.

Kaz

---

This post's program can be [downloeded here](../../examples/p2p_analysis_20210501.json) then push to your environment by content-cli.

---