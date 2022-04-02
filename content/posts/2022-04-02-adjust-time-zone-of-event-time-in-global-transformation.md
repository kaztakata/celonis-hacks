---
title: "Adjust Time Zone of Event Time in Global Transformation"
date: 2022-04-02T16:08:17+09:00
draft: false
tags: ['Data Integration','Transformation']
---

Until now I do not care about value of event time, that is stored at source system like Planio. If I analyse only one system like my current project it is no problem, but if I would like to analyze data from multiple source systems, I should take care time zone of each system. 

In my case, I previously analyzed multiple SAP ECC systems, those are located in various area of world (US, EU, JP etc.). Each SAP system has its original timezone in it. Timezone is not relevant to region of server, so for example UTC is used in JP, but other case JST (UTC+9) is used for server located in US. After extracting data from source systems, first I should ask system administrator what is the timezone of this system, then plan to adjust event time before analyzing it.

My client is not only Japanese but other country people too, so I should flexibly change timezone in analysis. To do so, I would like to unite timezones to UTC in Data Integration, then adjust local timezone in analysis (using PQL and hours variable).

OK, so where should I unite timezones to UTC ? My answer is at the last step of transformation before data model load. If I use multiple source systems, I have multiple data connections. Each data connection is linked to Vertica database schema, and one more global data schema exists in data pool. This global data schema does not have data connection but it is possible to access to all database schema (connections) under pool. So my plan is to pick up each connection's activity table in global transformation and at that time convert timezone to UTC.

Below is the sample SQL if I would like to unit two Planio systems, first one has JST timezone and second one has UCT (acutally Planio is used UTC, so this is just for explaining above SAP case etc.). Integration is done by creating `VIEW` and in the `VIEW` I use `UNION` to unite two data connections. Variable `<%=DATASOURCE:PLANIO_1%>` represents database schema ID in Vertica. `TIMESTAMPADD` is timezone adjustment function in Vertica SQL. I should set reversed value `-9` to adjust from JST (UTC+9) to UTC.

```sql
CREATE VIEW _cel_pl_activities AS 
SELECT 
     _CASE_KEY
    ,_ACTIVITY_MAIN_TABLE
    ,_ACTIVITY_KEY
    ,ACTIVITY_EN
    ,TIMESTAMPADD(hh, -9, EVENTTIME) AS EVENTTIME -- from JST to UTC
    ,_SORTING
    ,changed_by
FROM <%=DATASOURCE:PLANIO_1%>._cel_pl_activities -- first connection
UNION -- integrate two connections
SELECT 
     _CASE_KEY
    ,_ACTIVITY_MAIN_TABLE
    ,_ACTIVITY_KEY
    ,ACTIVITY_EN
    ,TIMESTAMPADD(hh, 0, EVENTTIME) AS EVENTTIME -- no change
    ,_SORTING
    ,changed_by
FROM <%=DATASOURCE:PLANIO_2%>._cel_pl_activities -- second connection
;
```

In my actual work, I have Python script to automatically create and execute above SQL not only activity tables but case tables etc. So I always create `VIEW` in global database schema and start data model load.

Kaz
