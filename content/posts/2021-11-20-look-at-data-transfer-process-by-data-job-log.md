---
title: "Look at Data Transfer Process by Data Job Log"
date: 2021-11-20T08:39:24+09:00
draft: false
tags: ['Data Integration','Extractor','Extraction','Data Job']
---

Last week I posted [Connect to Celonis and Bring Back Instruction](../2021-11-13-connect-to-celonis-and-bring-back-instruction) to look at how Extractor works to connect between Celonis and Postgres. This week I would like to extract data from Postgres and look at data transfer process by data job log.

In the Data Integration, I create new Data Job with Data Connection I created last week, then create new extraction task. In the next screen I add new table `public.app_users` that is pre-installed in Postgres when I deployed it (You can see detail by `sample.sql` in celonis-postgres folder). Going back to table configuration screen, I should touch somewhere in screen and redo it, then save button is enabled. Then click save it.

[![image](https://user-images.githubusercontent.com/67397583/142714726-6d35f478-a2d8-4c5f-8004-07b33dabbafa.png)](https://user-images.githubusercontent.com/67397583/142714726-6d35f478-a2d8-4c5f-8004-07b33dabbafa.png)


Then I start Data Job (Full Load) and look at the log in Extractor (in VS code terminal). Extractor log (shortened and commented) is below, with three parts.

```
-- first part : Extract data from Postgres and create file --
2021-11-19 23:53:26.292 Request with ID pDmB.... received  
2021-11-19 23:53:32.257 Starting extraction for resource: app_users  
2021-11-19 23:53:32.309 Executing query: SELECT * FROM "public"."app_users"  
2021-11-19 23:53:33.007 Response for request i3UZ.... is sent, finished. Stopping keep alive requests  
2021-11-19 23:53:34.170 Loading 100,000 records for table app_users. Total extracted records 100,000  
2021-11-19 23:53:34.432 Loading 54,821 records for table app_users. Total extracted records 154,821  
2021-11-19 23:53:34.433 Total 154,821 records loaded for table app_users  

-- second part : Transfer file to Celonis --
2021-11-19 23:53:34.439 Uploading result files  
2021-11-19 23:53:35.342 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.342 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.343 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.343 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.343 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.344 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.344 Called upload file 98b7856c-c685-4b28-ad33-e86b72483a9f, app_users  
2021-11-19 23:53:35.443 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:37.342 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:38.310 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:38.806 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:39.345 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:39.890 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:40.519 Pushing upsert for table app_users and id 98b7856c-c685-4b28-ad33-e86b72483a9f  
2021-11-19 23:53:41.283 called onSuccess for table app_users with id 98b7856c-c685-4b28-ad33-e86b72483a9f  

-- third part : Insert file content into table in Celonis --
2021-11-19 23:53:41.287 Starting load app_users to target ...  
2021-11-19 23:53:41.641 Waiting for success state ...  
2021-11-19 23:54:00.760 Table app_users loaded ..  
2021-11-19 23:54:00.760 Extraction successful  
```

1. Extractor bring back Extraction Job instruction (connection, table etc.) from Celonis then execute SELECT statement in Postgres to create file. SELECT execution is split to chunks (per 100K record) to minimize resource usage of local machine. 
1. Files stored at my local machine are 7 chunks (seems ~25K record). Those are transferred to Celonis via HTTPS. To control network resource usage, smaller size of files are better, so each files are compressed as parquet file.
1. After 7 files are moved to Celonis, then start INSERT statement to Celonis table. It takes most of time in total Extraction task.

When I looked at job log in Celonis, it is similar to that of Extractor so, it has also three parts as below.

In the second part of log is only one message `Uploading result files`, so you should patiently wait until next message if you need to do 1 day long full load to transfer huge volume of data.

[![image](https://user-images.githubusercontent.com/67397583/142714091-f1e3a6c6-5c99-4bb2-97a0-7220f24c931e.png)](https://user-images.githubusercontent.com/67397583/142714091-f1e3a6c6-5c99-4bb2-97a0-7220f24c931e.png)

By the way, most of staff managing source system said that they were worried about performance down of their source system and thier network, but at least I did not experience such a trouble, even when I extracted ~100M record of SAP FI Documnet (BSEG) etc. 

Kaz 