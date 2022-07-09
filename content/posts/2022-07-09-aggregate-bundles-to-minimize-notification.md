---
title: "Aggregate Bundles to Minimize Notification"
date: 2022-07-09T18:19:00+09:00
draft: false
tags: ['Studio', 'Action Flow', 'Data Integration']
---

### Integrate cancel operation bundles to one 

In the last [Manage Bundles in between Action Flow Modules](../2022-07-02-manage-bundles-in-between-action-flow-modules) I successfully automated Data Job cancel operation. To do this, I used `Iterator` module to split one bundle of Data Pool to multiple bundles of Data Jobs, and filtered bundles to specify running Data Jobs. In the end, bundles of running Data Jobs are selected and cancelled.

Continued from last post, goal of today is to notify result of automated operation by email (my Gmail), because I can not check automated operation is done correctly at midnight based on initial scenario. I will open email next morning then confirm automation has done correctly.

By the way, imagine I handle hundreds of Data Jobs and I should receive email, I do not want to receive multiple emails. Instead I would like to receive one email describing multiple cancel operation. 

To do this, I need to integrate multiple cancel operation bundles to one bundle again then create email based on integrated result. OK let's start.

### Describe result of each cancel operation

I reviewed previous result of HTTP cancel request, and found that there is no output even if cancel has done. So I add one more HTTP request to GET Data Job, including current status and its name. I cloned cancel request module and changed like below screen (remove `/cancel` from URL, change method to `GET`).

[![image](https://user-images.githubusercontent.com/67397583/178089301-0bd02bc0-21fc-4479-b5a8-0113b2a9230e.png)](https://user-images.githubusercontent.com/67397583/178089301-0bd02bc0-21fc-4479-b5a8-0113b2a9230e.png)

I run once then get data of current Data Job like below, actually status is changed to `CANCEL`.

```
"data": {
            "id": "0bae5839-6921-46dc-8d0f-3294ecd6b3fd",
            "name": "main job",
            "dataPoolId": "3d8a2621-f9c0-456a-9b2d-9bb8c0a9fcd1",
            "dataSourceId": "3702de43-b181-4b5b-90da-87e056ce7616",
            "timeStamp": 1642237998799,
            "status": "CANCEL",
            "currentExecutionId": null,
            "dagBasedExecutionEnabled": null
        },
```

### Aggregate bundles to describe email text

Current last module is still Data Job level bundles so it can be multiple. To pass single bundle to Gmail module, I choose `Tools > Text Aggregation` as next module. In this module I should choose `Source Module`. This `Source` means which bundle (module) do you want to aggregate. Now I would like to aggregate to Pool level bundle, so I choosed `Iterator` module (or previous modules are fine too).

Then I need to determine which text is generated from each Data Job bundles. I need to confirm current status, so I check whether status is `CANCEL` then output text like below. If multiple bundles are processed, output to new line per bundle.

```
Data Job [{{16.data.name}}] is {{if(16.data.status = "CANCEL"; "successfully cancelled."; "failed to cancel.")}}

```

[![image](https://user-images.githubusercontent.com/67397583/178089547-cfe610c2-9e21-437a-9718-3461ac45a3cb.png)](https://user-images.githubusercontent.com/67397583/178089547-cfe610c2-9e21-437a-9718-3461ac45a3cb.png)

### Send aggregated status by email 

Finally I add email (this time Gmail) module to send message. It depends on which mailer is used, but normally I need to login to email service then allow Celonis to operate email service. After that this authorization is stored as `Connection`. As below screen I set `To`, `Subject`, and `Content` that is described in previous aggregator module.

[![image](https://user-images.githubusercontent.com/67397583/178099193-abd9feff-5b3e-4e56-a084-ed594612903c.png)](https://user-images.githubusercontent.com/67397583/178099193-abd9feff-5b3e-4e56-a084-ed594612903c.png)

When I run total scenario, I got one mail in my inbox as expected. You can check whether it is only one email by removing filter after `Iterator` module.

[![image](https://user-images.githubusercontent.com/67397583/178099267-32916f4c-bd65-44fb-9017-0d502f1c5761.png)](https://user-images.githubusercontent.com/67397583/178099267-32916f4c-bd65-44fb-9017-0d502f1c5761.png)

### Deploy scenario as production one

Scenario is tested so it is time to deploy to production. I should do 1) Publish Package, 2) Set schedule, and 3) Activate scenario. 

Regarding 2) I should click clock icon in the first module when you can see `Published` at top left. First I can try by `At regular intervals` and set 1 minutes. After that wait for 1 minute to check this scenario is working correctly. After that change to apporopriate timing. 

[![image](https://user-images.githubusercontent.com/67397583/178099647-8e4ecbce-9357-43da-b98b-f811358d3759.png)](https://user-images.githubusercontent.com/67397583/178099647-8e4ecbce-9357-43da-b98b-f811358d3759.png)


Draft version of scenario is completed, but I would like to improve some points next time.

Kaz
