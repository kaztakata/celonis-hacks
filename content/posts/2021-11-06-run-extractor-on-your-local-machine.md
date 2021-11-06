---
title: "Run Extractor on Your Local Machine"
date: 2021-11-06T09:59:30+09:00
draft: false
tags: ['Event Collection','Extractor']
---

From this week I would like to explain my experience regarding Event Collection functions (Extraction, Transformation, Load etc.). To do this, I try to create sample source systems and build code in Celonis training environment.

As first topic, I would like to explain on premise Extractor, that is in the middle between your source systems (SAP, Oracle etc.) and Celonis EMS and support transferring data. By the way, because I do not want to pay licence of source systems for this blog, I would like to use open source Postgres database. Also I do not want to pay subscription of cloud server, I use my local machine instead.

I use Docker enviornment that can easily start and stop my sample source system and Extractor, and does not affect to other portion of my local machine. If you are using Windows 11, installing Docker environment is quite simple as below.

1. Install Windows Subsystem for Linux Preview and Ubuntu via Microsoft Store referring to [this](https://devblogs.microsoft.com/commandline/a-preview-of-wsl-in-the-microsoft-store-is-now-available/)
1. Install Docker Desktop via [website](https://www.docker.com/products/docker-desktop)
1. (To operate Docker and edit configuration files in WSL) Install Visual Studio Code via [website](https://azure.microsoft.com/en-us/products/visual-studio-code/)

Open Visual Studio Code (VS code) and press F1, select `Remote-WSL: New WSL Window`. At the new screen select Open Folder button and input your home directory as below.

![From Microsoft website](https://microsoft.github.io/vscode-remote-release/images/remote-wsl-new-window.png)

Next open terminal by pressing Ctrl+Shift+@, then input `git clone https://github.com/kaztakata/celonis-postgres`. Then `cd celonis-postgres` to move to newly created folder.

[![image](https://user-images.githubusercontent.com/67397583/140595660-5647157c-bb02-4332-ab87-e0d67d8d4857.png)](https://user-images.githubusercontent.com/67397583/140595660-5647157c-bb02-4332-ab87-e0d67d8d4857.png)

From here you need to configure both Extractor and Celonis EMS.

1. (In Celonis EMS) go to help page. Search 'Downloads and Version History' then download 'Dockerized JDBC Extractor Package' file (currently it is 2020-11-05_JDBC_Extractor_Package_Docker.tar.gz). Drug and drop this file to VS code folder celonis-postgres.
1. (In VS code terminal) enter `docker load -i 2020-11-05_JDBC_Extractor_Package_Docker.tar.gz`.
1. (In Celonis EMS) go to Team Setting from right top avatar. Go to Uplink integrations. Push Connect new system button, then type Connector, note client ID and Secret for next step then save it. 
1. (In VS code terminal) open `uplink.env` file then replace URL(`your-email-address`), client ID (`yourClientId`) and Secret (`yourClientSecret`).

Finally enter `docker-compose up` in VS code terminal. You can see flowing logs in the initiation step and it is stopped soon . If log does not stop and see `Uplink could not connect` in the log, you can stop running docker by entering Ctrl+C in VS code terminal, then check configuration and try `docker-compose up` again.

Again go to Celonis EMS, Team Setting, Uplink integrations. Check if your uplink status is green, then your Extractor in local machine can successfully access to IBC. After checking connection, enter Ctrl+C in VS code terminal, then Extractor and Postgres are stopped. You can close VS code afterward.

[![image](https://user-images.githubusercontent.com/67397583/140595109-d9f644ff-d1e2-4644-bd24-89a627ee7171.png)](https://user-images.githubusercontent.com/67397583/140595109-d9f644ff-d1e2-4644-bd24-89a627ee7171.png)

Next week I would like to extact data form Postgres and explain detail of Extractor mechanism.

Kaz
