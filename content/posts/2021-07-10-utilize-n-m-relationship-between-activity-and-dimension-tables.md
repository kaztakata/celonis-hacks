---
title: "Utilize N-M relationship between Activity and Dimension Tables"
date: 2021-07-10T10:08:50+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis','Data Model']
---

When you determine data model structure by yourself, basically you should follow snowflake schema writtern in [Understand how Tables are joined in Data Model](../2021-06-19-understand-how-tables-are-joined-in-data-model/). From case table perspective, n side is case table, and 1 side is another dimension table. On the other hand, activity table is the first case that 1 side is case table. If you determine second dimension table that 1 side is case table, please note that you can not use both activity and that dimension table at once because those tables are N-M relationship via case table. But you can utilize these tables reducing (filtering) rows same as that of case table by Pull Up function.

For example, I would like to sum up net value of sales order that is not `within a day` between Ship Goods and Send Invoice that is discussed in [Handle NULL efficiently in Aggregation Function](../2021-06-26-handle-null-efficiently-in-aggregation-function/). Also I would like to show net value of selected currency by dropdown button.

To do so, first I added new table `VBAP_NET_VALUE` to data model, that table have `_CASE_KEY` (sales order item), `CURRENCY` and `NET_VALUE` columns. Case table `VBAP` and `VBAP_NET_VALUE` are joind by `_CASE_KEY` with 1-N relationship. Below is the data model setting.

[![image](https://user-images.githubusercontent.com/67397583/125154438-01b5cb00-e195-11eb-8e9a-8396cb259043.png)](https://user-images.githubusercontent.com/67397583/125154438-01b5cb00-e195-11eb-8e9a-8396cb259043.png)

Then I create formula to filter `VBAP_NET_VALUE` table by `CURRENCY` column like below.

```
--net_value_by_currency
PU_SUM(
    VBAP,
    VBAP_NET_VALUE.NET_VALUE,
    VBAP_NET_VALUE.CURRENCY = {p1}
)
```

Finally I created single KPI to calculate `net value more than a day` that sum up sales order net value that if filtered by `currency` from dropdown button.

```
SUM(
    KPI(net_value_by_currency,VARIABLE(<%=currency%>)) * 
    CASE WHEN KPI(within_a_day) = 1 THEN 0 ELSE 1 END   -- sum up net value only when within_a_day is false 
)
```

Below video shows the behavior when switching currency from JPY to USD, then recalculate the net value.

[![2021-07-10-_-Process-Analytics-Google-Chrome-2021-07-10-15-30-02](https://user-images.githubusercontent.com/67397583/125154254-13e33980-e194-11eb-9d37-a872b2b33b5d.gif)](https://user-images.githubusercontent.com/67397583/125154254-13e33980-e194-11eb-9d37-a872b2b33b5d.gif)

Unlike this example, just mimicing source system's table structure to data model is not useful for Celonis data model I think.

Kaz
