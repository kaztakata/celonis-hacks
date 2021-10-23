---
title: "Create Matrix of Throughput Time by Pivot Table"
date: 2021-08-07T14:07:40+09:00
draft: false
tags: ['PQL','Process Analytics']
---

At the previous post of [Customize Process Explorer](../2021-05-08-customize-process-explorer/), I showed how to determine throughput time between two activities. This KPI is visible when expanding Process Explorer, but font size of Process Explorer become smaller and smaller when expanding connections, so it is difficult to grasp overview of throughput time.

Until previous post, I usually used OLAP table to show the list of KPI value. Of course it is good enough to grasp overview, but today I used different component 'Pivot Table' to show KPI.

Pivot Table in Celonis is similar to that of other solution, like Microsoft Excel. Characteristics of Pivot Table is two dimensions located at row and column as matrix, and one KPI value like below image. Compared with OLAP table, Pivot Table can only see one KPI at once, but matrix of two dimensions are easy to point out KPI value at one cell and look around neighbor value.

[![image](https://user-images.githubusercontent.com/67397583/128601216-cd4d2d0e-8845-46ce-aba1-f71d4e29c061.png)](https://user-images.githubusercontent.com/67397583/128601216-cd4d2d0e-8845-46ce-aba1-f71d4e29c061.png)

This time I created two dimensions as `SOURCE(_CEL_O2C_ACTIVITIES.ACTIVITY_EN)` and `TARGET(_CEL_O2C_ACTIVITIES.ACTIVITY_EN)` to determine connection between two activities. Then average of throughput time as KPI is determined as below. You can confirm `8 hours` from `Ship Goods` (column) to `Send Invoice` (row) in matrix, compared with Process Explorer.

```
AVG(
  DATEDIFF(
    hh, -- calculated as hours
    SOURCE(_CEL_O2C_ACTIVITIES.EVENTTIME),
    TARGET(_CEL_O2C_ACTIVITIES.EVENTTIME)
  )
)
```

You can also setup another KPIs and switch KPI by 'Configure' button. For example count Case `COUNT(VBAP.MANDT)` and count Activity `COUNT(SOURCE(_CEL_O2C_ACTIVITIES.ACTIVITY_EN))`.

One more thing regarding Pivot Table is, row side dimension can be setup multi layer dimensions. For example, I used Project - milestone - task levels row dimensions to see sum of planned / actual work hours (column dimension is date).

You can choose two types of tables (OLAP, Pivot) based on characteristics of them.

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20210807.json) then push to your environment by content-cli.

---

