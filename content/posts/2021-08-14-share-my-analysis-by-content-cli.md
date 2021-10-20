---
title: "Share my Analysis by Content-CLI"
date: 2021-08-14T13:33:06+09:00
draft: false
tags: ['Content-CLI','Machine Learning','Process Analytics']
---

Recently I was thinking how to share my work with you not only blog post but experience by live environment. Today I would like to tell how to reproduce my Analysis to your environment by [Content-CLI](https://github.com/celonis/content-cli). Content-CLI is used to copy programs between Celonis environments via backup file. I created backup file and upload to public repository. So you can download it and reproduce my program in your environment.

First you need to open your training environment if you do not. You will go to [Celonis LMS](https://lms.celonis.com/) then sign up for your account. After that you enroll Process Discovery Basics cource then you will receive invitation mail of training envioronment. Training environment URL is like https://your-email-address.training.celonis.cloud/ and your-email-address is based on your sign up email address to Celonis LMS.

From now on, I would like to tell procedure to reproduce Analysis of last blog post [Create Matrix of Throughput Time by Pivot Table](../2021-08-07-create-matrix-of-throughput-time-by-pivot-table/). In case of other Analysis, please change parameters accordingly.

After you can login to your training environment, go to Process Analytics first. Click New button at left side (Workspaces), then select Data Model `CelonisCertificationExam_Order-to-Cash_DataModel` and name workspace as `celonis-hacks O2C` (please see below table for other Processes / Data Models). You should note workspace ID from URL of new workspace, like `workspaces=2056362c-0576-4356-82d2-2ce341db6fbf`, that is required to push Analysis later.

| Process | Data Model                                          | 
| ------- | --------------------------------------------------- | 
| O2C     | CelonisCertificationExam_Order-to-Cash_DataModel    | 
| P2P     | Purchase-To-Pay_Training_EN                         | 
| AP      | CelonisCertificationExam_AnalysisBuilding_2020/2021 | 

Next you will create API-Key from your profile. Click your avatar on top right of screen, then click Edit Profile button. Then enter content-cli to New API Key name then click Create API Key button (below screen). You should note API Key that makes possible to login to this environment without UI.

[![image](https://user-images.githubusercontent.com/67397583/129434323-7ce72520-c163-46cc-932b-959926c01b46.png)](https://user-images.githubusercontent.com/67397583/129434323-7ce72520-c163-46cc-932b-959926c01b46.png)

Next is the last step of preparation. You will go to Machine Learning then click New App. Like below screen, you can create Jupyter Workbench in Machine Learning. Click App you created, then you will go to Jupyter Workbench.

[![image](https://user-images.githubusercontent.com/67397583/129434369-353cfa82-301b-499e-ae4b-c1d3e7de3401.png)](https://user-images.githubusercontent.com/67397583/129434369-353cfa82-301b-499e-ae4b-c1d3e7de3401.png)

Jupyter Workbench is web based interactive development environment for Jupyter notebook, but today I do not touch Jupyter notebook but Content-CLI installed to this environment.

First you need to copy my backup file. Easiest way to copy and sync file is using Git. In the Workbench select menu Git > Clone a repository, then enter https://github.com/kaztakata/celonis-hacks. You can see celonis-hacks folder at left side. If you would like to update it, then go to celonis-hacks folder and select Git > Pull from Remote.

Next you will start Content-CLI. In the Workbench select File > New > Terminal, then you can see terminal as below screen. Terminal is waiting for your command so you will input command after $ sign and input Enter key. If additional information is required, enter command and input Enter key too.

[![image](https://user-images.githubusercontent.com/67397583/129436118-acdbbe70-9b6c-485a-a4ea-5616f6cbe767.png)](https://user-images.githubusercontent.com/67397583/129436118-acdbbe70-9b6c-485a-a4ea-5616f6cbe767.png)

First Content-CLI command is to create profile for login to training environment via Content-CLI. Please change your team and api token value accordingly.

```
(base) jovyan@workbench-6599f565c6-sr2f8:~$ content-cli profile create
Name of the profile: mytraining
Your team (please provide the full url): https://your-email-address.training.celonis.cloud/
Your api token: your-api-token
info:    Profile created successfully!
```

Next command is to push my backup file to workspace you created. Please change Workspace ID to your O2C Process Workspace.

```
(base) jovyan@workbench-6599f565c6-sr2f8:~$ content-cli push analysis -p mytraining --workspaceId 2056362c-0576-4356-82d2-2ce341db6fbf -f celonis-hacks/docs/examples/o2c_analysis_20210807.json 
info:    Analysis was pushed successfully. New ID: 2c1c370b-2a4b-4321-b3e7-9f3c27dbaf19
```

Finally you can go back to Process Analytics then confirm new Analysis is created. Please enjoy my blog post with live environment.

Kaz
