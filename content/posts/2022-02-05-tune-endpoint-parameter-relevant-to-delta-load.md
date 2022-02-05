---
title: "Tune Endpoint Parameter Relevant to Delta Load"
date: 2022-02-05T10:05:51+09:00
draft: false
tags: ['Event Collection','Extractor Builder']
---

Until last post [Setup Dependent Endpoint in Extractor Builder](../2022-01-29-setup-dependent-endpoint-in-extractor-builder), I prepared endpoints of both Planio Issues and their journals. Today I would like to tackle final setup of extractor to deal with Delta Load option.

Referring to the [Planio Documentation](https://plan.io/api/#issues), `updated_on` column exists for filtering Issues. This timestamp column is updated when creating and updating relevant issue, so it is appropriate column for Delta Load. Open Celonis Extractor builder then go to 4 Define Endpoints. In the 2 Configure Request section click Request Parameters, then click New Request Parameter button. As below screen I setup `updated_on` parameter. This time filtering value is provided from already extracted data, so I clicked second radio button. Finish Endpoint setting then move to extraction setting.

[![image](https://user-images.githubusercontent.com/67397583/152627750-d567e1ff-d94f-4ca1-b133-828d860b6cf5.png)](https://user-images.githubusercontent.com/67397583/152627750-d567e1ff-d94f-4ca1-b133-828d860b6cf5.png)

In the extraction setting, I created new parameter `today` and set today's date (this parameter will be updated from what triggers data job, in my case ML workbench). Then write Delta Filter Statement as `updated_on = <%=today%>`. You may think the best way is to get maximum date of `updated_on` and compare as `updated_on > <%=maximum date of updated on%>`, but for the moment I could not do that. I hope this function will work later.

I updated one of issue in Planio and started data job, then I can extract only this issue as expected. OK, delta load seems to work correctly, but if I plan to kick data job many times per day (e.g. hourly), I will extract same issues and journals many times. Below screen shows that same journal `32` is extracted twice by two extraction jobs respectively. I expect to overwrite first record by second one, but how to do it ?

[![image](https://user-images.githubusercontent.com/67397583/152629985-29f6542a-0b38-46c6-86a2-e1b297aec775.png)](https://user-images.githubusercontent.com/67397583/152629985-29f6542a-0b38-46c6-86a2-e1b297aec775.png)

I should go back to extractor builder and setup Table Configuration in Configure Response. In this setting I will tick primary key button against `id` column of `journals` table as below. After that `id` behaved as primary key at extraction data job then update existing record based on `id`.

[![image](https://user-images.githubusercontent.com/67397583/152630192-4dc01927-e88d-49bc-aaa2-585441c65eb0.png)](https://user-images.githubusercontent.com/67397583/152630192-4dc01927-e88d-49bc-aaa2-585441c65eb0.png)

Same setting is required for `journals$details` table, and this time not only `id` but `property` and `name` are also primary keys (this record is created per changed column of each journal).

[![image](https://user-images.githubusercontent.com/67397583/152630287-e61ddbc3-aab3-4764-be8c-db50e624fac8.png)](https://user-images.githubusercontent.com/67397583/152630287-e61ddbc3-aab3-4764-be8c-db50e624fac8.png)

I reset record by Full Load then tried Delta Load again. This time journal `32` is overwritten by delta load correctly.

It is long to explain extraction setting, but I intended to explain next step, transformation. From next post I will do that.

Kaz

---

I would like to share my [configuration file](../../examples/planio_extractor_20220205.json) to you. If you would like to just use this, you will click New Extractor menu and Import from file, then use my json file.

---
