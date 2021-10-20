---
title: "Transform Source System Tables to Minimize Data Model Tables"
date: 2021-07-17T13:24:10+09:00
draft: false
tags: ['Data Model','Event Collection','Transformation']
---

In the last part of previous post [Utilize N-M relationship between Activity and Dimension Tables](../2021-07-10-utilize-n-m-relationship-between-activity-and-dimension-tables/), I said just mimicing source system's table structure to data model is not useful for Celonis data model. Also I feel loading time to data model is exponentially long when one more table is added. So minimizing data model table is good practice especially for real time analytics.

In case of SAP Order to Cash (O2C) scenario, case table is sales order item (`VBAP`). From `VBAP` perspective, sales order header `VBAK` can fully utilize as additional dimension table because `VBAK` is joined with `VBAP` as 1 side. In PQL, just choose `VBAK` column as dimension is same effect as choose `VBAP` column.

On the other hand, schedule line `VBEP` table that is joined with `VBAP` as N side can not utilize like `VBAK`. As discussed in previous post, `VBEP` and Activity table are N-M relationship so `VBEP` table should be filtered (e.g. first row of `VBEP`). Schedule line is managing date / time of delivery, so I recommend to convert `VBEP` content to activity table. For example, first row of `VBEP` is normally original request from customer, so you will generate `Request delivery date` activity from `VBEP` table in Event Collection (Transformation). In the end `VBEP` table is not loaded to data model but you can utilize activity record. If you would like to handle order quantity with `Request delivery date` activity, quantity field is added to activity table then use it in PQL.

One more example, partner VBPA table is joined with `VBAP` as N side. It contains sold-to party, ship-to party, bill-to party and payer rows against sales order item. Those are useful as dimension, so I recommend to convert these rows to additional columns of `VBAP` in Event Collection.

[![image](https://user-images.githubusercontent.com/67397583/126027089-d987f453-0434-421d-bc09-2442e55d4839.png)](https://user-images.githubusercontent.com/67397583/126027089-d987f453-0434-421d-bc09-2442e55d4839.png)

As discussed above, transforming source system table structure to Celonis data model is quite important design topic. To do so, of cource you need to think about what KPI is measured then which tables / columns are required for analysis.

Kaz
