---
title: "Insert Simple Record into Activity Table"
date: 2022-02-26T09:14:12+09:00
draft: false
tags: ['Event Collection','Transformation']
---

In the last post of [Determine Process Mining Tables based on Project Goal](../2022-02-19-determine-process-mining-tables-based-on-project-goal), I talked strategy of transforming Planio data to event log. Then I wrote SQL to create activity table. 

From now I would like to insert activity record to that table. Based on the discussion of last post, at least I need two activities `Raise Issue` and `Close Issue` to measure throughput time between them. That is the next step to achieve my goal.

First, `Raise Issue` activity is happened when Planio issue record is created. As I already looked at [Configure Endpoint for Suitable Extraction](../2022-01-22-configure-endpoint-for-suitable-extraction), when new issue record is created, it is transferred to `issues` table in Celonis EMS. I can see `created_on` column as `EVENTTIME` and `author$id` column as `changed_by` in issue table, and of course `issues` table has its primary key column `id`, it is enough to use only `issues` table to create `Raise Issue` activity. Simply I can write draft SQL as below.

```sql
-- draft --
INSERT INTO _cel_pl_activities (
     _CASE_KEY
    ,ACTIVITY_EN
    ,EVENTTIME
    ,changed_by
)
SELECT DISTINCT
     id AS _CASE_KEY
    ,'Raise Issue' AS ACTIVITY_EN
    ,created_on AS EVENTTIME
    ,author$id AS changed_by
FROM issues
;
```

Before executing all `INSERT` statement, I recommend to execute `SELECT` part of this SQL beforehand (as below), to validate table or column name and expected result. If you execute `INSERT` statement many times, duplicated record are inserted to `issues` table and it is difficult to validate `SELECT` result.

[![image](https://user-images.githubusercontent.com/67397583/155821647-e23209a3-f9a1-4c21-b5a3-532e18acf679.png)](https://user-images.githubusercontent.com/67397583/155821647-e23209a3-f9a1-4c21-b5a3-532e18acf679.png)

For example, in this case I was not sure whether `author$id` is appropriate value for operating user ID, so I tried `"author$id"||"author$name"` instead at `SELECT` statement and confirmed this is actually user ID (by the way, I decide not to include `author$name` because it is not required for further analysis. If `author$id` is not there and it is required to hide personal information, then I use pseudonymization. see [Use Pseudonymized Column as Grouping Key](../2021-12-18-use-pseudonymized-column-as-grouping-key)).

To complete first SQL, I need two more columns `_SORTING` and `_CELONIS_CHANGE_DATE`. `_SORTING` is the integer value to prioritize activity order if two or more activities are same `EVENTTIME`. It is sometimes happened if I changed many fields and save data at once e.g. I assume there is no activity that happened same timing as `Raise Issue`, so any value is fine. This time I set `0`. Regarding `_CELONIS_CHANGE_DATE`, I already saw this column at [Verify Cloning Table Contents via Delta Load](../2021-12-04-verify-cloning-table-contents-via-delta-load). This column is automatically inserted timing of extraction.
It is effective for future debug, so I decide same strategy to my activity table too. I can use `NOW()` function to insert timinig of `INSERT` execution.

Final version of SQL to insert `Raise Issue` is below.

```sql
-- final --
INSERT INTO _cel_pl_activities (
     _CASE_KEY
    ,ACTIVITY_EN
    ,EVENTTIME
    ,_SORTING 
    ,changed_by
    ,_CELONIS_CHANGE_DATE
)
SELECT DISTINCT
     id AS _CASE_KEY
    ,'Raise Issue' AS ACTIVITY_EN
    ,created_on AS EVENTTIME
    ,0 AS _SORTING 
    ,author$id AS changed_by
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues
;
```

Today is the simplest transformation case in process mining. Next time I would like to show another case of joining tables.

Kaz