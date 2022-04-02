---
title: "Understand Delta Load Configuration Difference in Adding Column Scenario"
date: 2021-12-11T21:54:02+09:00
draft: false
tags: ['Data Integration','Extractor','Extraction','Transformation','Data Job']
---

Last time I showed behavior when I added new record then extracted that record by Delta Load ([Verify Cloning Table Contents via Delta Load](../2021-12-04-verify-cloning-table-contents-via-delta-load)). Delta Load is effective way to minimize extraction effort, but it is not always applied. Today, it is continued from previous post, I would like to add column to cloned table and observe behavior of extraction task.

After starting system operation including database, normally system is changing its requirement and extend function and database etc. Let's assume `nickname` is newly required for `app_user` table. 

In pgAdmin, I will do SQL statement `ALTER TABLE app_users ADD COLUMN nickname TEXT;` to add column to table. Then I created new user with SQL `INSERT INTO app_users (firstname, lastname, nickname) VALUES ('Kazuhiko','Takata','Kaz');`. Looking at the table only latest record has nickname value.

| id     | firstname| lastname | nickname | 
| ------ | -------- | -------  | -------- | 
| 154823 | Kazuhiko | Takata   | Kaz      | 
| 154822 | Celonis  | Hacks    |          | 
| 154821 | 0e08.... | 23bd.... |          | 
| 154820 | 6d5f.... | 0a4b.... |          | 
| 154819 | 3201.... | d7ca.... |          | 

Switching to Celonis, then start extraction job with Delta Load option to clone latest record. But I got error message with below log.

```
2021-12-11 22:38:50 Columns: (nickname) have changed. Table won't be delta loaded. Please perform a full load instead.
```

As described in this error message, normally after column changes it is not possible to clone table except Full Load, because Celonis does not know how `nickname` of already cloned record is filled. 

Even above, if I already know cloned record do not have nickname value and only take care about new record, I can still use Delta Load. Going to Celonis extraction task and Extraction settings tab in it, then I will switch Delta Load Configuration from option A to B as below screen. It enables Delta Load with risk of data inconsistency as explained.

[![image](https://user-images.githubusercontent.com/67397583/145679033-19443a6c-0bd9-4c30-8d72-62d79cd79a72.png)](https://user-images.githubusercontent.com/67397583/145679033-19443a6c-0bd9-4c30-8d72-62d79cd79a72.png)

After changing it, again I execute data job and it turns successful. Also new column nickname is appened to last and filled value regarding latest record.

[![image](https://user-images.githubusercontent.com/67397583/145679340-1a53b5f9-941d-4030-90cd-201a68605e88.png)](https://user-images.githubusercontent.com/67397583/145679340-1a53b5f9-941d-4030-90cd-201a68605e88.png)

Which option is better for us ? Basically safer way is to raise error (option A) and manually resolve inconsistency by Full Load. But you should consider frequency of changing database. If it is frequent, everytime after changing database Celonis raises error. From operating perspective, frequent datajob error is may not by allowed because extraction is the first step of Data Integration tasks, so failure of extraction means Data Model still old. In my experience SAP is not changed so much, instead SalesForce is more frequently changed.

Kaz
