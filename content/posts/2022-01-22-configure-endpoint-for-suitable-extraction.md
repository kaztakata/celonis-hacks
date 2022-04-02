---
title: "Configure Endpoint for Suitable Extraction"
date: 2022-01-22T08:08:39+09:00
draft: false
tags: ['Data Integration','Extractor Builder']
---

In the last post [Connect to Source System via REST API](../2022-01-15-connect-to-source-system-via-rest-api), I shared how to set up Extractor Builder and extracted Issue from Planio. It was shortest path to be avaiable for extraction job, so it is not enough for production job. Today I would like to configure Endpoint in Extractor Builder to resolve problems I experienced.

First problem I faced is upper limit of extraction data. Some day I found that I could not get issue record until 25. This is described at [Collections and Pagination in Planio API documentation](https://plan.io/api/#collections-and-pagination), that said I need to setup Pagination to extract 1-25th record at first, then 26-50th record. Endpoint example is written as `https://example.plan.io/users.xml?limit=10&offset=20` to get 10 users starting at the 21st. So I would like to setup this to my Endpoint setting.

Open Celonis Extractor builder then go to 4 Define Endpoints. Move to 2 Configure Request and click Pagination Method. As below screen I choose Limit and Offset Pagination then fill in parameters as below. I can review my setting by looking at PAGINATION PREVIEW. In my result `offset` and `limit` are opposite to eample above, but it works correctly. 

```
1.Request:{Connection.API_URL}/issues.json?offset=0&limit=25
2.Request:{Connection.API_URL}/issues.json?offset=25&limit=25
3.Request:{Connection.API_URL}/issues.json?offset=50&limit=25
```

[![image](https://user-images.githubusercontent.com/67397583/150613164-64b8533f-34ba-44e8-a6c8-abd73f2c2ddc.png)](https://user-images.githubusercontent.com/67397583/150613164-64b8533f-34ba-44e8-a6c8-abd73f2c2ddc.png)

Second problem is status of issues. I found I could only extract open issues by default, but I am required to extract closed ones too for analysis purpose. Looking at [Listing issues section in Documentation](https://plan.io/api/#issues), there are possible parameters to filter issues including `status_id`. I should use `*` (means both open and closed). 

In the 2 Configure Request section click Request Parameters, then click New Request Parameter button. As below screen I setup `status_id` parameter as `*`, then save it.

[![image](https://user-images.githubusercontent.com/67397583/150616324-dca64e78-986f-411f-a190-1be0046e62b8.png)](https://user-images.githubusercontent.com/67397583/150616324-dca64e78-986f-411f-a190-1be0046e62b8.png)

Third one is not problem, but I am not required to get statistics table. To review the sample response structure at 3 Configure Response, I just require from line 5 `"issues" : [` to line 40 `]` (that is described at NESTED TABLES in right side). To filter the part of response structure, I choose Response Root to `"issues"` as below screen (additionally I change Target Table name to simply issues).

[![image](https://user-images.githubusercontent.com/67397583/150617221-568dc7fb-6d5a-418a-a83c-abd3fc1002b7.png)](https://user-images.githubusercontent.com/67397583/150617221-568dc7fb-6d5a-418a-a83c-abd3fc1002b7.png)

Let's check my change of Endpoint. I will go to data job then create new extraction, then first I found that table name is changed to `issues`. After extraction job completion, I checked count of record in table `issues` and it is more than 25 (if you need to upload multiple issues to Planio at once, you can use [Import Issues](https://plan.io/import-issues/)). Finally I checked `status$name` column and see both open and closed status exist.

Actually there are two more challenges. one is to extract change history of issues, the others is to extract data by Delta Load option based on `updated_on` column to minimize extraction effort (refer to [Minimize Extraction Time by Delta Load Option](../2021-11-27-minimize-extraction-time-by-delta-load-option)). There are discussed in the next posts.

Kaz