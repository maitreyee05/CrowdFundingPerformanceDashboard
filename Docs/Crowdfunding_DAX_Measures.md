# DAX Measures: Crowdfunding Campaign Performance Dashboard

All 25 measures live in the `00_Measures` table of `CrowdFundingCampaignPerformanceDashboard.pbix`. The definitions below were read directly from the model.

**Tables used:** `F_Campaigns`, `F_Pledges`, `D_Date`

> **Real vs. simulated data.** Measures on `F_Campaigns` use the real Kickstarter data. Measures on `F_Pledges` use the **simulated** pledge table (5,000 campaigns), so they are marked *(simulated)*.

---

## 1. Core campaign measures (real data)

### TotalCampaigns
Number of campaigns.
```dax
TotalCampaigns = COUNTROWS(F_Campaigns)
```

### SuccessfulCampaigns
Campaigns that reached their goal.
```dax
SuccessfulCampaigns =
CALCULATE([TotalCampaigns], F_Campaigns[is_successful] = 1)
```

### SuccessRate
Share of campaigns that succeeded. Failed and canceled campaigns both count as not successful.
```dax
SuccessRate = DIVIDE([SuccessfulCampaigns], [TotalCampaigns])
```

### SuccessRate(min 100)
Same as `SuccessRate`, but blank when a group has fewer than 100 campaigns. Use it in visuals that slice into small groups (subcategories, country × category) so a tiny group can't look like a trend.
```dax
SuccessRate(min 100) =
IF([TotalCampaigns] >= 100, [SuccessRate])
```

### GoalAmount
Total funding goals, in USD.
```dax
GoalAmount = SUM(F_Campaigns[goal_usd])
```

### PledgeAmount(Campaign)
Total amount pledged, in USD, from the campaign totals.
```dax
PledgeAmount(Campaign) = SUM(F_Campaigns[pledged_usd])
```

### Funding%
Pledged as a share of the goal.
```dax
Funding% = DIVIDE([PledgeAmount(Campaign)], [GoalAmount])
```

---

## 2. Pledge-detail measures (simulated)

These measures use the generated `F_Pledges` table.

### Pledge(Detail)
Total pledge amount in the pledge table.
```dax
Pledge(Detail) = SUM(F_Pledges[pledge_amount_usd])
```

### TotalPledge(Detail)
Number of pledges (one row per pledge).
```dax
TotalPledge(Detail) = COUNTROWS(F_Pledges)
```

### Avg Pledge
Average size of one pledge.
```dax
Avg Pledge = DIVIDE([Pledge(Detail)], [TotalPledge(Detail)])
```

### TotalPledge
Number of campaigns that have pledge-level detail (`has_pledge_detail = 1`). Despite the name, it counts campaigns and not pledges.
```dax
TotalPledge =
CALCULATE(COUNTROWS(F_Campaigns), F_Campaigns[has_pledge_detail] = 1)
```

> `Pledge(Detail)` covers only the 5,000 sampled campaigns, so it will not match `PledgeAmount(Campaign)`. For overall totals, use `PledgeAmount(Campaign)`.

---

## 3. Last-year measures

These feed the year-over-year measures in section 4. They use `SAMEPERIODLASTYEAR` on `D_Date[date]`.

```dax
TotalCampaignsLY =
CALCULATE([TotalCampaigns], SAMEPERIODLASTYEAR(D_Date[date]))

GoalAmountLY =
CALCULATE([GoalAmount], SAMEPERIODLASTYEAR(D_Date[date]))

PledgeAmountLY(Campaign) =
CALCULATE(SUM(F_Campaigns[pledged_usd]), SAMEPERIODLASTYEAR(D_Date[date]))

SuccessRateLY =
CALCULATE([SuccessRate], SAMEPERIODLASTYEAR(D_Date[date]))
```

### Last-year measures on pledge detail (simulated)
```dax
PledgeLY(Detail) =
CALCULATE(SUM(F_Pledges[pledge_amount_usd]), SAMEPERIODLASTYEAR(D_Date[date]))

TotalPledgeLY(Detail) =
CALCULATE(COUNTROWS(F_Pledges), SAMEPERIODLASTYEAR(D_Date[date]))

Avg Pledge LY =
CALCULATE([Avg Pledge], SAMEPERIODLASTYEAR(D_Date[date]))

TotalPledgeLY =
CALCULATE([TotalPledge], SAMEPERIODLASTYEAR(D_Date[date]))
```

### FundingLY%
Funding % for the same period last year. **See the fix in section 6.**
```dax
FundingLY% = DIVIDE([PledgeAmount(Campaign)], [GoalAmount])
```

---

## 4. Year-over-year measures

Each one returns `(this year - last year) / last year`, and `DIVIDE` handles a blank or zero denominator. They drive the **YOY%** values on the dashboard's KPI cards.

