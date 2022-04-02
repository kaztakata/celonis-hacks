---
title: "Consider Case ID before Starting Transformation"
date: 2022-02-12T13:03:00+09:00
draft: false
tags: ['Data Integration','Transformation']
---

After long explanation of Extractor Builder, I can move to Transformation Topic from now, using Planio issue and change history. But before starting detail discussion, I would like to discuss general issues at first.

In process mining context, Transformation is procedure to generate event log table from source tables. [Event Log or Event Data](http://www.processmining.org/event-data.html#data) is the collection of case and its event (activity) with timestamp. Case ID can be associated with multiple activities in source system, in other word it is not possible to generate event log without case ID.

When I tried to design Event Log before, I referred Business Process Flow (Diagram) but this flow was normally written in perspective of how business staff (or role) and resources (system, form etc.) are ordered, without taking care of case ID, so I was stucked how to assign case ID to some of activities. For the moment my answer is all activities can not perfectlly be mapped to one (integrated) process.

For example, I was requested to design event log of supply chain management of a company, that include sales, production, and purchasing activities. What is the case ID of this supply chain ? Of cource I know that for example production has its ID like order number, but in transition from productoin to sales, its ID are not transferred if production purpose is to make stock. 

How about if I behave stock as case ID ? Stock may have its batch number and it seems to be case ID. But if I go back to upstream (purchasing side) there may be powder bag that is not easy to identify by each.

In another example, some of monthly closing tasks in finance area are operated once per month, and I do not know how to create case ID of these tasks. In this case year and month itself may be case ID, but how to integrate with other tasks that are operated many times per month ?

Going back to Business Proces Flow, I should think about how many operation is done in each box (activity) and is it possible to transfer case ID of previous activity to next one. After that I should split business process to small processes that can be trasnformed to process mining event log.

Kaz
