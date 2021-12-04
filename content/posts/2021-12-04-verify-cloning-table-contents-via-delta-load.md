---
title: "Verify Cloning Table Contents via Delta Load"
date: 2021-12-04T12:49:44+09:00
draft: false
tags: ['Event Collection','Extractor','Extraction','Transformation','Data Job']
---

Following last week's [Minimize Extraction Time by Delta Load Option](../2021-11-27-minimize-extraction-time-by-delta-load-option), today I would like to insert new record to Postgres table then try Delta Load again to extract it. To do this, I will start from operating [pgAdmin](https://www.pgadmin.org/), that is already ready for my loal machine after `docker-compose`.

First step is to enter [localhost:5050](http://localhost:5050) to my browser, then at the login screen enter `pgadmin@celonis.cloud` as email and `pgadmin` as password then click login button. 

In the welcome screen, click Add New Server button and enter Postgres connection information as below screenshot (same information for Celonis to connect to Postgres database, see [Connect to Celonis and Bring Back Instruction](../2021-11-13-connect-to-celonis-and-bring-back-instruction)), then save it.

[![image](https://user-images.githubusercontent.com/67397583/144696270-28c5595d-2e81-4750-b724-879ce35a66b9.png)](https://user-images.githubusercontent.com/67397583/144696270-28c5595d-2e81-4750-b724-879ce35a66b9.png)

After saving server information, Postgres can be operated via pgAdmin. In the left panel, go to Servers > postgres > Databases > postgres > Schemas > public > tables. Then right click Tools > Query Tool, then Query Editor is opened. I can verify this database is same as what I extracted data from Celonis, by entering `SELECT COUNT(*) FROM app_users;` then get value 154821.

[![image](https://user-images.githubusercontent.com/67397583/144696601-b2585927-f89b-4e65-8f4f-810ed2b1026e.png)](https://user-images.githubusercontent.com/67397583/144696601-b2585927-f89b-4e65-8f4f-810ed2b1026e.png)

Now I am ready for inserting new record. Let's enter `INSERT INTO app_users (firstname, lastname) VALUES ('Celonis','Hacks');` then execute SQL. To confirm new record, enter `SELECT * FROM app_users ORDER BY id DESC LIMIT 5;` then I can see new id `154822` at first row.

Finally I will go to Celonis to execute extraction job with Delta Load option, then only new record is extracted from Postgres as I expected. Below is the job logs.

```
2021-12-04 13:34:47 Extraction successful
2021-12-04 13:34:46 Table app_users loaded ..
2021-12-04 13:34:24 Inserting extracted data into the Data Pool (this may take a while)
2021-12-04 13:34:24 Starting load app_users to target ...
2021-12-04 13:34:24 Uploading result files
2021-12-04 13:34:23 Total 1 records loaded for table app_users
2021-12-04 13:34:23 Loading 1 records for table app_users. Total extracted records 1
2021-12-04 13:34:23 Setting filter to: id > 154821
2021-12-04 13:34:22 Starting extraction for resource: app_users
```

To confirm new record I executed `SELECT` statement in Celonis transformation as below screen. The difference between Postgres and Celonis is the last column `_CELONIS_CHANGE_DATE`, that is UTC timestamp when this record is cloned (extracted) to Celonis (not relevant to when this record is inserted in Postgres). `_CELONIS_CHANGE_DATE` column is automatically generated at extraction timing.

[![image](https://user-images.githubusercontent.com/67397583/144697298-d3924858-3498-4a1c-a5d4-6c1f827e999e.png)](https://user-images.githubusercontent.com/67397583/144697298-d3924858-3498-4a1c-a5d4-6c1f827e999e.png)

In real situation, not only inserting record but updating and deleting record should be considered. If there is some sorting column to use for updating scenario (e.g. last updated timestamp column) it is possible to use Delta Load. Only Full Load is available to synchronize table contents if not such a update column or if record is deleted.

One more case to consider for extraction is adding or deleting table column. This topic is discussed next week.

Kaz