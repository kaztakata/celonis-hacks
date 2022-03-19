---
title: "Unite SQL statements by CASE Expression"
date: 2022-03-19T09:14:35+09:00
draft: false
tags: ['Event Collection','Transformation']
---

Until [Split Long SQL Using Views](../2022-03-12-split-long-sql-using-views) post I created four activities and each SQLs, and I found three of four have same table join pattern. So I created `VIEW` to shorten `JOIN` predicate for each SQL. It is nice to shorten total statement volume but almost all statement except for `JOIN` predicate is duplicated among SQLs. When something change is required, maintenance of each SQLs is annoying work. Today I would like to integrate SQLs using `CASE` expression.

Remember that three activities `Close Issue`, `Take Note` and `Update Progress` are based on `issues` table and `tmp_pl_journals` view including `journals` and `journals$details` tables. Difference is how to filter `tmp_pl_journals` record by its columns (`name`, `new_value` and `notes`).

First step to extract shared pattern like below. `<activity name>` and `<sorting number>` are dummy for explanation.

```sql
-- 1st draft, only SELECT statement --
SELECT DISTINCT
     i.id AS _CASE_KEY
    ,'journals' AS _ACTIVITY_MAIN_TABLE -- which table I mainly get event log 
    ,j.id||':'||j.name AS _ACTIVITY_KEY -- ID to find record of journals and journals$details
    ,'<activity name>' AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,'<sorting number>' AS _SORTING
    ,j."user$id" AS changed_by
    ,j.name AS changed_attr             -- changed column name
    ,j.old_value AS changed_from        -- changed from value
    ,j.new_value AS changed_to          -- changed to value
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues AS i
JOIN tmp_pl_journals AS j ON 1=1
    AND i.id = j."issues$id"
```

Second step is to replace dummy with `CASE` expression based on difference between activities.

```sql
-- 2nd draft, only replacing part --
    ,CASE
        WHEN j.name = 'status_id' AND j.new_value = 8 THEN 'Close Issue'
        WHEN j.notes IS NOT NULL THEN 'Take Note'
        WHEN j.name = 'done_ratio' THEN 'Update Progress'
     END AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,CASE
        WHEN j.name = 'status_id' AND j.new_value = 8 THEN 1000 --'Close Issue'
        WHEN j.notes IS NOT NULL THEN 200 -- 'Take Note'
        WHEN j.name = 'done_ratio' THEN 500 -- 'Update Progress'
     END AS _SORTING
```

Finally filter `tmp_pl_journals` record in `WHERE` clause to match above three cases by `OR` condition. Add `INSERT` statement and become final version.

```sql
-- final version --
INSERT INTO _cel_pl_activities (
     _CASE_KEY
    ,_ACTIVITY_MAIN_TABLE
    ,_ACTIVITY_KEY
    ,ACTIVITY_EN
    ,EVENTTIME
    ,_SORTING
    ,changed_by
    ,changed_attr
    ,changed_from
    ,changed_to
    ,_CELONIS_CHANGE_DATE
)
SELECT DISTINCT
     i.id AS _CASE_KEY
    ,'journals' AS _ACTIVITY_MAIN_TABLE
    ,j.id||':'||COALESCE(j.name, 'notes') AS _ACTIVITY_KEY
    ,CASE
        WHEN j.name = 'status_id' AND j.new_value = 8 THEN 'Close Issue'
        WHEN j.notes IS NOT NULL THEN 'Take Note'
        WHEN j.name = 'done_ratio' THEN 'Update Progress'
     END AS ACTIVITY_EN
    ,j.created_on AS EVENTTIME
    ,CASE
        WHEN j.name = 'status_id' AND j.new_value = 8 THEN 1000 --'Close Issue'
        WHEN j.notes IS NOT NULL THEN 200 -- 'Take Note'
        WHEN j.name = 'done_ratio' THEN 500 -- 'Update Progress'
     END AS _SORTING
    ,j."user$id" AS changed_by
    ,j.name AS changed_attr
    ,j.old_value AS changed_from
    ,j.new_value AS changed_to
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues AS i
JOIN tmp_pl_journals AS j ON 1=1
    AND i.id = j."issues$id"
WHERE  (j.name = 'status_id' AND j.new_value = 8)
    OR j.notes IS NOT NULL
    OR j.name = 'done_ratio'
;
```

Comparing with single `Close Issue` statement in [Compose Activity from Joining Multiple Tables](../2022-03-05-compose-activity-from-joining-multiple-tables), this version is similar lines. So this version including three activities shortens one third SQL lines. 

By the way `WHERE` clause seems complex due to `OR` condition. Alternatively I can do `INSERT` statement without `WHERE` clause, then later `DELETE` record of which `CASE` expression returns `NULL`. 

Below screen shows result without `WHERE` clause. Third record returns `NULL` in `ACTIVITY_EN` column because this is not matched with all cases. Delete record is done simply by `DELETE FROM _cel_pl_activities WHERE ACTIVITY_EN IS NULL;`

[![image](https://user-images.githubusercontent.com/67397583/159101624-694ebc27-c9a6-47c6-a956-510f61d0c2c0.png)](https://user-images.githubusercontent.com/67397583/159101624-694ebc27-c9a6-47c6-a956-510f61d0c2c0.png)

Kaz
