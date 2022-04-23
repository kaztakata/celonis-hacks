---
title: "Group similar cases by Clustering"
date: 2021-10-02T10:56:43+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis','Pull Up Function']
---

Last week I posted [Convert Quantitative value to Categorical one by Quantile Function](../2021-09-25-convert-quantitative-value-to-categorical-one-by-quantile-function/) to create categorical dimension. Today I will create categorical dimension via different way, clustering.

Clustering is one of the unsupervised learning method, to automatically group cases by their attributes. Celonis PQL has clustering funciton `KMEANS`, so you are ready to use clustering.

Today I will use O2C process and would like to group customers by (1) their lead time from Invoice send to Clear Invoice and (2) net value. Imagine lower (Quicker) lead time and higher net value is high priority customer and vise versa.

Today's output is below, I clusetered each dot (customer) by k-means. So group 1 is higher priority than group 0.

[![image](https://user-images.githubusercontent.com/67397583/135702244-36a5125d-dfb2-4978-8dba-3965f6d33a61.png)](https://user-images.githubusercontent.com/67397583/135702244-36a5125d-dfb2-4978-8dba-3965f6d33a61.png)

By the way, I found that current customer code is too much to plot in graph, so I would like to group customer by its name before starting k-means. To do this I used `CLUSTER_STRINGS` function and convert customer name to similar one.

Below is check sheet of this function. I created `customerGroup` formula as `CLUSTER_STRINGS(KNA1.NAME1,EDIT_THRESHOLD({p1}))` then passed edit distance from 1 to 3 in each columns. You can see that result of edit distance 3 can create bigger group than 1, for example customer `K5392` belongs to one big group in case of 3, instead different group in case of 1 and 2.

[![image](https://user-images.githubusercontent.com/67397583/135702356-4bd7a601-0887-42ba-b851-7720ffd97eba.png)](https://user-images.githubusercontent.com/67397583/135702356-4bd7a601-0887-42ba-b851-7720ffd97eba.png)

Going back to k-means clustering, I created two dimensions mentioned above. I used edit distance 3 to group customer then defined `DOMAIN_TABLE`. I used Pull up funciton instead of aggregation function, so later I can filter customers by clicking group color dot in scatter plot.

```
-- avgDaysInvoiceClear
PU_AVG(
  DOMAIN_TABLE(KPI(customerGroup,3)),
  DATEDIFF(
    dd,
    PU_FIRST(VBAP, _CEL_O2C_ACTIVITIES.EVENTTIME, _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Send Invoice'),
    PU_FIRST(VBAP, _CEL_O2C_ACTIVITIES.EVENTTIME, _CEL_O2C_ACTIVITIES.ACTIVITY_EN = 'Clear Invoice')
  )
)

-- sumNetValue
PU_SUM(
  DOMAIN_TABLE(KPI(customerGroup,3)),
  VBAP.NETWR_CONVERTED
)
```

Then I determined clustering using two formulas (input and cluster are same). By passing number to parameter, I can change number of clusters by button dropdown.

```
-- clusterByCustomer
KMEANS (
  TRAIN_KM (
    INPUT (
      KPI(avgDaysInvoiceClear),
      KPI(sumNetValue)
    ), {p1}
  ),
  CLUSTER (
    KPI(avgDaysInvoiceClear),
    KPI(sumNetValue)
  ) 
)
```

In scatter plot you should use Grouping option and set k-means result, so each dot is colored based on cluster.

[![image](https://user-images.githubusercontent.com/67397583/135702994-4526a63a-b625-421c-b4a1-f0b3eae8e32c.png)](https://user-images.githubusercontent.com/67397583/135702994-4526a63a-b625-421c-b4a1-f0b3eae8e32c.png)

Finally, similar to last week topic, I chose Pull up function so filtering by other dimension (e.g. company code) does not affect this result.

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20211002.json) then push to your environment by content-cli.

---
