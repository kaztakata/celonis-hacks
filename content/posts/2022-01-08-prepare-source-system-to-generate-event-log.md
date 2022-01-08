---
title: "Prepare Source System to Generate Event Log"
date: 2022-01-08T16:35:30+09:00
draft: false
tags: ['Event Collection','Transformation']
---

Until last post I explained extraction topics using own Postgres database. I think it is better to test extraction functions with changing database by yourself. But it is hard to manually input data record that is meaningful as event log.

From now on I will move to transformation topic. To explain this, I think it is required to prepare source system that has user interface, database and API to connect to Celonis EMS, to easily generate event log and extract it. 

I would like to introduce [Planio](https://plan.io/) as source system. This is cloud hosting system using [Redmine](https://www.redmine.org/), that is open source project management web application. I already used Planio for some months and I feel it is sinple and powerful solution to manage my team.

At the top page of [Planio](https://plan.io/) you can create your environment by clicking `Start your free trial` button, that has no charge if you change to [Bronze (free) plan](https://plan.io/pricing/) after initiation of environment. If you successfully create your enviornment, you can see below screen as me (I can see Japanese text but you may see different language).

[![image](https://user-images.githubusercontent.com/67397583/148636749-46d1adc5-1ea2-4b81-b49e-852f031760bb.png)](https://user-images.githubusercontent.com/67397583/148636749-46d1adc5-1ea2-4b81-b49e-852f031760bb.png)

To generate your event log with free license restriction (only one project), first I should delete existing project `website-redesign`. I click tile at the home page to move to project top page, then click trash button. 

After that I can create new project. From top left `+` button, I can create new project. I named `Prototype production of XYZ` project. 

Next I would like to create issue. From `+` button, I can create new issue too. This time simply enter Subject `Prepare Kick off` then create issue. Then new issue number (in my case 8) is registered.

[![image](https://user-images.githubusercontent.com/67397583/148637709-e01ec65f-92f5-4650-8901-9eabefa34f00.png)](https://user-images.githubusercontent.com/67397583/148637709-e01ec65f-92f5-4650-8901-9eabefa34f00.png)

Finally to create change log, click `Edit` button and change status then submit change. Change history is displayed as note #1.

[![image](https://user-images.githubusercontent.com/67397583/148637844-46a670d0-c074-4047-9da4-d6cdf6c8963b.png)](https://user-images.githubusercontent.com/67397583/148637845-ac6e9d63-ea89-4716-859a-a9297a1db342.png)

Next time I would like to setup Extractor Builder in Celonis EMS to extract event log from Planio.

Kaz
