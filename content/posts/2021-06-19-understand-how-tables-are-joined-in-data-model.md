---
title: "Understand how Tables are joined in Data Model"
date: 2021-06-19T11:42:09+09:00
draft: false
tags: ['PQL','Process Analytics','Data Model']
---

In the [Understand Difference between Dimension and KPI](../2021-05-01-understand-difference-between-dimension-and-kpi/) post, I mentioned that Columns used as dimensions and KPIs are inplicitly joined based on data model, but I did not mention how to join tables in data model. Today I would like to say about it.

Celonis Data Model support snowflake schema that consists of multiple tables, each table pair join as 1:N relationship. Normally activity table is top of N side and dimension tables including case table are bottom of 1 side as pyramid hierarchy.

From here, I used part of Account Payable data model. First I look at the relationship between Vendor Master table `LFA1` (1 side) and Accounting document item table `BSEG` (N side).

When I used both `LFA1` and `BSEG` tables in PQL, `BSEG` (N side) tables is left joined with `LFA1` (1 side) by join key (Vendor code of two tables). Same as standard SQL function, left join keeps all record of `BSEG` and relevant `LFA1` record. This means Vendor that is not used in Accounting document are not displayed in the output of PQL. It is not possible to switch left side of table by PQL.

Below screenshot shows comparison between PQL of single table `LFA1` (blue part) and that of join tables `LFA1`-`BSEG` (red part). I used filter by Vendor name to easily distinguish the difference. Even blue part shows `755` vendor count , red part only shows `6` vendor count because N side `BSEG` record only have `6` vendors. When I set up count of `BSEG` table, those value are always non-zero value. Because of above join functionality, e.g. it is not possible to display vendor `0000308885` has `0` count of `BSEG` table.

[![image](https://user-images.githubusercontent.com/67397583/122629713-f7eef980-d0f9-11eb-8ab3-f7184295c8ff.png)](https://user-images.githubusercontent.com/67397583/122629713-f7eef980-d0f9-11eb-8ab3-f7184295c8ff.png)

As second example, I looked at relationship between Purchase Order Item table `EKPO`, Vendor Invoice Item table `RSEG`, and Accounting document item table `BSEG` (`EKPO` is 1 side and `BSEG` is N side). Accounting document item may have Vendor Invoice number, and Vendor Invoice item may have Purchase Order number. Using these three tables in PQL, because `BSEG` is the N side, all record of `BSEG` are kept, but some of `BSEG` record does not have `RSEG` record. Also some of `RSEG` record does not have `EKPO` record.

Below screenshot shows OLAP table to see `BSEG` count that can connect with `RSEG` and `EKPO` record respectively. As mentioned, all `BSEG` record can be seen in OLAP table (`600,941 = 312,293 + 127,869 + 160,779`).

[![image](https://user-images.githubusercontent.com/67397583/122631506-1efff800-d107-11eb-8652-cfdcdee1ef7a.png)](https://user-images.githubusercontent.com/67397583/122631506-1efff800-d107-11eb-8652-cfdcdee1ef7a.png)

By the way, first and second record of `EKPO.MANDT` value `-` is not `NULL` value like standard SQL left join result (it is ambiguous that `NULL` value is also displayed as `-`). As proof, this value cannot be used for filtering condition as below screenshot (total count is changed to `0` after filtering). This `-` means “Not evaluated” because no `EKPO` record exists in this case. Take care that PQL function using this record returns “Not evaluated” too (on the other hand `NULL` value can be handled by `ISNULL` function or `IS NULL` predicate).

[![2021-06-19-_-Process-Analytics-Google-Chrome-2021-06-21-20-49-08](https://user-images.githubusercontent.com/67397583/122757589-bebdb180-d2d2-11eb-93c4-2f2ae58dd49b.gif)](https://user-images.githubusercontent.com/67397583/122757589-bebdb180-d2d2-11eb-93c4-2f2ae58dd49b.gif)

To summarize, join behaviors in data model are

- Always N side is the left side of left join
  - If N side table record missing, 1 side table record are disappeared.
  - If 1 side table record missing, N side table record remains and 1 side table record is “Not evaluated” (different from `NULL` value).

Kaz

---

This post's program can be [downloaded here](../../examples/ap_analysis_20210619.json) then push to your environment by content-cli.

---

