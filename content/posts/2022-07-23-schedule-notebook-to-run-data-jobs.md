---
title: "Schedule Notebook to Run Data Jobs"
date: 2022-07-23T14:49:00+09:00
draft: false
tags: ['Machine Learning', 'Data Integration']
---

From today I would like to use Machine Learning Workbench again. As described at [Start Deep Dive to Machine Learning and Action Flow](../2022-05-07-start-deep-dive-to-machine-learning-and-action-flow) it can manipulate Celonis EMS functions from outside of it. And this manipulation can be scheduled periodically.

Using scheduling function, Machine Learning Workbench can be extension of Data Job scheduler in Data Integration. Today I would like to build simple job scheduler.

### Parallel Data Job execution

You may think your ETL duration is too long so you would like to shorten it as soon as possible. One of the solution is parallel job execution. 

For example, one Data Job is inserting into activity table A, and another Data Job is inserting into activity table B, these two Data Jobs are possible to run parallelly.

By the way, scheduling function in Data Integration is only possible to run Data Jobs one by one. So that scheduling parallel run by Machine Learning Workbench is good alternative.

### Build Jupyter Nootbook

I will create new Jupyter Notebook from Launcher, rename it as `JobExecution.ipynb`. In the first cell I copied script from [Login to Celonis EMS from Jupyter Workbench](../2022-05-14-login-to-celonis-ems-from-jupyter-workbench) to login to Celonis EMS.

In the second cell, I determined Data Pool then execute two Data Jobs as below. When parameter `wait_for_execution` is `False`, then `execute` method just kick first Data Job and do not wait until completion. Then immediately go to next line, second Data Job kick. That means I can run two Jobs parallelly.

```python
# Pool / Job IDs are in my environment
pool = celonis.pools.find('3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1')
datajob1 = pool.data_jobs.find('0bae5839-6921-46dc-8d0f-3294ecd6b3fd')
datajob2 = pool.data_jobs.find('7355c620-0c46-49b5-b466-b31aad09f6ec')
datajob1.execute(wait_for_execution=False) # kick Data Job and immediatelly go to next line 
datajob2.execute(wait_for_execution=False) # kick Data Job and immediatelly go to next line
```

In the third cell, I will wait for completion of two Jobs. I use `check_data_job_execution_status`
method that is used at [Operate Celonis from outside by REST API](../2022-06-18-operate-celonis-from-outside-by-rest-api).
If any Job is still `RUNNING` then wait for 5 seconds then check status again. 

```python
from time import sleep
while True:
    if any([j['status']=='RUNNING' for j in pool.check_data_job_execution_status()]):
        print('Job executing ...')
        sleep(5)
    else:
        print('Job finished')
        break
```

### Schedule Jupyter Notebook

After saving notebook, I will select Manage > Scheduling, then move to Schedule screen. Click New Schedule button, then choose ML workbench and Notebook as below screenshot. Please note that notebooks can be detected in home directory or first subdirectory.

[![image](https://user-images.githubusercontent.com/67397583/180594101-e7405c94-848e-43fc-81b0-e45cf3d76bd9.png)](https://user-images.githubusercontent.com/67397583/180594101-e7405c94-848e-43fc-81b0-e45cf3d76bd9.png)

To check whether notebook works correctly, I click three dots button and select `Run`. Then immediately notebook is started and check the result.

[![image](https://user-images.githubusercontent.com/67397583/180594257-29e1879e-6e2f-42a9-9877-c97e503553bb.png)](https://user-images.githubusercontent.com/67397583/180594257-29e1879e-6e2f-42a9-9877-c97e503553bb.png) 

I can receive notebook error notification when I click `Subscribe`. And I can also receive notification if I setup `Alert` of Data Job. 

Kaz
