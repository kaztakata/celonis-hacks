---
title: "Pay attention to Extract SAP Tables"
date: 2021-12-25T08:53:48+09:00
draft: false
tags: ['Data Integration','Extraction',]
---

Until last post I explained general topics of extraction task, adapted to all kind of source systems. Today I would like to focus on SAP ECC or S4HANA as source system and would like to tell you the SAP specific issues.

First issue is regarding source system itself. We would like to guarantee source system's availability even if I connect Celonis EMS to that. So we may choose testing environment that is snapshot of production system, as source system that connect to Celonis EMS. 

Especially SAP system, there are various options to copy environment, and major one is called Client copy. Client copy enable us to copy table data that is separated by Client number (technically first column `MANDT` in almost all tables). For example, I can copy production data assigned to Client number 100, to another Client 200 in testing environment. 

Issue regarding Client copy for me is this program does not copy change history tables (technically `CDHDR` header and `CDPOS` item tables). Chagne history is useful to trace rework activities in process mining, so normally Client copy environment is not good for us. Instead we can use System Copy, that is full backup of the system, including all Clients and change history ([ref](https://answers.sap.com/questions/10323065/data-difference-between-system-copy-and-remote-cli.html)).

In my experience Client copy is used for most of enterprise, so SAP admin normally proposed to use Client copy environment. To check copy environment, I requested them to see the minimum date of table `CDHDR-UDATE`. If that is greater than the period I analyse process data, this environment is not useful.

Second issue is relevant to data volume of change history tables. In my experience change history rows in enterprise may be 1 billion around. SAP change history tables store all aspects of change in SAP system, so most of records are not relevant to my analysis. 

Looking at header table `CDHDR`, first column is Client `MANDT` as I explained, and second column is object class `OBJECTCLAS` that groups SAP tables by business functions, for example sales order tables are assigned to object class `VERKBELEG` (list of object classes and tables are stored at SAP table `TCDOB`). I can filter extraction record by writing Filter Statement at extraction table configuration.

Regarding item table `CDPOS`, this table does not have timestamp column. I recommend to use join configuration and add `CDHDR` table that has timestamp column `UDATE` and `UTIME`. `CDPOS` and `CDHDR` table rows are not updated and deleted, so I can use Delta Load option for these tables (see [Verify Cloning Table Contents via Delta Load](../2021-12-04-verify-cloning-table-contents-via-delta-load)).

Using both Filter Statement and Join Configuration, it is possible to minimize extraction volume of change history tables.

Kaz
