---
title: "Execute Periodic ETL Automatically"
date: 2022-04-30T08:54:10+09:00
draft: false
tags: ['Data Integration','Extraction', 'Transformation','Data Model', 'Content-CLI', 'Machine Learning']
---

Until last post [Validate Data Model by Studio Analysis](../2022-04-23-validate-data-model-by-studio-analysis), I completed creating ETL programs in Data Integration. But this is still test product because this programs are executed manually by operator. In this post, I would like to share the final piece of Data Integration, how to run ETL programs automatically after production release. 

There are several ways to achieve automatic ETL, for example in my production system I am using scheduler in Machine Learning Workbench to trigger Jupyter Notebook, then operate Data Integration via [Pycelonis (Python API for Celonis EMS)](https://celonis.github.io/pycelonis). I would like to create posts regarding Pycelonis later, but today I do not use it because it takes a lot to explain Pycelonis. Instead today I use build in Scheduler in Data Integration, it is easy to construct and enough to use in production too.

If you remember discussion in [Tune Endpoint Parameter Relevant to Delta Load](../2022-02-05-tune-endpoint-parameter-relevant-to-delta-load), I created Data Pool Parameter `today` for filtering extracting scope. This parameter should be updated to current time before extraction, so I will introduce new (global) `initial job`. SQL in this job is just getting current time like below.

```
DROP TABLE _CEL_CURRENT_TIME;
CREATE TABLE _CEL_CURRENT_TIME AS
SELECT NOW()
;
```

Next I would like to fetch this value for extraction. Open Data Pool Parameters and update `today` setting to get value from above table `_CEL_CURRENT_TIME`. Then go to Extraction setting and change parameter `today` to use value of Data Pool Parameter.

[![image](https://user-images.githubusercontent.com/67397583/166088925-8e1a9814-5015-40fd-917c-f716564ef4cd.png)](https://user-images.githubusercontent.com/67397583/166088925-8e1a9814-5015-40fd-917c-f716564ef4cd.png)

[![image](https://user-images.githubusercontent.com/67397583/166088988-945945e8-5e72-46f1-95f9-8243564076fd.png)](https://user-images.githubusercontent.com/67397583/166088988-945945e8-5e72-46f1-95f9-8243564076fd.png)

OK then I would like to collect extraction and transformations to `main job`, and global view creation and Data Model Reload to `final job` like below.

[![image](https://user-images.githubusercontent.com/67397583/166089091-c952cd91-4629-4ffa-a0d0-1151b5f4cafe.png)](https://user-images.githubusercontent.com/67397583/166089091-c952cd91-4629-4ffa-a0d0-1151b5f4cafe.png)

[![image](https://user-images.githubusercontent.com/67397583/166089119-78f0fcb7-9711-43dd-8e47-750798ae61dc.png)](https://user-images.githubusercontent.com/67397583/166089119-78f0fcb7-9711-43dd-8e47-750798ae61dc.png)

I am ready for scheduling data jobs. Click Scheduling menu and create new one. From bottom to top in below screen I setup scheduling. First I will choose data jobs which is executed periodically. I choose three jobs above and order them to run `initial job`, `main job` and `final job` sequentially. Second I will choose Scheduling Plan to Hourly. If I would like to execute more frequently I should use Custom Cron and setup parameter ([reference](https://www.netiq.com/documentation/cloud-manager-2-5/ncm-reference/data/bexyssf.html#beykoa4)). Third I will enable scheduling, and switch to Delta Load. Final setup is as below screen.

[![image](https://user-images.githubusercontent.com/67397583/166091175-e2f57443-feee-4a8e-832d-4e8afdcd1ae4.png)](https://user-images.githubusercontent.com/67397583/166091175-e2f57443-feee-4a8e-832d-4e8afdcd1ae4.png)

Then just wait for one hour and see if job is executed successfully. If you would like to monitor background run, click three dots of data job or data model and setup notification (or subscription) setting. Scheduler itself does not have notification function.

That is it. I can conclude Data Integration series now. And I am thankful to you to continue reading my posts for one year. I would like to share my ideas with you in second year too.

Kaz

---

I would like to share my [Data Pool File](../../examples/planio_pool_20220430.json) to you. If you would like to copy my pool to your environment, run content-CLI in your Machine Learning Workbench like `content-cli push data-pool -f planio_pool_20220430.json -p mytraining`. Please see [Share my Analysis by Content-CLI](../2021-08-14-share-my-analysis-by-content-cli) for detail of content-CLI. 

Extactor is excluded from this copy, so please see [Tune Endpoint Parameter Relevant to Delta Load](../2022-02-05-tune-endpoint-parameter-relevant-to-delta-load) to upload Extractor and replace connection to that with uploaded Extarctor.

---
