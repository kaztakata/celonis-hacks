---
title: "Determine Process Mining Tables based on Project Goal"
date: 2022-02-19T13:43:41+09:00
draft: false
tags: ['Data Integration','Transformation','Data Model']
---

I talked about how to consider Case ID in advance in [last post](../2022-02-12-consider-case-id-before-starting-transformation). That is same as goal setting of process mining project, it means what would like to be measured in process mining and why. Case is the unit of measuring performance and grouping activities.

Considering my case of Planio, for example I would like to measure 1) Throughput time from raising issue to closing it, 2) How many users are involved until closing issue. From now on I assume my goal is that both performance are measured by each case, then aggregate to some of statistic value (average, median etc.). So that my case ID should be issue ID.

OK, next step is how to store event log. As you may already know, Celonis process mining data model has two major tables, one is activity table and other is case table. Going back to definition of [event log](http://www.processmining.org/event-data.html#data), it is only one table, but Celonis enable us to split it to two parts, one has different field value especially event time, another has same field value in one case ID (case attribute). Two tables are joined in data model and there are same role as event log, also this strategy can reduce duplication of case attribute record.

Going back to my project, first (activity) table should store at least case ID, activity, event time, and user ID those are different value in each record. Second (case) table should store at lease case ID (equals issue ID) and other attributes stored at Planio issue (project ID, Priority, Milestone etc.) to grouping issues. Normally case table is quite similar to source system table that has case ID as primary key, so today I just use issue table that extracted before and do not touch any more.

Of course first activity table is not in source system, so I would like to create table at first transformation. By the way,Celonis EMS transformation is over Vertica Database, so I can see SQL syntax following [Vertica documentation](https://www.vertica.com/docs/). 

I will write below SQL to new transformation then execute it to create activity table `_cel_pl_activities`. There are no case attribute fields those are in `issues` table.

```sql
DROP TABLE IF EXISTS _cel_pl_activities;

CREATE TABLE IF NOT EXISTS _cel_pl_activities (
     _CASE_KEY VARCHAR(50) -- case ID
    ,_ACTIVITY_MAIN_TABLE VARCHAR(100)
    ,_ACTIVITY_KEY VARCHAR(1000)
    ,ACTIVITY_EN VARCHAR(200) -- activity 
    ,EVENTTIME DATETIME -- event time
    ,_SORTING INT
    ,changed_by VARCHAR(80) -- user
    ,changed_email VARCHAR(80)
    ,changed_attr VARCHAR(20)
    ,changed_from VARCHAR (200)
    ,changed_to VARCHAR(200)
    ,changed_from_float FLOAT
    ,changed_to_float FLOAT
    ,changed_from_date DATE
    ,changed_to_date DATE
    ,_CELONIS_CHANGE_DATE DATETIME
);
```

This is just creating table and there are no activity record now, so from next post I would like to insert activity record based on my project goal.

Kaz
