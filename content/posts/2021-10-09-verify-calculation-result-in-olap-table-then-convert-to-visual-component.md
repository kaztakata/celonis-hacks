---
title: "Verify calculation result in OLAP table then convert to visual component"
date: 2021-10-09T09:33:43+09:00
draft: false
tags: ['Process Analytics','Studio','Analysis']
---

Today is 24th post of this blog series and I am suprised that I can continue to post blog every week. I got some reply from who loves Celonis in the world and it is my fun to continue posting. Celonis is releasing new functionarity year by year and I am interested in catching up them and thinking how these functions help my clients. After listening to next week's Celonis World Tour Webiner in Tokyo, I will get more inspiration to write blog.

Until this post I mainly shared how to calculate KPIs in PQL and not mentioned about visual components that is one of the characteristic of Celonis, because I believe correct calculation is basis of quality assurarance. After you can verify calculation result in OLAP table, it is easily convert to visual component then decorate it as you like. Today I would like to show how to do it, and tips of decoration.

Today I try to visualize the result of days between Ship and Invoice in [Handle NULL efficiently in Aggregation Function](../2021-06-26-handle-null-efficiently-in-aggregation-function/) post. As mentioned before, first step is to create OLAP table for counting Invoice within a day etc. For visualization I determined month of Ship Goods as dimension. Below is the result of OLAP table.

[![image](https://user-images.githubusercontent.com/67397583/136639035-05639d01-256a-447c-8a6d-48c7a0002d81.png)](https://user-images.githubusercontent.com/67397583/136639035-05639d01-256a-447c-8a6d-48c7a0002d81.png)

Second step is converting component type. After verifying OLAP table you will right click it and go to setting, then simply change `component type`. In this case I choose column chart.

[![image](https://user-images.githubusercontent.com/67397583/136639162-bb76469b-cfaa-445d-8e0c-23b3b5e4f946.png)](https://user-images.githubusercontent.com/67397583/136639162-bb76469b-cfaa-445d-8e0c-23b3b5e4f946.png)

Resulting appearance is not good enough because three columns appear per one month. So as third step I would like to change it to stacked bar chart.

Again go to settings and choose `Data series: within a day` at the top dropdown. Then scroll down to the buttom and tick `Stacked` button. Same operation is done against `Data series: over a day` then two columns are stacked successfully.

How about `total` column ? I would like to use it to display total number but would not like to stack. Go to `Data series: total` then select Line chart in `Alternative type` dropdown, then height of node of line is same as that of stacked bar. Next change to transparent in `Series color` button and tick `Display data labels`. Finally line is disappeared and only total number is displayed like legend of stacked bar. Below is process of converting OLAP table to final appearance.

[![image](https://user-images.githubusercontent.com/67397583/136639637-90fff977-499c-48bf-9be4-3809666b75fb.png)](https://user-images.githubusercontent.com/67397583/136639637-90fff977-499c-48bf-9be4-3809666b75fb.png)

Even if this process is not applied (e.g. histogram), you should first create OLAP table then manually copy PQL to visual component for keeping calculation quality.

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20211009.json) then push to your environment by content-cli.

---
