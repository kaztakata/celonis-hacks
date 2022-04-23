---
title: "Maintain Saved Formulas effectively"
date: 2021-07-03T21:25:28+09:00
draft: false
tags: ['PQL','Process Analytics','Studio','Analysis','Pull Up Function']
---

In the last post [Handle NULL efficiently in Aggregation Function](../2021-06-26-handle-null-efficiently-in-aggregation-function/), I used saved formulas to split long and complex PQL to reusable components. Today I would like to share my best practice to use saved formulas.

For example I would like to ananlyze throughput time between arbitrary two activities. As below screenshot I created three dropdown buttons, switching time unit (sec, min, hour, day etc.) and two activities (from / to). Variables (time_unit, from, to) are set by user action and passed to PQL for calculating other components relevant to throughtput time (single KPI, histogram, boxplot, drill down table, and fact table).

[![image](https://user-images.githubusercontent.com/67397583/124353919-52cd3880-dc44-11eb-8516-effc7047db33.png)](https://user-images.githubusercontent.com/67397583/124353919-52cd3880-dc44-11eb-8516-effc7047db33.png)

In each component it is possible to write down throughput time PQL like below. But if I need to change something in PQL, I need to open each component and maintain it multiple times. So I will try to create saved formulas and simplify each component.

```
DATEDIFF(
    <%=time_unit%>,
    PU_FIRST(
        EKPO,
        _CEL_P2P_ACTIVITIES_EN.EVENTTIME,
        _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN = <%=from%>
    ),
    PU_FIRST(
        EKPO,
        _CEL_P2P_ACTIVITIES_EN.EVENTTIME,
        _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN = <%=to%>
    )
)
```

First I focused on the `PU_FIRST` portion. These functions can be summarized to formula `first_time` as below.

```
--first_time
PU_FIRST(
    EKPO,
    _CEL_P2P_ACTIVITIES_EN.EVENTTIME,
    _CEL_P2P_ACTIVITIES_EN.ACTIVITY_EN = {p1}
)
```
Then original PQL is changed to below.

```
DATEDIFF(
    <%=time_unit%>,
    KPI(first_time,VARIABLE(<%=from%>)),    -- call first_time formula passing parameter 'from'
    KPI(first_time,VARIABLE(<%=to%>))       -- call first_time formula passing parameter 'to'
)
```

This is something better than previous PQL, but let's encapsulate the `DATEDIFF` portion. I will create second formula `throughput_time` as below.

```
--throughput_time
DATEDIFF(
    {p1},
    KPI(first_time,VARIABLE({p2})),
    KPI(first_time,VARIABLE({p3}))
)
```

Finally original PQL is changed to `KPI(throughput_time, VARIABLE(<%=time_unit%>), VARIABLE(<%=from%>), VARIABLE(<%=to%>))`. It means each component is just call `throughput_time` with three parameters.

By the way, single KPI and drill down table are aggregating throughput time by `AVG` and `COUNT` function. In my opinion it is not required to create one more formula of each aggregation function. My focus is replacement of complex 'dimension' determination to formula (Exceptionally, I would like to create formula for adding multiple aggregation function case, like [Calculate Multi Dimensional KPIs](../2021-06-12-calculate-multi-dimentional-kpis/) post). In the end each component are determined like below.

| Title                | Component  | PQL                                                                                           | 
| -------------------- | ---------- | --------------------------------------------------------------------------------------------- | 
| Avg Throughput Time  | single KPI | `AVG(KPI(throughput_time, VARIABLE(<%=time_unit%>), VARIABLE(<%=from%>), VARIABLE(<%=to%>)))`   | 
| Case Count           | single KPI | `COUNT(KPI(throughput_time, VARIABLE(<%=time_unit%>), VARIABLE(<%=from%>), VARIABLE(<%=to%>)))` | 
| Distribution         | histogram  | `KPI(throughput_time, VARIABLE(<%=time_unit%>), VARIABLE(<%=from%>), VARIABLE(<%=to%>))`        | 
| Time Trend           | boxplot    | same as histogram                                                                             | 
| Drill Down by vendor | OLAP table | same as single KPI                                                                            | 
| Fact                 | OLAP table | same as histogram                                                                             | 

One more tips, I recommend to determine multi layered formualas as above because I can use interim result in verification purpose. In the right lower fact sheet, columns of from / to timestamp are displayed then `KPI(first_time,VARIABLE(<%=from%>))` and `KPI(first_time,VARIABLE(<%=to%>))` are assigned. So that it is easy to verify throughput time calculation of each case.

To summarize today's discussion,

- decomposite complex dimensional PQL to multi layered formulas
- assign variable to formula's parameters (not directly assign variable in the formula)
- not required to create formula for simple aggregation

Kaz

---

This post's program can be [downloaded here](../../examples/p2p_analysis_20210703.json) then push to your environment by content-cli.

---
