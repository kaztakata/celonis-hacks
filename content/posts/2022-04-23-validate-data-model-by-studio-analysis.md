---
title: "Validate Data Model by Studio Analysis"
date: 2022-04-23T08:43:22+09:00
draft: false
tags: ['Data Integration','Transformation','Data Model','Studio','Analysis','Variant Explorer','Case Explorer']
---

At last post [Construct My First Data Model](../2022-04-16-construct-my-first-data-model), I created Data Model and load data to it. Normally initial load is not perfect, so I should check data in Data Model. Today I would like to share how to validate my Data Model using Analysis. By the way, Celonis EMS main function is now Studio (and App for viewer) and Analysis is also part of Studio, so today I will create Studio instead of Process Analytics to create Analysis.

First step is to create package for collecting Analysis and other nodes (View, Knowledge Model, Action Flow etc.). Just click Cerate Package button and enter name (and key automatically). Then click three dots button at right side of new package (im my case `planio`), and go to settings. In the setting I must first setup variable of Data Model for this package. So click Variable tab and Create Variable button in that tab.

[![image](https://user-images.githubusercontent.com/67397583/164874444-6652abce-1ed8-4327-bc75-9ccedc89504d.png)](https://user-images.githubusercontent.com/67397583/164874444-6652abce-1ed8-4327-bc75-9ccedc89504d.png)

I should choose Data Model I previously loaded, and check status of data load (green is OK). Then determine key of data model variable (normally default value is fine), and save it. 

Next step is to create blank Analysis. I will click plus button at right of my package, and enter name and key. The point is node key under package is unique, even it is deleted. So you are careful to determine temporary key. I used `1` that is really temporary key for validation. And I should setup Data Model Variable I created in the first step. Another option Knowledge Model is too much for today's validation, so I simply use Data Model.

[![image](https://user-images.githubusercontent.com/67397583/164874576-aaa02936-b131-4541-82df-7b4791ec7716.png)](https://user-images.githubusercontent.com/67397583/164874577-d61b82d5-97ec-49ab-8556-baecb5f472ce.png)

It is ready to create components in Analysis, but before that I will check number of case at upper left of Analysis sheet. It must be same as table count of case table (in my case `issues`) so I will go back to Data Integration and use SQL `SELECT COUNT(*) FROM issues` to compare number. If different, there are two possibilities, one is data model reload is not sufficient (if you did reload many times and final reload failed e.g.), and the other is incorrect configuration of data model especially case table assignment.

Then I will create Variant Explorer in blank sheet. Until now I creaated `Raise Issue`, `Close Issue`, `Take Note`, `Update Progress` and `Due date has passed`, so I will check these activities are appeared in either of process variant. Also I will check order of activities is appropriate.

[![image](https://user-images.githubusercontent.com/67397583/164875154-e4887f4e-ad88-45cb-8c45-d66ecaf9a092.png)](https://user-images.githubusercontent.com/67397583/164875154-e4887f4e-ad88-45cb-8c45-d66ecaf9a092.png)

To check more detail, I will create new sheet for Case Explorer. I can click case I would like to check, and see each activity and their columns. For example I will check `_SORTING`, and `changed_by` are appropriate for each activity.

[![image](https://user-images.githubusercontent.com/67397583/164876250-93b6f880-7cf7-4e5c-b51e-b0a6dff122a7.png)](https://user-images.githubusercontent.com/67397583/164876250-93b6f880-7cf7-4e5c-b51e-b0a6dff122a7.png)

Finally I will check table columns in UI selection screen. Click selection button next to `100%` number, and click attribute selection. You can see table list I loaded, and columns and value of column. For example I will check `priority$name` value under `issues` table that will be used for dimension column.

[![image](https://user-images.githubusercontent.com/67397583/164877666-e705a507-c332-4363-9f75-128bff780e47.png)](https://user-images.githubusercontent.com/67397583/164877666-e705a507-c332-4363-9f75-128bff780e47.png)

Doing these steps I checked Data Model load is good enough, and if not I will go back to transformation (or extraction) and try data model reload again. This validation step is quite important for following Analysis construction, otherwise I will find trouble later and I need huge rework.

Kaz
