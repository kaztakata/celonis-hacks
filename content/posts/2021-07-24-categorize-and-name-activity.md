---
title: "Categorize and Name Activity"
date: 2021-07-24T10:47:12+09:00
draft: false
tags: ['Data Model','Data Integration','Transformation','PQL','Process Analytics','Pull Up Function','Process Explorer']
---

In the previous post [Transform Source System Tables to Minimize Data Model Tables](../2021-07-17-transform-source-system-tables-to-minimize-data-model-tables/), I recommended to convert some kind of source system tables to Activity. Today I focus on Activity and would like to share my way how to categorize and name Activity.

First point is to split Activity name to two parts, more general part and detail part. For example, in SAP ECC or S4HANA Order to Cash process, general Activity name is `Create Sales Order` when data committed in VA01 transaction. On the other hand detail Activity name is from sales document type (`Standard sales order`, `Cash sale` etc.). General Activity name is used as default Activity name in process mining, more system perspective analysis purpose. Detail Activity name is used as optional analysis or busines perspective analysis.

Detail Activity name can be used in Process Explorer (and Variant Explorer) in Custom dimension setting (refer to [Customize Process Explorer](../2021-05-08-customize-process-explorer/)). In my case `_CEL_O2C_ACTIVITIES.ACTIVITY_EN || ' : ' || COALESCE(_CEL_O2C_ACTIVITIES.ACTIVITY_DETAIL,'')` to concatenate General and Detail parts (consider converting `NULL` to zero byte string to prevent disappering General part).

Second point is to use verb as Activity when it is direct user action to source system, instead use noun when it is just milestone to compare with some user action. One set of example is `Create Goods Issue Document` and `Planned Goods Issue Date`. For further programming purpose, norn Activity does not fill user name column to distinguish it is not direct user action. I do not want to count it for analysing automation Activity, rework Activity and throughuput between Activities, so I used `_CEL_O2C_ACTIVITIES.USER_NAME` IS NOT NULL to exclude norn Activity.

Also you can exclude norn activity in Process Explorer by below PQL.

```
CASE 
    WHEN _CEL_O2C_ACTIVITIES.USER_NAME IS NOT NULL THEN _CEL_O2C_ACTIVITIES.ACTIVITY_EN
    ELSE NULl
END
```

After I found I can hide norn Activities as I like, I create new Activities without hesitation. Now I used hundred of Activities for multiple purposes. For example, create `Planned Goods Issue Date` for on time Delivery purpuse.

Third point is to fill additional case key columns to Activity table. In case of on time Delivery analysis, I need to consider multiple (partial) delivery scenario against one sales order. I filled delivery number to delivery relevant Activity record to distinguish each record is first delivery, second delivery etc.

After filling delivery number `VBELN_VL` and delivery item number `POSNR_VL`, I can use Pull Up Function to calculate each delivery is on time like below PQL. `DOMAIN_TABLE` is the way to group by delivery even activity main case key is sales order.

```
DATEDIFF(
    dd,
    PU_FIRST(
        DOMAIN_TABLE(
            _CEL_O2C_ACTIVITIES._CASE_KEY,
            _CEL_O2C_ACTIVITIES.VBELN_VL,
            _CEL_O2C_ACTIVITIES.POSNR_VL
        ),
        _CEL_O2C_ACTIVITIES.EVENTTIME,
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Create Goods Issue Document'
    ),
    PU_FIRST(
        DOMAIN_TABLE(
            _CEL_O2C_ACTIVITIES._CASE_KEY,
            _CEL_O2C_ACTIVITIES.VBELN_VL,
            _CEL_O2C_ACTIVITIES.POSNR_VL
        ),
        _CEL_O2C_ACTIVITIES.EVENTTIME,
        _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Planned Goods Issue Date'
    )
)
```

Using above three points, I believe I can maximize process analysis with minimum dataset.

Kaz
