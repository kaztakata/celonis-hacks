---
title: "Operate Celonis from outside by REST API"
date: 2022-06-18T17:59:00+09:00
draft: false
tags: ['Machine Learning', 'Data Integration']
---

### About REST API

Until last post, I explained some of API use cases (export Pool program and configure data job alert) using Pycelonis script from ML Workbench. Is is something Celonis "internal" operation. By the way, as I mentioned in [Login to Celonis EMS from Jupyter Workbench](../2022-05-14-login-to-celonis-ems-from-jupyter-workbench), I can call API from outside of Celonis. Generally this kind of API using HTTP request is called [REST API (or RESTful API)](https://www.techtarget.com/searchapparchitecture/definition/RESTful-API#:~:text=A%20RESTful%20API%20is%20an,deleting%20of%20operations%20concerning%20resources.)

Some cloud services including Celonis are open their REST API to interact with other applications. It is convenient that I can learn standard REST API rules, then I can handle different cloud services using REST API.

As first step, today I would like to operate Celonis from my local computer without login to Celonis.

### Use case

Imagine that I setup quite long data extraction job and source system (SAP ECC etc.) will shutdown at midnight, so if data job is still running at the timing of shutdown, I should cancel data job for safer shutdown. 

By the way, myself do not monitor this job but someone else will do it. That person monitoring midnight batch job is managing not only Celonis but many systems. So that person do not have user account of Celonis. Even this situation that person should cancel data job. How to do that ?

First I should register Application key for that person (means application of that person) reffering to [Limit permissions of API token to minimize risk](../2022-05-21-limit-permissions-of-api-token-to-minimize-risk). Of course permissions of this key are limited to relevant Data Pool.

Second I will write simple command to look at data job exextion status and to cancel it. Let's look at the detail.

I assume, I can not ask that person to install any kind of application. That person is using Windows, normally Python is not installed. So Pycelonis is not the option.

### Monitor Data Job execution status

I found that Windows PowerShell is installed by default, and [cURL](https://curl.se/) which can send HTTP request is also available as `curl.exe` command in PowerShell. So I decided to use `curl.exe` for this task (You can confirm by entering `curl.exe --version` whether it is installed).

Next step is to find endpoint (URL) of two things, displaying data job status and cancel data job. Former one is documented in Pycelonis as [check_data_job_execution_status](https://celonis.github.io/pycelonis/1.7.0/reference/celonis_api/event_collection/data_pool/#celonis_api.event_collection.data_pool.Pool.check_data_job_execution_status) method.

That said `GET: /integration/api/pools/{pool_id}/logs/status`. Adding my team URL to this endpoint and fill in pool_id is good enough.

Also, to authenticate to Celonis, Authorization header is required as I showed in [Observe HTTP request in Pycelonis login script](../2022-05-28-observe-http-request-in-pycelonis-login-script). In cURL header is set by `-H` option. Finally cURL command is

```
curl.exe https://{team_id}.training.celonis.cloud/integration/api/pools/{pool_id}/logs/status -H "Authorization: AppKey {app_key}"
```

I entered command to PowerShell, then return the status of each Data Job as below screen. In this case second job `0bae5839-6921-46dc-8d0f-3294ecd6b3fd` is still `RUNNING` so I should cancel this one.

[![image](https://user-images.githubusercontent.com/67397583/174424482-93d60f3d-9780-42d9-8a8d-0ff1ddea8253.png)](https://user-images.githubusercontent.com/67397583/174424482-93d60f3d-9780-42d9-8a8d-0ff1ddea8253.png)

### Send Cancel request of Data Job execution

Cancel request is also sent by cURL. This request is not documented in Pycelonis, so I will check endpoint by `Stop Execution` button. `POST: /integration/api/pools/{pool_id}/jobs/{job_id}/cancel` is the endpoint I confirmed.

This case is the first one I should specify HTTP request method `POST` by cURL `-X` option (until then method was automatically determined and it was working). Finally cURL command is.

```
curl.exe -X POST https://{team_id}.training.celonis.cloud/integration/api/pools/{pool_id}/jobs/{job_id}/cancel -H "Authorization: AppKey {app_key}"
```

I entered command to PowerShell, then waiting for few seconds and return new prompt without any message. So I entered first command again and confirmed status is actually changed to `CANCEL`.

[![image](https://user-images.githubusercontent.com/67397583/174430359-e3119eef-3e2e-4f98-93c2-b10f9e4bbed5.png)](https://user-images.githubusercontent.com/67397583/174430359-e3119eef-3e2e-4f98-93c2-b10f9e4bbed5.png)

Even above explanation is quite simplified, but I believe you can imagine how REST API is convenient.

Kaz
