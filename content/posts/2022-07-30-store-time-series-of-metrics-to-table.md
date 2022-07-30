---
title: "Store Time Series of Metrics to Table"
date: 2022-07-30T11:13:24+09:00
draft: false
tags: ['Machine Learning', 'Data Integration', 'Process Analytics','Studio','Analysis']
---

At last post [Schedule Notebook to Run Data Jobs](../2022-07-23-schedule-notebook-to-run-data-jobs) I showed you how to schedule Data Job using Machine Learning Workbench.
Of course, not only Data Job but any kind of tasks can be scheduled by Python script. Today I would like to schedule backup some kind of data to table by Machine Learning Workbench.

### Store time series of metrics

Imagine I am running Celonis EMS for months and I already established dash board Aalysis to observe some KPIs (metrics). Data is continously updated so metrics is moved up and down respectively. So that I think I would like to see the time series of metrics, but old metrics is based on old data set so it is not possible to calculate it again.

Based on above situation, I will store metrics after each Data Model reload, then later use this time series of metrics for another Analysis component.

To simplify discussion from here, I picked up one simple metrics - average throughput time from process start to process end. Using Planio Data Model I already build at [Construct My First Data Model](../2022-04-16-construct-my-first-data-model), metrics is represented as below. This PQL is set to single KPI component in my Analysis.

```sql
AVG(
  DATEDIFF(
    dd,
    PU_FIRST(issues, _cel_pl_activities.EVENTTIME),
    PU_LAST(issues, _cel_pl_activities.EVENTTIME)      
  )
)
```

### Pick up metrics from Data Model

To pick up metrics from Machine Learning Workbench, first I would like to select above component, then get calculated value to local variable.

I copy previously created notebook and rename it as `StoreMetrics.ipynb`. I do not touch first cell regarding login to Celonis EMS. Then replace second and third cells from now.

In the second cell, I write below script. At first line I select my package then analysis. Using this analysis object, in second line I find sheet and component respectively. At third line, calculate PQL in this component (same result as just displaying in the browser) and get result to table data called [DataFrame](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html), but this component is sigle KPI so result is 1 x 1 table. At last line pickup first column, first row of table and assign value to local variable `tpt`.

```python
analysis = celonis.default_space.packages.find('your package').analyses.find('your analysis')
component = analysis.draft.sheets.find('your sheet').components.find('your component')
df = component.get_data_frame(True) # calculate component and get table data
tpt = df[df.columns[0]][0] # pickup first column, first row of table
```

### Insert metrics into Table

I will write script below in third cell to insert average throughput time into new table `TPT_TS` in my Data Pool that has two columns `EVENTTIME` and `TPT` to store time series of average thruoghput time. 

I will call `CREATE TABLE` and `INSERT` SQL statement from Machine Learning Workbench, same as doing it in Transformation. To do this I choose arbitrary transformation from existing transformations, then from that object call `execute_from_workbench` method. This method does not affect to existing SQL statement in this transformation. 

```python
pool = celonis.pools.find('your pool')
transformation = pool.data_jobs.find('your data job').transformations[0]
transformation.execute_from_workbench('CREATE TABLE IF NOT EXISTS TPT_TS(EVENTTIME TIMESTAMP, TPT FLOAT)')
transformation.execute_from_workbench(f"INSERT INTO TPT_TS VALUES (NOW(), {tpt})")
```

By the way `CREATE TABLE` is happened at first time only, so it is fine to create table outside of Machine Learning Workbench, and omit `CREATE TABLE` statement from below script.

Finally save `StoreMetrics.ipynb` notebook and schedule it after Data Model Reload has done. Then I can backup time series like below.

|EVENTTIME|TPT|
| --- | --- |
|2022-07-30 01:47:46|2.28148748285322|
|2022-08-01 01:22:08|1.98482858748148|

Like today it is possible to run arbitrary SQL statement from Machine Learning Workbench, so I love to use this functoin for mass database operation. 

Kaz
