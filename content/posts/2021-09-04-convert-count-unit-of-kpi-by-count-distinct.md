---
title: "Convert count unit of KPI by COUNT DISTINCT"
date: 2021-09-04T09:06:11+09:00
draft: false
tags: ['PQL','Process Analytics','Pull Up Function']
---

Last week I was asked to convert count unit of some KPI (that returns 1 or 0) from delivery item to delivery document (convert if all items in the document are 1 then document KPI is 1, else 0). In this case delivery document and item columns are stored at Activity table like [this post's third topic](../2021-07-24-categorize-and-name-activity/). I already used `DOMAIN_TABLE` to group activity record by delivery item then calculate KPI.

First I tried to group above KPI by `DOMAIN_TABLE` of delivery document but it failed. Error message said `DOMAIN_TABLE` of delivery item for KPI is connected with Activity table with 1-N relationship, and newly created `DOMAIN_TABLE` is also connected with Activity table with 1-M relationship. So two domain tables are N-M relationship that can not be used at once.

Today's post is how to solve this issue by `COUNT DISTINCT` I explained last week ([Count rows of Tables in various way](../2021-08-28-count-rows-of-tables-in-various-way/)). I cannot find exactly same dataset as my issue so I used P2P data model for explanation.

At first I would like to show you the analysis that count document level KPI with item level KPI.

[![image](https://user-images.githubusercontent.com/67397583/132077281-5eaaf877-27b4-4900-a3b0-549add6c2548.png)](https://user-images.githubusercontent.com/67397583/132077281-5eaaf877-27b4-4900-a3b0-549add6c2548.png)

Item level KPI (formula `withinThreshold`) is whether throughput time between Activity 'Book Invoice' and 'Scan Invoice' is within 14 days then 1 else 0. Also, to simulate issues I met above, I used `PU_FIRST( DOMAIN_TABLE( _CEL_P2P_ACTIVITIES_EN._CASE_KEY), _CEL_P2P_ACTIVITIES_EN._CASE_KEY)` as Purchase item.

Next I created Purchase document `PU_FIRST( DOMAIN_TABLE( _CEL_P2P_ACTIVITIES_EN._CASE_KEY), LEFT(_CEL_P2P_ACTIVITIES_EN._CASE_KEY,13))` that removes item number part from case key.

Then I prepared two types of value conversion of item level KPI, false (`withinThresholdFalse`)and all (`withinThresholdAll`). False is converting 1 to NULL and retunr only false (`REMAP_INTS(KPI(withinThreshold),[1,NULL])` ). All is converting 1 to 0 then show both true and false (`REMAP_INTS(KPI(withinThreshold),[1,0])`).

Finally I created formula to sum up document level KPI by `COUNT DISTINCT` as below.

```
COUNT(DISTINCT KPI(purchaseDocument) || KPI(withinThresholdAll)) - 
COUNT(DISTINCT KPI(purchaseDocument) || KPI(withinThresholdFalse))
```

What's happend ? Concatenating purchase document and all flag 0 can create string the has same unique number as purchase document. Concatenating purchase docuemnt and false flag can also create unique number if item level KPI is false (0), but retun NULL value if true (1). If all items are true, concatenating with false flag is all NULL, so this purchase document is not appeared. In the end, counting distinct purchase document minus counting distinct false purchase document means counting distinct true purchase document.

Doing this I can convert item level KPI to document level KPI even I used `DOMAIN_TABLE` for creating item level record.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210904.json) then push to your environment by content-cli.

---