```dax
TotalCampaigns YOY% =
DIVIDE([TotalCampaigns] - [TotalCampaignsLY], [TotalCampaignsLY])

Goal YOY% =
DIVIDE([GoalAmount] - [GoalAmountLY], [GoalAmountLY])

PledgeAmount YOY% =
DIVIDE([PledgeAmount(Campaign)] - [PledgeAmountLY(Campaign)], [PledgeAmountLY(Campaign)])

SuccessRate YOY% =
DIVIDE([SuccessRate] - [SuccessRateLY], [SuccessRateLY])

TotalPledge YOY% =
DIVIDE([TotalPledge] - [TotalPledgeLY], [TotalPledgeLY])
```

> **Reading `SuccessRate YOY%`:** it is a *relative* change. If the success rate goes from 36.0% to 36.2%, the measure shows +0.6%, not +0.2 percentage points.

---

## 5. Calculated columns (`F_Campaigns`)

These group goals and campaign lengths into bands for the heatmap matrix. Each band has a numeric sort column. Set **Column tools > Sort by column** so the labels appear in order.

```dax
Goal Band =
SWITCH(TRUE(),
    F_Campaigns[goal_usd] < 1000,   "Under $1K",
    F_Campaigns[goal_usd] < 5000,   "$1K-5K",
    F_Campaigns[goal_usd] < 10000,  "$5K-10K",
    F_Campaigns[goal_usd] < 25000,  "$10K-25K",
    F_Campaigns[goal_usd] < 50000,  "$25K-50K",
    F_Campaigns[goal_usd] < 100000, "$50K-100K",
    "$100K+")

Goal Band Sort =
SWITCH(TRUE(),
    F_Campaigns[goal_usd] < 1000,   1,
    F_Campaigns[goal_usd] < 5000,   2,
    F_Campaigns[goal_usd] < 10000,  3,
    F_Campaigns[goal_usd] < 25000,  4,
    F_Campaigns[goal_usd] < 50000,  5,
    F_Campaigns[goal_usd] < 100000, 6,
    7)

Duration Band =
SWITCH(TRUE(),
    F_Campaigns[duration_days] <= 15, "Up to 15 days",
    F_Campaigns[duration_days] <= 30, "16-30 days",
    F_Campaigns[duration_days] <= 45, "31-45 days",
    F_Campaigns[duration_days] <= 60, "46-60 days",
    "Over 60 days")

Duration Band Sort =
SWITCH(TRUE(),
    F_Campaigns[duration_days] <= 15, 1,
    F_Campaigns[duration_days] <= 30, 2,
    F_Campaigns[duration_days] <= 45, 3,
    F_Campaigns[duration_days] <= 60, 4,
    5)
```

---

## 6. Review notes and suggested fixes

1. **`FundingLY%` is the same as `Funding%`.** It has no last-year filter, so it returns this year's value. If you use it in a visual, replace it with:
   ```dax
   FundingLY% = DIVIDE([PledgeAmountLY(Campaign)], [GoalAmountLY])
   ```
2. **`TotalPledge` has a misleading name.** It counts campaigns with pledge detail and not pledges. Consider renaming it to `CampaignsWithPledgeDetail`, and `TotalPledge YOY%` and `TotalPledgeLY` to match.
3. **Naming style is mixed.** Some names use `LY` and `YOY%`, others use parentheses such as `(Campaign)` and `(Detail)`. This works, but a single convention (for example `Pledged USD`, `Pledged USD LY`) is easier to maintain.
4. **Year-over-year needs a complete year.** The data ends in January 2018, so 2018 has only a few campaigns. The dashboard's year slicer (2009 to 2017) avoids this, so keep it.
5. **Add display formats.** The format strings are blank in the model. Set `SuccessRate`, `Funding%` and the YOY measures to percentage, and the amount measures to a currency format.
6. **Add measures to a display folder** (for example Core, Last Year, YoY, Pledge Detail) to make the field list easier to scan.

---

## 7. Measure index

| Group | Measures | Count |
|---|---|---|
| Core campaign | TotalCampaigns, SuccessfulCampaigns, SuccessRate, SuccessRate(min 100), GoalAmount, PledgeAmount(Campaign), Funding% | 7 |
| Pledge detail (simulated) | Pledge(Detail), TotalPledge(Detail), Avg Pledge, TotalPledge | 4 |
| Last year | TotalCampaignsLY, GoalAmountLY, PledgeAmountLY(Campaign), SuccessRateLY, FundingLY% | 5 |
| Last year, pledge detail | PledgeLY(Detail), TotalPledgeLY(Detail), Avg Pledge LY, TotalPledgeLY | 4 |
| Year-over-year | TotalCampaigns YOY%, Goal YOY%, PledgeAmount YOY%, SuccessRate YOY%, TotalPledge YOY% | 5 |
| **Total** | | **25** |
