---
title: "Customize Process Explorer"
date: 2021-05-08T09:15:03+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis','Process Explorer']
---

Process explorer in Celonis Analysis is default application to analyze process. First you can see main process pattern (called happy path), count of each step (activity) and count of path between two steps (connection). You can show minor activities and connections, or switch throughput time instead of count of connection.

I think there are two ways to customize Process explorer.

- Change activity name
- Add more KPI against activity / connection

### Change activity name
If you want to add detail information to each activity, you can change activity dimension. For example, if I would like to see activity name + user ID to see who is doing this activity in this process, I will change activity dimension to `_CEL_AP_ACTIVITIES.ACTIVITY_EN || ' : ' || _CEL_AP_ACTIVITIES.USER_NAME`. Function `||` is used to concatenate columns or fixed string like `:`.

[![image](https://user-images.githubusercontent.com/67397583/117003132-7137b600-ad1f-11eb-8afb-66cd11b10f42.png)](https://user-images.githubusercontent.com/67397583/117003132-7137b600-ad1f-11eb-8afb-66cd11b10f42.png)

As demonstrated in [Understand Difference between Dimension and KPI](../2021-05-01-understand-difference-between-dimension-and-kpi/), two (and more) tables in activity dimension PQL are inplicitly joined then concatenated string is set to activity dimension. You are careful to use additional column that may have NULL value. Concatenation with NULL returns NULL, and NULL activity is not displayed in Process Explorer. If I would like to prevent it, I will use `COALESCE( _CEL_AP_ACTIVITIES.USER_NAME, '')` instead, then NULL value will convert to zero byte string and activity name is displayed. Below screenshots are to test `COALESCE` funciton in OLAP table, and Process Explorer. Activity `Due Date passed` is displayed in Process Explorer, even it does not have user information.

[![image](https://user-images.githubusercontent.com/67397583/117004211-c88a5600-ad20-11eb-8eea-6a4a3e199a69.png)](https://user-images.githubusercontent.com/67397583/117004211-c88a5600-ad20-11eb-8eea-6a4a3e199a69.png)

[![image](https://user-images.githubusercontent.com/67397583/117003857-59acfd00-ad20-11eb-9d61-88024275fee4.png)](https://user-images.githubusercontent.com/67397583/117003857-59acfd00-ad20-11eb-9d61-88024275fee4.png)

### Add more KPI against activity / connection
If I want to add activity KPI, for example automation ratio judged by user type (B is automation and others are manual), I will determine activity KPI as `AVG ( CASE WHEN _CEL_AP_ACTIVITIES.USER_TYPE = 'B' THEN 1.0 ELSE 0.0 END )` in Process Explorer KPIs. In below screenshot, automation / manual activities are also calculated as KPI to compare with OLAP table result.

[![image](https://user-images.githubusercontent.com/67397583/117014050-62ef9700-ad2b-11eb-9f9b-c4950225739c.png)](https://user-images.githubusercontent.com/67397583/117014050-62ef9700-ad2b-11eb-9f9b-c4950225739c.png)

When I look at `Approve Invoice` activity, automation activity count is `95085` instead manual activity is `207401`, so automation ratio is `95085 / ( 95085 + 207401 ) = 31.43%`

[![image](https://user-images.githubusercontent.com/67397583/117090813-55222c00-ad94-11eb-9e05-19573a400340.png)](https://user-images.githubusercontent.com/67397583/117090813-55222c00-ad94-11eb-9e05-19573a400340.png)

On the other hand if I want to add connection KPI, for example same day ratio, based on throughput time between two activities is less than 1 day, I will determine connection KPI as below PQL.

```
AVG(
   CASE 
   WHEN DATEDIFF(
        dd,
        SOURCE(_CEL_AP_ACTIVITIES.EVENTTIME),
        TARGET(_CEL_AP_ACTIVITIES.EVENTTIME)
    ) < 1.0 THEN 1.0 
    ELSE 0.0 
    END
)
```

This PQL, especially `SOURCE( _CEL_AP_ACTIVITIES.EVENTTIME )` and `TARGET( _CEL_AP_ACTIVITIES.EVENTTIME )` combination, will implicitly create temporary table grouped by neighboring two activities, like below OLAP table (Temporary table can be joined with case table to add another dimensions).

For example, in both OLAP table and Process Explorer you can see same day ratio `3.41%` at connection between `Record Goods Receipt` and `Record Invoice Receipt`,

[![image](https://user-images.githubusercontent.com/67397583/117090514-7f271e80-ad93-11eb-816d-f26ff48c7bf0.png)](https://user-images.githubusercontent.com/67397583/117090514-7f271e80-ad93-11eb-816d-f26ff48c7bf0.png)

Kaz

---

This post's program can be [downloaded here](../../examples/ap_analysis_20210508.json) then push to your environment by content-cli.

---