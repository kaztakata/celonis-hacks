---
title: "Create Additional Entry to Button Dropdown"
date: 2021-07-31T10:00:39+09:00
draft: false
tags: ['PQL','Process Analytics','Process Explorer']
---

I looked at the [Celopeers post](https://www.celopeers.com/s/question/0D50700000KZNLICA5/could-you-please-help-me-to-find-out-how-i-can-check-for-empty-variable-input-to-prevent-the-component-specifically-olat-table-from-running-into-an-error) that had issue when variable input is blank (`NULL`) then PQL using this variable had error. 

I already used work around below to skip FILTER execution if variable is null. But I also felt troublesome in two points. First is this is not officially documented so myself need to instruct to my colleagues. Second is more important, I would like to unfilter this selection if variable is not set, but there was no way to do it.

```
<% if(targetVariable != "") { %> FILTER "table"."column" IN (<%=targetVariable%>); <% } %>
``` 

Today I would like to share my alternative solution against second issue.

First step is setup Button Dropdown. When you choose `Load Entries` option you can determine PQL, normally it is just column. But I set formula `dropdownEntries` as below.

```sql
-- dropdownEntries
-- {p1} is table.column
CASE
  WHEN ISNULL({p1}) = 1 THEN '-'
  WHEN ISNULL(LEAD({p1}, PARTITION BY ({p1}))) = 1 THEN {p1}
  ELSE ' All of them'
END
```

- This formula have three WHEN / ELSE parts. First one is to handle `NULL` value in the column selected. As mentioned `NULL` can not be set as variable, so `NULL` is converted to hyphen (string).
- Second one is important but complex formula. `LEAD` function is looking at next row, and `PARTITOIN BY` is grouping key. So this looks at next row that has same column value. And `ISNULL(...) = 1` happens only when it is last entry of each value. Only last row of each value returns same value. Button Dropdown entries is displayed removing duplicated value, so doing this is no change to the output of entries.
- Third one, that is the rest of (duplicated) rows are conveted to fixed string ` All of them`. This fixed string is passed to variable and behave as it said at receiver PQL. It is imporatant that this addtional entry is created by unnecessary (duplicated) rows.

Next step to setup receiver side PQL. Main parts is standardized boolean formula `filterByDropdownEntries` that receives variable value and evaluate column value.

```sql
-- filterByDropdownEntries
-- {p1} is table.column
-- {p2} is variable
CASE
  WHEN {p2} = ' All of them' THEN 1
  WHEN {p2} = '-' THEN CASE WHEN ISNULL({p1}) = 1 THEN 1 ELSE 0 END
  ELSE CASE WHEN {p1} = {p2} THEN 1 ELSE 0 END
END
```

First `WHEN` returns 1 to all rows, second one returns 1 when column value is `NULL`, and third one is judging if variable value is same as column value then return 1.

Finally I demonstrate demo using this function. I created two sets of Button Dropdown to choose entries of two different columns (document type without `NULL` value, and rejection reason that has `NULL` value). Receiver components are single KPI to count cases that matches column selection, and process explorer that is filtered same as single KPI. You can see videos (click screenshot) to confirm each dropdown selection works, especially additional entry `All of them` works as not filtered.

[![thumb](https://user-images.githubusercontent.com/67397583/127732206-38f00f4b-c198-4dc9-95ef-bab8597d1d28.png)](https://user-images.githubusercontent.com/67397583/127732217-d83284e4-4c5b-402d-8996-30d2ef89ab4a.mp4)

PQL of receiver components are below.

```sql
-- single KPI to count cases that match Button Dropdown selection
COUNT(
  CASE 
    WHEN KPI(filterByDropdownEntries,VARIABLE(VBAK.AUART),VARIABLE(<%= documentType %>)) = 1 THEN VBAP.MANDT
    ELSE NULL
  END
)
```
```sql
-- display process explorer that matches Button Dropdown selection
CASE
  WHEN KPI("filterByDropdownEntries", VARIABLE(VBAK.AUART), VARIABLE(<%=documentType%>)) = 1 THEN _CEL_O2C_ACTIVITIES.ACTIVITY_EN
  ELSE NULL
END
```

This example is simple case, so for example you can customize multiple conditions to one component.

Kaz

---

This post's program can be [downloaded here](../../examples/o2c_analysis_20210731.json) then push to your environment by content-cli.

---

