---
title: "Use BIND function to enable multiple DOMAIN_TABLE"
date: 2021-09-11T13:16:26+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis','Pull Up Function','Variant Explorer']
---

Variant Explorer is the major Celonis Analytical view to find process pattern by GUI operation. Of course I used it many times and enjoyed it at first, and found that I had a lot of effort to filter on and off to observe process. Today I would like to create collective list of variant to see major process KPI. Also I would like to show you `BIND` function that is difficult to understand but quite convenient if you know it.

Today's output report image is below. My goal is to calculate case count of variant by vendor (`EKKO.LIFNR`) in P2P process. Also I would like to cacululate case coverage and ranking (index order) of variant by vendor as Variant Explorer. Finally I used it to find vendor list that does not use Purchase Requisition (starts from Purchase Order) in its major pattern (> 20%).

[![image](https://user-images.githubusercontent.com/67397583/132936310-2c1e9f17-d2ae-48d8-9044-57722c232e22.png)](https://user-images.githubusercontent.com/67397583/132936310-2c1e9f17-d2ae-48d8-9044-57722c232e22.png)

First step is to determine two dimensions, vendor and variant. Later I need to create `DOMAIN_TABLE` from these two columns, so both columns should be derived from same table (this time it is `EKPO`). I need to introduce `BIND` for vendor column in header table `EKKO` to bind to item table `EKPO` as `BIND(EKPO,EKKO.LIFNR)` , then it looks like vendor columns is copied to `EKPO`. I determined two columns as formulas `vendorDomain` and `variantDomain` as below.

```
-- _vendor
BIND(EKPO,EKKO.LIFNR)

-- _variant
VARIANT(_CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN)

-- vendorDomain
PU_FIRST(
  DOMAIN_TABLE(
    KPI(_vendor),
    KPI(_variant)
  ),
  KPI(_vendor)
)

-- variantDomain
PU_FIRST(
  DOMAIN_TABLE(
    KPI(_vendor),
    KPI(_variant)
  ),
  KPI(_variant)
)
```

From this I use above `DOMAIN_TABLE` with vendor and variant as baseline domain. Next step is to calculate case count for each domain, and it is simply count case key (formula `countCaseDomain`). Then I would like to count case by vendor, but it is not simple as previous column.

To count case by vendor I need to use `DOMAIN_TABLE` of vendor. This is joined to `EKPO` table, not baseline domain. As discussed [last week](../2021-09-04-convert-count-unit-of-kpi-by-count-distinct/), normally two `DOMAIN_TABLE`s are N-M relationship so it does not work. Today's breakthrough is to `BIND` case count by vendor to `EKPO` table, then used it from baseline domain (formula `countCaseVendor` as below).

```
-- countCaseDomain
PU_COUNT(
  DOMAIN_TABLE(
    KPI(_vendor),
    KPI(_variant)
  ),
  EKPO._CASE_KEY
)

-- countCaseVendor 
PU_FIRST( -- 3. pick up EKPO first record by each baseline domain
  DOMAIN_TABLE(
    KPI(_vendor),
    KPI(_variant)
  ),
  BIND( -- 2. bind case count by vendor to EKPO table
    EKPO,
    PU_COUNT( -- 1. case count by vendor
      DOMAIN_TABLE(
        KPI(_vendor)
      ),
      EKPO._CASE_KEY
    )
  )
)
```

Finally I created formula `caseCoverageVendor` as `KPI(countCaseDomain) / KPI(countCaseVendor)`, then using this I created `indexCoverageVendor` as below.

```
INDEX_ORDER(
  KPI(caseCoverageVendor),
  ORDER BY (KPI(caseCoverageVendor) DESC),
  PARTITION BY (KPI(vendorDomain))
)
```

Everything works well, isn't it ? Actually this is not perfect. When I selected some other column filter (e.g. purchasing group), because all of above formula is using PU function that is not recaluculated by UI filter, so after filtering case count etc. are kept as it is. For the moment I do not have good answer to solve it, but adding condition term in PU function in some way may be work around.

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210911.json) then push to your environment by content-cli.

---
