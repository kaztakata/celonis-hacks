---
title: "Connect to Celonis and Bring Back Instruction"
date: 2021-11-13T10:19:49+09:00
draft: false
tags: ['Data Integration','Extractor']
---

From last week I started Data Integration series and posted [Run Extractor on Your Local Machine](../2021-11-06-run-extractor-on-your-local-machine) to prepare for my Extractor and Postgres database. Today I will start using Extractor and show you the mechanism to extract data safely. 

From this week I will start Extractor and Postgres by next two steps (same as last week).
1. Open VS code and open terminal at celonis-postgres folder.
1. Enter `docker-compose up` in VS code terminal.

Next, I will login to Celonis training environment and go to Data Integration, then create new Data Pool. Then go to Data Connection and click New Data Connection button. Choose Database at next screen, then parameter setting screen is appeared as below screenshot.

Input parameters as below and click Test Connection. Normally it is successful.

- Uplink connection : choose one you created last week
- Database type : PostgreSQL (not PostgreSQL encrypted!!)
- Host : postgres
- Database Name : postgres,
- Username : celonis
- Password : Celonis123!

[![image](https://user-images.githubusercontent.com/67397583/141601188-6fd71779-f5ae-4be7-bae8-67c74cd56a3e.png)](https://user-images.githubusercontent.com/67397583/141601188-6fd71779-f5ae-4be7-bae8-67c74cd56a3e.png)

I will go back to VS code terminal that ran Extractor, then I can find Extractor log like below (shortened ID and remove unnecessary portion).

```
2021-11-13 01:46:58.215 Request with ID Dt4i.... received  
2021-11-13 01:46:58.304 Request with ID Dt4i.... is processed 
2021-11-13 01:46:59.162 Response for request Dt4i.... is sent, finished. Stopping keep alive requests 
```

First log said instruction to test connection (get zero byte data from Postgres), including host information from Extractor perspective and login information to Postgres, is passed from Celonis EMS to Extractor. 

Point is this instruction is not sent from EMS but Extractor periodically (per few seconds) access to EMS and if Extractor finds instruction then bring back it to Extractor. Extractor always use HTTPS request to EMS, and EMS do not use HTTPS to Extractor. If EMS needs to access to Extractor, this opening port is risk of invalid access to on-premise network. Normally enterprise customer do not want to open port considering this risk, but Extracor works well in that situation.

Second and third log said Extractor is executing extraction task instead of EMS, and return response (zero byte in this case) to EMS.

[![image](https://user-images.githubusercontent.com/67397583/141602382-ae8f0127-dd35-496c-8792-c1913f90301d.png)](https://user-images.githubusercontent.com/67397583/141602382-ae8f0127-dd35-496c-8792-c1913f90301d.png)

I will go back to EMS and click Save button then saved data connection is used for future data extraction. 

In the Data Connection screen you can click three dots button and find Extractor Logs menu. It returns log text file that is same as VS code terminal output. If you can not access to Extractor (normally yes), but you can find log from here (This time it does not work due to my incorrect configuration of Docker).

[![image](https://user-images.githubusercontent.com/67397583/141602837-971e0582-5421-44c1-87c9-b8c209508de4.png)](https://user-images.githubusercontent.com/67397583/141602837-971e0582-5421-44c1-87c9-b8c209508de4.png)

Next week I will connect to Postgres and extract data.

Kaz