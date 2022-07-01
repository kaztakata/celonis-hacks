---
title: "Run first Action Flow Scenario"
date: 2022-06-25T05:05:00+09:00
draft: false
tags: ['Studio', 'Action Flow', 'Data Integration']
---

### Action Flow as HTTP client

From today I would like to introduce Action Flow, that is possible to automate scenario and integrate SaaS systems (also on-premise too). 

You may know that 100 over SaaS systems are registered in Action Flow and easily build your own scenario. Great, but I sometimes felt that I could not find appropriate module from that. How do I fullfill my requirement ? 

Actually Action Flow can be used HTTP client, so what I did by [cURL](https://curl.se/) at last post [Operate Celonis from outside by REST API](../2022-06-18-operate-celonis-from-outside-by-rest-api) is also possible in Action Flow. 

From today I will create my own module by HTTP client in Action Flow. Today is the first time to explain Action Flow, I will create one HTTP module and test it. 

By the way my scenario is same as previous post, monitoring Celonis Data Jobs at midnight. At last post I deregated someone else to monitor Data Jobs, but this kind of task can be automated by Action Flow.

### Create Action Flow and initialization module

Action Flow is one of the node in Studio Package, so I will create Package beforehand and click plus button to create Action Flow under that package. Then enter Action Flow name (and set key automatically) and click Create button.

I can see blank scenario cambus with one blank circle (module) as below screen. I normally choose initialization at first module, so click plus button then enter `tools` then select `Set multiple variables`.

[![image](https://user-images.githubusercontent.com/67397583/175759087-863c1980-4a25-4bae-b39d-975e24a2eb6a.png)](https://user-images.githubusercontent.com/67397583/175759087-863c1980-4a25-4bae-b39d-975e24a2eb6a.png)

In this module, I create two variables `pool_url` and `authorization`. Both are similar to cULR command parameters. Later there variables are reused so it is better to collect to initialization module. After setup, my module is like below.

[![image](https://user-images.githubusercontent.com/67397583/175759324-8cd8f4a6-33bb-4785-879e-f3857f248d4f.png)](https://user-images.githubusercontent.com/67397583/175759324-8cd8f4a6-33bb-4785-879e-f3857f248d4f.png)

```
pool_url : https://{team_id}.training.celonis.cloud/integration/api/pools/{pool_id} 
authorization : AppKey {app_key}
```

### Create own HTTP request module

After setup first module, click right side small circle, then next blank circle (module) is appeared. Select `HTTP` and click `Make a request`. This is HTTP client module.

Same as cURL command, fill URL and Headers as below screen. When entering value, previously setup variables are listed up. Click them so variable value can be used. Additionally tick `Parse response`, and `Evaluate all states as errors (except for 2xx and 3xx)` in advanced settings. Former one is to use response value for further steps (modules), and latter one is to stop scenario if I could not get result. Finally save scenario.

[![image](https://user-images.githubusercontent.com/67397583/175759577-a88dbbfd-e1e2-491e-b36b-5019d40ebd01.png)](https://user-images.githubusercontent.com/67397583/175759577-a88dbbfd-e1e2-491e-b36b-5019d40ebd01.png)

### Run Action Flow as test

It is ready to test this scenario. Click Green button (run once) and wait for few seconds. Then process modules one by one. I can see balloon describing number in each module, that represents processed bundles (number of processing unit) .

Click balloon in HTTP module, then drilldown Bundle 1 and Data. I can see array of each Data Jobs. That is also same appearance as cURL result, but this result can be restructured for next steps to process each record.

[![image](https://user-images.githubusercontent.com/67397583/175760230-77d6a323-4d08-4c9f-8784-fb911d4ee1a5.png)](https://user-images.githubusercontent.com/67397583/175760230-77d6a323-4d08-4c9f-8784-fb911d4ee1a5.png)

In the next post, I would like to add one more HTTP module to cancel data job if still running status.

Finally, Action Flow is derived from Integromat, so [documentation of term](https://www.integromat.com/en/help/basic-terms) of Integromat is helpful if you are struggled with terms like scenario, bundle etc. 

Kaz
