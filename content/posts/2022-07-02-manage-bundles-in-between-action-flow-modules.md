---
title: "Manage Bundles in between Action Flow Modules"
date: 2022-07-02T07:25:00+09:00
draft: false
tags: ['Studio', 'Action Flow', 'Data Integration']
---

### Convert array to bundles by Iterator module

At last post [Run first Action Flow Scenario](../2022-06-25-run-first-action-flow-scenario) I created HTTP request module to check data job status. Based on the scenario, today I would like to create next module to cancel running data jobs.

Because next module is also HTTP request, it is convenient to clone last module and modify something. I right click the last module, then select `Clone` menu, then new module is generated and connected with previous module. In the next module, I will change below two points based on cURL result of [Operate Celonis from outside by REST API](../2022-06-18-operate-celonis-from-outside-by-rest-api).

- URL : {{5.pool_url}}/jobs/{job_id}/cancel
- Method : POST

By the way, to set each {job_id} in cancel module, unfortunately current direct connection between HTTP request is not enough (currently only first data job can be chosen). I will click gear icon in left lower corner and select `Iterator`. Then drag this module between two HTTP request modules as below screen. At last I will set `Data[]` of previous module to Array. 

[![image](https://user-images.githubusercontent.com/67397583/176899124-dfce27f3-513e-4ee0-8a8b-32d3f109d827.png)](https://user-images.githubusercontent.com/67397583/176899124-dfce27f3-513e-4ee0-8a8b-32d3f109d827.png)

After doing this, HTTP cancel request can select data job id that is split in `Iterator` module.

[![image](https://user-images.githubusercontent.com/67397583/176899992-d394c1ec-3d0f-4857-aaa6-ef86f0c692c9.png)](https://user-images.githubusercontent.com/67397583/176899992-d394c1ec-3d0f-4857-aaa6-ef86f0c692c9.png)

To confirm that point I can run once and confirm cancel request is operated 3 times (same as number of data jobs).

[![image](https://user-images.githubusercontent.com/67397583/176900345-f76d09ef-f202-4a6e-b7d1-2a1080acde78.png)](https://user-images.githubusercontent.com/67397583/176900345-f76d09ef-f202-4a6e-b7d1-2a1080acde78.png)

### Filter data job bundles by their status

Even current scenario is enough for achieving my purpose, but I found that I do not need to post request to all jobs, so it is better to post cancel request to only running jobs.

To do this, I will focus on the connection between Iterator and cancel request. I will click spanner button and select `Setup up a filter` menu. 

At filter setup, I create condition that `status` column in Iterator module is equal to `RUNNING` as below.
 
[![image](https://user-images.githubusercontent.com/67397583/176902745-6fbeeadf-b900-4e96-af9f-a4b106856cdd.png)](https://user-images.githubusercontent.com/67397583/176902745-6fbeeadf-b900-4e96-af9f-a4b106856cdd.png)

I am ready for testing. I will run data job and immediately run scenario to cancel it. The only running job is passed to cancel request and successfully finished (honestly I tried some times to prevent timeout of cancel request, you will set timeout column to 1 second.)

[![image](https://user-images.githubusercontent.com/67397583/176906516-1e337f45-aee9-431b-a88c-137ad5936834.png)](https://user-images.githubusercontent.com/67397583/176906516-1e337f45-aee9-431b-a88c-137ad5936834.png)

In action flow scenario, managing bundles like today is important but not documented in anywhere so I would like to document this today.

Kaz
