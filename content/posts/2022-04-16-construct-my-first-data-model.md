---
title: "Construct My First Data Model"
date: 2022-04-16T11:03:14+09:00
draft: false
tags: ['Data Integration','Transformation', 'Data Model']
---

Today I would like to create draft version of data model then complete series of ETL (Extraction, Transformation, Load) started from last November.

As I showed in [Adjust Time Zone of Event Time in Global Transformation](../2022-04-02-adjust-time-zone-of-event-time-in-global-transformation), I created global `VIEW` for activity table. Before creating Data Model, I would like to create one more global `VIEW` against case table. As below SQL, it is quite simple to copy all columns of `issues` in Planio connection, plus one more column `_CASE_KEY` that is converting from interger `id` to character. Reason of type conversion is to match with type of activity table (talk later in this post).

```sql
CREATE VIEW issues AS
SELECT 
     TO_CHAR(id) AS _CASE_KEY
    ,* 
FROM <%=DATASOURCE:PLANIO%>.issues
```

OK, I will go to Process Data Models menu, then click New Data Model button. This time I named `Planio Data Model`. First step is to choose tables for data models. As below screen, there are tables from global schema and from data connection planio too. Click `_cel_pl_activities` and `issues` and go to next step. 

[![image](https://user-images.githubusercontent.com/67397583/163664362-5dd2635d-306f-4b8a-9d3d-8e886dfd0347.png)](https://user-images.githubusercontent.com/67397583/163664362-5dd2635d-306f-4b8a-9d3d-8e886dfd0347.png)

Second step is to choose (default) activity table. Of course I choose `_cel_pl_activities` then go to next step. Third step is to configure columns in activity table. As below screen, I will choose 1) Case ID, 2) Activity name, 3) Timestamp and 4) Sorting columns. Normally respective columns are `_CASE_KEY`, `ACTIVITY_EN`, `EVENTTIME` and `_SORTING`, but I can choose any other columns (Timestamp column is limited to Date type column). 

[![image](https://user-images.githubusercontent.com/67397583/163664485-b0e4714d-ae05-4965-ab2d-a9cad3c8bdc0.png)](https://user-images.githubusercontent.com/67397583/163664485-b0e4714d-ae05-4965-ab2d-a9cad3c8bdc0.png)

Fourth step is create join pair in data model tables. As described in [Understand how Tables are joined in Data Model](../2021-06-19-understand-how-tables-are-joined-in-data-model), Celonis Data Model support snowflake schema and the point is join path is determined before data model load. This time join pair is simply between `issues` and `_cel_pl_activities`. Click New Foreign Key button of two tables, then go to Foreign key setting screen as below.

[![image](https://user-images.githubusercontent.com/67397583/163664872-6bb6aa17-b586-4850-a81f-3b19e1635b1c.png)](https://user-images.githubusercontent.com/67397583/163664872-6bb6aa17-b586-4850-a81f-3b19e1635b1c.png)

It is quite important to check which is Dimension table (1) and which is Fact table (N). As mentioned many times in this blog, it is determining 1:N relationship. This time 1 side is `issue` and N side is `_cel_pl_activities` (if oppsite side, click Swap Tables button). Then I will click join columns in both side, this time only one column `_CASE_KEY` but more columns are fine (standard SAP case). Join columns should be same data type (this case character type) and 1 side should have unique value.

Fifth step is to assing case table against activity table. Click three dots in activity table then choose Assign case table, then choose `issues` table. Finally circle `C` mark is displayed before `issues` table.

[![image](https://user-images.githubusercontent.com/67397583/163665198-2db7e8d9-a0a1-40f2-80a2-e5082cd2256d.png)](https://user-images.githubusercontent.com/67397583/163665198-2db7e8d9-a0a1-40f2-80a2-e5082cd2256d.png)

Final step is loading data to data model. Click Data Loads tab and click Force Complete Reload button. It takes few minutes in my environment and finally returned green status. Otherwise you can check error message then retry Data Jobs or Data Model setting.

In the next post, I would like to create simple Analysis to validate my ETL result.

Kaz
