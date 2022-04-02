---
title: "Compose Activity from Joining Multiple Tables"
date: 2022-03-05T08:30:59+09:00
draft: false
tags: ['Data Integration','Transformation']
---

In the last post of [Insert Simple Record into Activity Table](../2022-02-26-insert-simple-record-into-activity-table), I created SQL to insert `Raise Issue` activity. That SQL was simple because only one table `issues` are used as data source. Today I will create SQL of `Close Issue` activity that requires multiple tables.

Same as previous post, first I should analyze columns to compose `Close Issue` activity. This is happend when I change Status column in each issue, so I need change history. I already looked at where is change hisory table, that is `journals` table and `journals$details` table (see [Setup Dependent Endpoint in Extractor Builder](../2022-01-29-setup-dependent-endpoint-in-extractor-builder)). As discussed these two tables are recorded at same timing. 

This time case key is provided by `issues` table, but other columns are provided by change history tables. Looking at columns (you can browse columns by Schema Explorer in transformation screen), `journals.created_on` is `EVENTTIME` and `journals."user$id"` is `changed_by` column respectively. I created draft SQL (only `SELECT` statement) as below.

```sql
-- draft 1 --
SELECT DISTINCT
     i.id AS _CASE_KEY
    ,'Close Issue' AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,j."user$id" AS changed_by
FROM issues AS i            -- i is alias of issues
JOIN journals AS j ON 1=1   -- j is alias of journals
    AND i.id = j."issues$id"
;
```

Is it enough ? No, I should restrict record of `journals` to only status change to closed one. To do this I need to check `journals$details` table additionally. 

Change column is stored at `"journals$details".name` column, and in case of status column it is `status_id`. Also changed to value is stored at `"journals$details".new_value` column, and closed status in my enviornment is `8`. I use `JOIN` to get these columns and use `WHERE` clause to restrict journal record as below.

```sql
-- draft 2 --
SELECT DISTINCT
     i.id AS _CASE_KEY
    ,'Close Issue' AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,j."user$id" AS changed_by
FROM issues AS i
JOIN journals AS j ON 1=1
    AND i.id = j."issues$id"
JOIN "journals$details" AS d ON 1=1 -- d is alias of journals$details
    AND j.id = d.journals_id
WHERE 1=1
    AND d.name = 'status_id'        -- only column of status
    AND d.new_value = 8             -- only closed status (value 8 is dependent on my enviornment)
;
```

To complete SQL, I should add `_SORTING` and `_CELONIS_CHANGE_DATE` columns same as previous post. Also for future tracking purpose, I added few information columns from `journals` and `journals$details`. Finally I add `INSERT` statement and final version is as below. In my environment, I ran SQL successfully.

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
    ,'journals' AS _ACTIVITY_MAIN_TABLE -- which table I mainly get event log 
    ,j.id||':'||d.name AS _ACTIVITY_KEY -- ID to find record of journals and journals$details
    ,'Close Issue' AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,1000 AS _SORTING
    ,j."user$id" AS changed_by
    ,d.name AS changed_attr             -- changed column name
    ,d.old_value AS changed_from        -- changed from value
    ,d.new_value AS changed_to          -- changed to value
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues AS i
JOIN journals AS j ON 1=1
    AND i.id = j."issues$id"
JOIN "journals$details" AS d ON 1=1
    AND j.id = d.journals_id
WHERE 1=1
    AND d.name = 'status_id'
    AND d.new_value = 8
;
```
[![image](https://user-images.githubusercontent.com/67397583/156861416-b6f92838-cc25-4260-8a5f-37dfbe5790bd.png)](https://user-images.githubusercontent.com/67397583/156861416-b6f92838-cc25-4260-8a5f-37dfbe5790bd.png)

Today's SQL statement is double lines of previous one. Currently it is still readable but if I write more lines, I need to split SQL to multiple parts and manage readability. Next time I would like to show how to split SQL by creating view.

Kaz
