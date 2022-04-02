---
title: "Create Key Column of Activity Table"
date: 2021-10-23T08:02:29+09:00
draft: false
tags: ['PQL','Process Analytics','Data Model']
---

Celonis Data Model always require unique key in case tabel (case key) to group activities belong to each case. How about activity table ? Activity table do not have explicit key column, instead combination of case key, activity name, timestamp, and sorting number are similar to activity key (those four columns are configured in Data Model). 

This week I was asked to create activity table key column due to duplication check purpose. So today I would like to share how to create key column. For analysis purpose it is not so important, but if you will go to next stage such as notification, automation etc., it may be important topic.

Below screen is the result of creating activity key column. I showed key column and four columns relevant to uniqueness, then counted activity table. The last column told me that key column is unique because maximum count is one.

[![image](https://user-images.githubusercontent.com/67397583/138547491-5ff2a9dc-45c1-4d2f-ac8f-903bb0582079.png)](https://user-images.githubusercontent.com/67397583/138547491-5ff2a9dc-45c1-4d2f-ac8f-903bb0582079.png)

First step to create key column is to concatenate (`||`) four columns, that I already used it many times in this blog. Point to use `||` is columns to be concatenated are always string and not `NULL` value. 

Reviewing four columns, timestamp column is not string column, and sorting column include `NULL` value. Timestamp is not easily converted to string in PQL, so I use `REMAP_TIMESTAMPS` function to convert to counts of the seconds for timestamp since the epoch year (1970-01-01 00:00:00.000). After `REMAP_TIMESTAMPS` it is still number, but it is automatically converted to string before the concatenation is executed. Sorting column is wrapped with `COALESCE` function and set default value 0 (any number is fine).

Concatenated value has already uniqueness in this case but its length differs because e.g. activity name has different length. So I use `STRINGHASH` function against concatenated value to generate hashed and fixed length value. It computes a cryptographic hash of a string, encoded using base64 encoding, so output value is combination of numbers, alphabets, and two special characters (+, /). Concluding above points, below PQL is for activity key column.

```
STRINGHASH(
  _CEL_P2P_ACTIVITIES_EN._CASE_KEY ||
  _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN ||
  REMAP_TIMESTAMPS(_CEL_P2P_ACTIVITIES_EN.EVENTTIME,SECONDS) ||
  COALESCE(_CEL_P2P_ACTIVITIES_EN._SORTING,0)
)
```

By the way, generally I should keep uniqueness by four columns in Data Model load, but even if I violate that uniqueness I just return warning message and succeeded data load. So I need to check count of activity table, then add more columns to create uniqueness if required. Hashed function is good because it shortened to 39 characters, even if concatenated value is longer than.

Finally if you can design activity table, it may be more beneficial to create activity key column in Data Integration. This point I would like to discuss later when I will start Data Integration series.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20211023.json) then push to your environment by content-cli.

---

