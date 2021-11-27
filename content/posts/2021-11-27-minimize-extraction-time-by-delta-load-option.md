---
title: "Minimize Extraction Time by Delta Load Option"
date: 2021-11-27T09:51:19+09:00
draft: false
tags: ['Event Collection','Extractor','Extraction','Transformation','Data Job']
---

Last week I extracted Postgres table and looked at the log to understand mechanism of data transfer. At that time I used Full Load option to extract data, that is to replace all table contents and schema to latest version. That is easiest way to synchronize tables between source system (Postgres) and Celonis, but it takes a lot of time to complete this task. So that I should also use second option Delta Load to minimize extraction time.

I already have `app_users` table in Celonis, so I will continue to execute Delta Load today.

First step is to find columns that is sorted by the time of insert record. Let's create new (temporary) transformation task in Celonis then look at the table, then type `SELECT * FROM app_users LIMIT 5;` and execute that SQL. I can see SQL result like below. 

It seems value of column `id` is incremental number from 1. To proof this I also execute two SQLs, `SELECT MAX(id) FROM app_users;` and `SELECT COUNT(*) FROM app_users;`. Both result are same `154821`, so `id` is the column I would like to use for Delta Load.

| id  | firstname | lastname | _CELONIS_CHANGE_DATE | 
| --- | --------- | -------- | -------------------- | 
| 1   | 849b.... | 21cb.... | 2021-11-27 01:49:48  | 
| 2   | 02eb.... | 944e.... | 2021-11-27 01:49:48  | 
| 3   | df57.... | c182.... | 2021-11-27 01:49:48  | 
| 4   | 1bf9.... | 294f.... | 2021-11-27 01:49:48  | 
| 5   | b9f3.... | 6546.... | 2021-11-27 01:49:48  | 


Second step is to change extraction settings I used last week. If I minimize extraction effort, I should not extract already extracted record again and should extract newly created record only. To recognize boundary between those, I create parameter `max_id` that will return value same as `SELECT MAX(id) FROM app_users;` SQL. Setting value is as below screen.

[![image](https://user-images.githubusercontent.com/67397583/143665240-981482ba-0647-499f-ba49-e1a1ea5449ff.png)](https://user-images.githubusercontent.com/67397583/143665240-981482ba-0647-499f-ba49-e1a1ea5449ff.png)

After creating parameter, it is set to Delta Filter Statement at Table Configuration. Statement (WHERE clause of SQL) is `id > <%=max_id%>` as below screen.

[![image](https://user-images.githubusercontent.com/67397583/143665352-c940e7cf-b346-4ddd-9f74-71d01bba8fb6.png)](https://user-images.githubusercontent.com/67397583/143665352-c940e7cf-b346-4ddd-9f74-71d01bba8fb6.png)

To check the Delta Filter, I execute extraction job with Delta Load option. Looking at the log in Celonis, I can find below messages. Parameter `max_id` works well so I do not extract duplicated record, also execution time is 10 seconds, compared with 40 seconds of Full Load last week.

```
2021-11-27 11:28:59 Extraction successful
2021-11-27 11:28:59 Uploading result files
2021-11-27 11:28:59 Total 0 records loaded for table app_users
2021-11-27 11:28:58 Setting filter to: id > 154821
2021-11-27 11:28:58 Starting extraction for resource: app_users
```

By the way, in case of `app_users` table, I use integer number column for Delta Load, but best thing is to find timestamp column at each table. But in my experience, such a column is not always found. If so, you may think about second best option (integer column) as I did.

Next week I would like to insert new record to Postgres table then try Delta Load again to extract it.

Kaz

