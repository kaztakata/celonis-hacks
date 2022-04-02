---
title: "Setup Dependent Endpoint in Extractor Builder"
date: 2022-01-29T09:17:16+09:00
draft: false
tags: ['Data Integration','Extractor Builder']
---

In the last post [Configure Endpoint for Suitable Extraction](../2022-01-22-configure-endpoint-for-suitable-extraction), I configured Endpoint in Extractor Builder to suit my business requirements, and still there are points to extract change history of issues, and to extract data by Delta Load option. Today I would like to setup regarding change history using Dependent Endpoint in Extractor Builder.

At first how do I extract change history of Planio Issue ? Again I looked at [Planio Documentation](https://plan.io/api/#issues) and found I can get single issue with journals (meaning change history in Planio). To do this, request `GET /issues/123.[json|xml]` with `journals` to `include` parameter. To identify issue ID (`123` part at above URI), issue list I created before is required as prerequisite. So I should at first get issue list then iterate to get journals by each issue ID.

In Celonis Extractor Builder it is possible to fullfill above by adding Dependent Endpoint. At 4 Define Endpoints screen click three dots of Endpoint `issues` I created before and cilck add Dependent Endpoint. 

In the first step I named new Endpoint as `journals`. In the second step Configure Dependency, I choose `id` as Depends on Column. In the third step Configure Request, set `{Connection.API_URL}/issues/{Dependency.id}.json` to URL, to iterate by `id` of each issue. Then I add New Request Parameter `include` with value `journals`.

[![image](https://user-images.githubusercontent.com/67397583/151640026-d5a98c58-bc9e-4aff-957e-dbd212852e5a.png)](https://user-images.githubusercontent.com/67397583/151640026-d5a98c58-bc9e-4aff-957e-dbd212852e5a.png)

In the fourth step Configure Response, I determined `journals` as target table and click Build Response same as previous post. But this time I did not get response structure by default, so I temporarily changed `{Dependency.id}` in request URL to actual ID (e.g. `8`) then I can successfully get response structure.

After saving configuration above, I would like to extract change histor data from Planio. In the extraction setting, I can see `journals` table is displayed under `issues` table, so save this update and then execute data job.

[![image](https://user-images.githubusercontent.com/67397583/151641508-3fbf326e-bb5d-411a-87f2-b94799261bb5.png)](https://user-images.githubusercontent.com/67397583/151641508-3fbf326e-bb5d-411a-87f2-b94799261bb5.png)

[![image](https://user-images.githubusercontent.com/67397583/151641868-a48c9866-ee2e-4c8b-9486-7195a812aafd.png)](https://user-images.githubusercontent.com/67397583/151641868-a48c9866-ee2e-4c8b-9486-7195a812aafd.png)

I can additionally get two tables `journals` and `journals$details`. `journals` table contains journal ID, event time, user ID, and relevant issue ID. On the other hand `journals$details` table contains changed column and old / new value of this column. These two tables are used at once like SAP `CDHDR` / `CDPOS` tables.

[![image](https://user-images.githubusercontent.com/67397583/151642191-9e65960e-8b72-4730-96b5-46562ead7e51.png)](https://user-images.githubusercontent.com/67397583/151642191-9e65960e-8b72-4730-96b5-46562ead7e51.png)

Next time I would like to setup to fullfill Delta Load requirement, then complete extraction setting.

Kaz
