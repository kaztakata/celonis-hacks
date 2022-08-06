---
title: "Manipulate Transformatoin Scripts Collectively"
date: 2022-08-06T10:13:42+09:00
draft: false
tags: ['Machine Learning', 'Data Integration', 'Transformation']
---

At described in [Start Deep Dive to Machine Learning and Action Flow](../2022-05-07-start-deep-dive-to-machine-learning-and-action-flow) it is possible to manipulate Celonis EMS function by Machine Learning Workbench. Function relevant to EMS data is already discussed at last post [Store Time Series of Metrics to Table](../2022-07-30-store-time-series-of-metrics-to-table). Machine learning Workbench also enables to manipulate programs in EMS for developers. Today I would like to demonstrate some of functions to manipulate Transformation (SQL) script developed in Data Integration.

### Motivation to manipulate SQL scripts

I have some time to create and maintain SQL script in Transformations for adding new Activity or improving new features, like deal with multi languages, adjust timezone to UTC etc. Also I need to refactor scripts to simplify code sets, for example group similar logic in various script to one VIEW statement. In these situation, I need to handle SQL scripts described in various data jobs and transformations. Of course it is possible to open transformation one by one, but it is better to create small program to mass operation against all SQL scripts, because small (Python) program is quickly constructed and executed in 5 minutes etc.

### Replace keyword in multiple SQL scripts

First live example is to replace some of keyword spreaded to various transformations. Imagine I would like to rename column of activity table from `ACTIVITY_EN` to `ACTIVITY`,because this column will be stored not only English Activity but other languages too. This keyword is used in `CREATE TABLE` and multiple `INSERT` statements. To deal with this task, first I picked up transformation script from EMS then replace keyword and push back to EMS again. After login to Celonis EMS (refer to last post), I will write below Python script. I use `replace` method ([reference](https://docs.python.org/3/library/stdtypes.html?highlight=str%20replace#str.replace)) against SQL script in this simple case, but I can also use [regular expression](https://docs.python.org/3/library/re.html?highlight=regular%20expression) to deal with complex cases too.

```python
dry_run = True # if true, no push back but just show replacement simulation
pool = celonis.pools.find('3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1')
datajob = pool.data_jobs.find('0bae5839-6921-46dc-8d0f-3294ecd6b3fd')

for transformation in datajob.transformations:
    statement = transformation.statement # pick up SQL script
    statement = statement.replace('ACTIVITY_EN', 'ACTIVITY') # replace method against text (SQL script)
    if not dry_run:
        transformation.statement = statement # push back to Celonis EMS
    print(statement)
```

### Move transformations from one datajob to another datajob

Second case is to move transformations from one datajob to another datajob. It is to reconstructure (merge or split) datajobs for improving performance. Below is example to just copying transformations in one datajob to another for merging two datajobs. I use `create_transformaton` method in datajob object ([reference](https://celonis.github.io/pycelonis/1.7.1/reference/celonis_api/event_collection/data_job/#celonis_api.event_collection.data_job.DataJob.create_transformation)) by passing two parameters - name and statement. To do this, I can copy transformation name and script from one to another.

```python
datajob_from = pool.data_jobs.find('cbff607e-c376-4c84-9c97-ac75ea86e19f')
datajob_to = pool.data_jobs.find('da5a5e5e-b01a-465a-bc3f-1e504f8043ef')

for t in datajob_from.transformations:
    datajob_to.create_transformation(t.name, t.statement)
```

Like using this method, I already established version control programs, that can extract programs to outside of EMS (GitHub) then load it to another EMS team example another. It helps my team to minimize distributing our template program to multiple teams.

Kaz