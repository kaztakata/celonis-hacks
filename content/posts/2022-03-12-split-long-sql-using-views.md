---
title: "Split Long SQL Using Views"
date: 2022-03-12T16:28:51+09:00
draft: false
tags: ['Event Collection','Transformation']
---

At last post I wrote [Compose Activity from Joining Multiple Tables](../2022-03-05-compose-activity-from-joining-multiple-tables) to create second activity `Close Issue`. Final version of SQL statement was long even I just used three tables. In real process mining project I handled hundred of tables and wrote quite long SQLs. At that time I faced same patterns of SQL in multiple activities. So I introduces `VIEW` in my SQLs for grouping same pattern of `SELECT` SQL, similar to create function (method) in programming. I would like to try it using current project.

Going back to my goal 2) How many users are involved until closing issue written in [Determine Process Mining Tables based on Project Goal](../2022-02-19-determine-process-mining-tables-based-on-project-goal), I would like to create few activities in the middle of on-going issues. For example I would like to add `Take Note` and `Update Progress` activities those are relevant to second and third updates in below screen.

[![image](https://user-images.githubusercontent.com/67397583/158009616-821663ae-b12f-4e4e-a8fa-90addfea1380.png)](https://user-images.githubusercontent.com/67397583/158009616-821663ae-b12f-4e4e-a8fa-90addfea1380.png)

Because these are Planio issue changes appending to change history (journals), necessary fields for activity record are same as `Close Issue` but filtering conditons are different. So I executed Delta Load and see the data by transformation. I found `Update Progress` is similar to `Close Issue` pattern, changing from `status_id` to `done_ratio` in the filter condition of `"journals$details".name`. On the other hand `Take Note` does not have `"journals$details"` record but `journals.notes` column is filled by what I wrote in Planio. 

Until above observation, I feel there is pattern to join `journals` and `journals$details`. Considering `Take Note` (no `journals$details` record) case, it is better to use `LEFT JOIN` than `JOIN`. So I write below SQL to create `tmp_pl_journals VIEW`.

```sql
CREATE VIEW tmp_pl_journals AS
SELECT
     j.id AS id
    ,j.notes AS notes
    ,j.created_on AS created_on
    ,j."issues$id" AS "issues$id"
    ,j."user$id" AS "user$id"
    ,d.name AS name
    ,d.old_value AS old_value
    ,d.new_value AS new_value
FROM journals AS j
LEFT JOIN "journals$details" AS d ON 1=1
    AND j.id = d.journals_id
;
```

Using view `tmp_pl_journals` skips to write JOIN between `journals` and `journals$details`. In case of `Close Issue`, final version SQL of last post is changed to,

```sql
-- only SELECT statement --
SELECT DISTINCT
     i.id AS _CASE_KEY
    ,'journals' AS _ACTIVITY_MAIN_TABLE -- which table I mainly get event log 
    ,j.id||':'||j.name AS _ACTIVITY_KEY -- ID to find record of journals and journals$details
    ,'Close Issue' AS ACTIVITY_EN 
    ,j.created_on AS EVENTTIME
    ,1000 AS _SORTING
    ,j."user$id" AS changed_by
    ,j.name AS changed_attr             -- changed column name
    ,j.old_value AS changed_from        -- changed from value
    ,j.new_value AS changed_to          -- changed to value
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues AS i
JOIN tmp_pl_journals AS j ON 1=1
    AND i.id = j."issues$id"
WHERE 1=1
    AND j.name = 'status_id'
    AND j.new_value = 8
;
```
Compared with previous version, `JOIN` statement is shortened and I feel it is effective. This Planio `JOIN` key is one column, but in SAP tables at least two columns (one is always client number) so it is much effective. 

By the way, in case of `Take Note`, the SQL is 
```sql
-- only WHERE clause --
WHERE 1=1
    AND j.notes IS NOT NULL
;
```

In case of `Update Progress`,
```sql
-- only WHERE clause --
WHERE 1=1
    AND j.name = 'done_ratio'
;
```

I can successfully crop `tmp_pl_journals` part as `VIEW` and split long SQL, and reuse this `VIEW` for another SQL of activities. But still there are three separate SQLs of `Close Issue`, `Take Note`, and `Update Progress`. In the next post I would like to integrate three SQLs to one using `CASE` expression.

Kaz
