---
title: "Inspect Table Data by SELECT statement"
date: 2022-04-09T17:40:32+09:00
draft: false
tags: ['Data Integration','Transformation']
---

Today I would like to share small tips in transformation tasks. Of course main purpose of transforamtion is to create event log table etc. but we are not always sure about source system behavior, so before completing transformation SQL I must investigate data of source system. As I already showed, I can use simple `SELECT` statement to inspect tables. 

Previously when I was familiar with SAP ECC, the only way to easily inspect SAP table is to download table data by `SE16` transaction and open Microsoft Excel and simply filter data or use pivot table etc. There are multiple steps to inspect table data, and annoying thing is every time I should extract all table data from SAP (though simply filter data by where clause is possible). Compared with `SE16` and Excel, Celonis transformation screen enable me to directly write complex SQL to inspect data. I would like to show some of examples.

First scenario is to check key columns are really unique, that means all table record has unique key value. For example at [Setup Dependent Endpoint in Extractor Builder
](../2022-01-29-setup-dependent-endpoint-in-extractor-builder/) post I said `journals$details` table is created by its journal ID and column name. That means `journals$details` table should have key columns of above two columns. To check this, I will execute below `SELECT` statement in new transformation task (or use existing transformation without saving SQL).

```sql
SELECT 
     journals_id
    ,name 
FROM journals$details
GROUP BY journals_id, name
HAVING COUNT(*) > 1
```

As below screen, I expected zero record from this SQL, and got that. This SQL inspect key column values that have 2 and more record (see `HAVING` clause), that is not expected.
[![image](https://user-images.githubusercontent.com/67397583/162565438-4bdaa44f-b554-474e-a1f6-98e332768421.png)](https://user-images.githubusercontent.com/67397583/162565438-4bdaa44f-b554-474e-a1f6-98e332768421.png)

Second scenario is to find combination of attributes in source system table. For example referring [Compose Activity from Joining Multiple Tables](../2022-03-05-compose-activity-from-joining-multiple-tables/) how many patterns of `old_value` and `new_value` are there in changing Planio status. To check this I will execute below statement.

```sql
SELECT DISTINCT
     old_value
    ,new_value
FROM journals$details
WHERE 1=1
    AND name = 'status_id'
ORDER BY old_value, new_value
OFFSET 1
;
```

`DISTINCT` combination of two columns are displayed. Point of this SQL is last part `ORDER BY` and `OFFSET`, because if combination is more than 100, Celonis only displays first 100 records. To check more, I should change to `OFFSET 100`, `OFFSET 200` etc. `ORDER BY` guarantee the sort of record, so result of `OFFSET 100` is exactly next record of `OFFSET 0`.

Third scenario is to check my event log table. As mentioned in [Determine Process Mining Tables based on Project Goal](../2022-02-19-determine-process-mining-tables-based-on-project-goal/), event log in Celonis is composed by two tables, activity table and case table. In the end, I should guarantee all activity record has its case record. To check this relationship, I will execute below statement.

```sql
SELECT
     COUNT(*)
FROM _cel_pl_activities AS a
LEFT JOIN issues AS i ON 1=1
    AND a._CASE_KEY = i.id
WHERE 1=1
    AND i.id IS NULL
;
```

I expected zero count from this SQL. Check if `LEFT JOIN` table column is `NULL`, meaning case (issue) record does not exist.

After learning few technique of SQL like above scenarios, I can really reduce time to investigate source system. It is very nice to do this instead of I did in SAP ECC.

Kaz
