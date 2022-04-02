---
title: "Connect to Source System via REST API"
date: 2022-01-15T10:27:09+09:00
draft: false
tags: ['Data Integration','Extractor Builder']
---

At previous post [Prepare Source System to Generate Event Log](../2022-01-08-prepare-source-system-to-generate-event-log), I prepared Planio as source system for this blog, and entered few events (create Issue, update Issue Status) to it. Now it is time to extract event log from Planio. As other SaaS solution do, Planio also has [REST API](https://plan.io/api/) to extract data from outside. Currently Celonis EMS has ability to extract from arbitrary system that has REST API, Extractor Builder. Today I would like to setup Extractor Builder and try to extract Issue from Planio.

First step is to get Planio API key to access from Celonis EMS to Planio. After sign in to Planio, I will go to My Account page from Avatar menu at top right of screen. Next click show button at API Access Key menu in the right pane, then I can see API key (this is credential information so it is possible to reset key any time).

[![image](https://user-images.githubusercontent.com/67397583/149604286-0db72830-d36e-49eb-a2a5-78ad614e3f05.png)](https://user-images.githubusercontent.com/67397583/149604286-0db72830-d36e-49eb-a2a5-78ad614e3f05.png)

Even using API key is deprecated solution as [documentation](https://plan.io/api/#deprecated-authentication-methods) said, OAuth2.0 authentication is not feasible to Extractor Builder when I tried before. So I should go with API key for the moment.

Next step is Extractor setup in the Data Integration of Celonis EMS. I will go to Extractor Builder then click Build a New Extractor button. After determining Extractor name, I will setup Authentication Methods as below screen. As I got API key from Planio, I will select API Key Authentication. Header key is changed from x-api-key to X-Redmine-API-Key referring to documentation.

[![image](https://user-images.githubusercontent.com/67397583/149615062-e239c258-6d2a-4cc0-be88-53a2e5707181.png)](https://user-images.githubusercontent.com/67397583/149615062-e239c258-6d2a-4cc0-be88-53a2e5707181.png)

Then skip the Connection parameters and move to Define Endpoints. Endpoint is the address to access to resources in source systems, similar to table name in case of previous Postgres database. This is something complex part but at first I would like to create quite simple Endpoint, so click Add New Endpoint. 

Looking at [Planio resources](https://plan.io/api/#resources), I found that I can access to Issues that I entered before. I would like to get all entries of Issues, so I focus on `GET /issues.[json|xml]`. So endpoint name is determined as `issues`, then enter `{Connection.API_URL}/issues.json}` to Request URL. `{Connection.API_URL}` is automatically replaced with my URL later. `json` is the response format acceptable for Extractor Builder.

Let's verify this endpoint URL at configuring Response section. As below screen I will click Create Data Source button. Then enter connection name, my Planio URL and API key. Finally test connection to check previous steps, then save connection.

[![image](https://user-images.githubusercontent.com/67397583/149615914-dd7866b2-0b2f-4c2d-930a-e39dc4bf0877.png)](https://user-images.githubusercontent.com/67397583/149615914-dd7866b2-0b2f-4c2d-930a-e39dc4bf0877.png)

[![image](https://user-images.githubusercontent.com/67397583/149615988-e5fec6ff-b9b0-4b6c-ab0a-497e0e5e6d71.png)](https://user-images.githubusercontent.com/67397583/149615988-e5fec6ff-b9b0-4b6c-ab0a-497e0e5e6d71.png)

Above operation enable me to access to issues so click Build Response button then I can get sample structure of Issue response (if you can not see it, go back to Planio and enter new issue then try again). From this structure I can configure extraction tables. But this is first draft of issue endpoint, so finish setting as default.

[![image](https://user-images.githubusercontent.com/67397583/149616422-b003eaa6-a480-49d1-852c-f9e63b061866.png)](https://user-images.githubusercontent.com/67397583/149616422-b003eaa6-a480-49d1-852c-f9e63b061866.png)

Finally test end to end extraction by creating new data job like previous Postgres Database. By default I can get two tables, one is statistics of issues, and others is list of issues.

[![image](https://user-images.githubusercontent.com/67397583/149616647-d1b107a5-4a19-4437-91d9-d63e524d20f6.png)](https://user-images.githubusercontent.com/67397583/149616647-d1b107a5-4a19-4437-91d9-d63e524d20f6.png)

[![image](https://user-images.githubusercontent.com/67397583/149616688-a06d611c-8844-449c-b1ec-1e2761b804ce.png)](https://user-images.githubusercontent.com/67397583/149616688-a06d611c-8844-449c-b1ec-1e2761b804ce.png)

Next time I will configure endpoints to get more suitable output for following transformations.

Kaz
