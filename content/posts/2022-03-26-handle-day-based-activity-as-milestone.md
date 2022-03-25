---
title: "Handle Day based Activity as Milestone"
date: 2022-03-26T02:28:53+09:00
draft: false
tags: ['Event Collection','Transformation', 'Process Analytics']
---

I already created four activities until last post [Unite SQL statements by CASE Expression](../2022-03-19-unite-sql-statements-by-case-expression), those are fillfilled requirement of event log. Going back to [Consider Case ID before Starting Transformation](../2022-02-12-consider-case-id-before-starting-transformation), case ID is the biggest requirement. Also is is not so big as case ID, but event time is important too. In process mining, event time should be year, month, day plus hour, minute, second (`YYYY-MM-DD HH:MI:SS` in Vertica format). I guess event time in process mining referred to that is recorded automatically by system responding to something action. But there are some exceptional cases. Today I would like to discuss about that point, day based activity.

I already determined `Close Issue` activity to calculate throughput time from `Raise Issue`. This itself is meaningful KPI but it is more understandable if I compare with standard throughput time. To calculate standard throughput time, I would like to focus on planned `Due date` column in Planio Issue, and from `Raise Issue` to `Due date` is determined as standard time.

Lookign at Planio screen below, soon I found that I can enter `Due date` as `YYYY-MM-DD` date format, and I can not enter time. I believe creating `Due date` activity is convenient for PQL calculation at creating Analysis, so I somehow fill time for `Due date` to fullfill event time requirement.

[![image](https://user-images.githubusercontent.com/67397583/159487601-38bd7abf-0fb4-4447-a5bb-5704cdea60a4.png)](https://user-images.githubusercontent.com/67397583/159487601-38bd7abf-0fb4-4447-a5bb-5704cdea60a4.png)

By the way if I do not fill time, what is happened ? Below is simple test of day based activity. Simply said, day based activity in Vertica (upper) is converted wierd referring to next activity in Analysis (lower). Detail is described in `Activity table sorting` help page. I do not expect to convert like this, so I should fill default time for day based activity like `Due date`.

[![image](https://user-images.githubusercontent.com/67397583/159489824-fc764de9-a2e3-462f-8a9d-3f07313e6f26.png)](https://user-images.githubusercontent.com/67397583/159489824-fc764de9-a2e3-462f-8a9d-3f07313e6f26.png)

Going back to Planio scenario, SQL to create `Due date has passed` activity is below. I determined `23:59:59` as default time because I want this activity to last of this date's activity.

```sql
INSERT INTO _cel_pl_activities (
     _CASE_KEY
    ,ACTIVITY_EN
    ,EVENTTIME
    ,_SORTING 
    ,_CELONIS_CHANGE_DATE
)
SELECT DISTINCT
     id AS _CASE_KEY
    ,'Due date has passed' AS ACTIVITY_EN
    ,CAST(due_date AS DATE) + CAST('23:59:59' AS TIME) AS EVENTTIME
    ,800 AS _SORTING
    ,NOW() AS _CELONIS_CHANGE_DATE
FROM issues AS i
WHERE 1=1
    AND due_date IS NOT NULL
;
```

In case of my experience in SAP, there are many day based activities like Posting date, Schedule line date etc. Especially many client wanted to replace event time activity with Posting date activity. Generally I do not agree with Posting date activity because this date can be easily manipulated by SAP user, and I think KPI based on that activity is less reliable. 

Finally I should mention about user ID for this activity. This activity is not operated by anyone else, but this is generated from column value, so I do not set user ID. Using user ID column, I can ignore this activity from rework count, automation count etc. Previously I called it as milestone activity in [Categorize and Name Activity](../2021-07-24-categorize-and-name-activity).

Kaz