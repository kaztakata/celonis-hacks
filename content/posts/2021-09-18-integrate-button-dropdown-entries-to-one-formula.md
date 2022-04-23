---
title: "Integrate Button Dropdown Entries to one Formula"
date: 2021-09-18T10:42:24+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis']
---

I usually use two types of analysis components, time scale graph and drilldown table for my development projects. These compnents makes it possible to discover root cause of target KPIs by changing time scale or drilldown dimension.

In the end, these components and attaching button dropdown were maintained many times, but I was annoyed to set variable value (especially long PQL) to each button entry (it was also cause of defects against my analysis sheet). After some version ups I reached to the way how to manage these dropdowns, so today I would like to share with you.

Today's output report image is below. Today's KPIs are simple (case count and net value) to focus on the dimensions. I created button dropdown for time scale unit, and those for drilldown.

[![image](https://user-images.githubusercontent.com/67397583/133868792-062ee0fe-b18e-458d-840b-962f61ba1cde.png)](https://user-images.githubusercontent.com/67397583/133868792-062ee0fe-b18e-458d-840b-962f61ba1cde.png)

Next image is button of time scale unit. Previously I assigned two variables (text and PQL) per button entry, but now one variable (`timeScaleUnit`) that is simply text of time scale unit. X axis of chart (time scale) is described as `KPI(timeScale,<%=timeScaleUnit%>,VARIABLE(poCreationTime))`, I will move time scale PQL from button to formula.

[![image](https://user-images.githubusercontent.com/67397583/133868855-33b22925-80bc-49e4-9479-d9e9de05a423.png)](https://user-images.githubusercontent.com/67397583/133868855-33b22925-80bc-49e4-9479-d9e9de05a423.png)

Next, let's see the formula `timeScale`. It passes two parameters, first is unit name like 'Year', and second is the column to get eventtime (this time it is PO creation date).

```
-- timeScale
CASE
  WHEN {p1} = 'Year' THEN 
    YEAR(KPI({p2}))||''
  WHEN {p1} = 'Month' THEN 
    YEAR(KPI({p2}))||'-'||
    RIGHT(0||MONTH(KPI({p2})), 2)
  WHEN {p1} = 'Week' THEN 
    YEAR(ROUND_WEEK(KPI({p2})))||'-'||
    RIGHT(0||MONTH(ROUND_WEEK(KPI({p2}))),2)||'-'||
    RIGHT(0||DAY(ROUND_WEEK(KPI({p2}))),2)
  WHEN {p1} = 'Date' THEN 
    YEAR(KPI({p2}))||'-'||
    RIGHT(0||MONTH(KPI({p2})), 2)||'-'||
    RIGHT(0||DAY(KPI({p2})), 2)
  WHEN {p1} = 'Day of Week' THEN
    CASE 
      WHEN DAY_OF_WEEK (KPI({p2})) = 1 THEN '1-Monday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 2 THEN '2-Tuesday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 3 THEN '3-Wednesday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 4 THEN '4-Thursday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 5 THEN '5-Friday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 6 THEN '6-Saturday'
      WHEN DAY_OF_WEEK (KPI({p2})) = 0 THEN '7-Sunday'
    END
END
```

I developed `CASE WHEN` statements against each time scale unit judging by unit text p1. You may be doubtful why I do not simply use `ROUND_YEAR`,`ROUND_MONTH` etc. Because these function returns timestamp value and it is not compatible to the result of 'Day of Week' that returns string (`CASE WHEN` should return only one data type from all pattern). So that I converted to number of year, month, day of timestamp, then concatenate them to build string.

Development of drilldown dimension is similar strategy. This formula requests one parameter that represents dimension text, and return string of concatenating value column and text column. This time you are careful which table column is used for drilldown dimension compared with table of KPIs, because all `CASE WHEN` statements are read in this PQL formula even if you choose one of the dimension. This time `LFA1`,`EKKO` and `EKPO` are good for dimension because KPIs are calculated based on case table `EKPO`. On the other hand, if you will add `_CEL_P2P_ACTIVITIES_EN._CASE_KEY` to drilldown dimension, I found net value is unintentionally duplicated from 539M (you will choose '-' entry from button, then you can compare single line value in OLAP table with that of left side single KPI).

```
-- drilldownDimension
CASE
  WHEN {p1} = '-' THEN '-'
  WHEN {p1} = 'Company Code' THEN EKKO.BUKRS||' - '||EKKO."Company Name (EKKO_BUTXT)"
  WHEN {p1} = 'Purchasing Organization' THEN EKKO.EKORG||' - '||EKKO."Purchasing Organization Text (EKKO_EKORG)"
  WHEN {p1} = 'Purchasing Group' THEN EKKO.EKGRP
  WHEN {p1} = 'Document Category' THEN EKKO.BSTYP||' - '||EKKO."Document Category Text (EKKO_BSTYP)"
  WHEN {p1} = 'Document Type' THEN EKKO.BSART||' - '||EKKO."Document Type Text (EKKO_BSART)"
  WHEN {p1} = 'Vendor' THEN EKKO.LIFNR||' - '||LFA1.NAME1
  WHEN {p1} = 'Plant' THEN EKPO.WERKS||' - '||EKPO."Plant Text (EKPO_WERKS)"
  WHEN {p1} = 'Item Category' THEN EKPO."Item Category Text(EKPO_PSTYP)"
  WHEN {p1} = 'Material Group' THEN EKPO.MATKL||' - '||EKPO."Material Group Text (MATKL_TEXT)"
  WHEN {p1} = 'Material' THEN EKPO.MATNR||' - '||EKPO."Material Text (MAKT_MAKTX)"
END
```

Even this strategy is not perfect but it reduces my effor of analysis sheet maintenance. I hope it will help your development.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210918.json) then push to your environment by content-cli.

---
