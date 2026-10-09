# Case Study: What Makes a Crowdfunding Campaign Succeed?

**Project:** Crowdfunding Campaign Performance Dashboard (Power BI)
**Author:** Maitreyee
**Data:** 370,216 Kickstarter campaigns, April 2009 to January 2018

---

## 1. Background

Crowdfunding lets creators raise money from many small backers, but the odds are poor: only about one campaign in three reaches its goal. Creators choose two things before they launch that they fully control: **how much to ask for** and **how long to run the campaign**.

I have worked on crowdfunding applications as a full stack developer, so I knew how these platforms work technically. This project looks at the other side: what the outcome data says about which campaigns succeed.

## 2. Business problem

A creator, a platform team or an advisor has no quick way to answer:

1. What goal size and campaign length give the best chance of success?
2. Which categories and countries perform best, and which struggle?
3. How has the success rate changed over time?

## 3. Objectives

- Build an interactive Power BI dashboard that answers the three questions above.
- Show the combined effect of goal size and campaign length in one view.
- Turn the numbers into plain-English guidance for creators.

## 4. Data and approach

| Item | Detail |
|---|---|
| Source | Public Kickstarter Projects dataset on Kaggle (`ks-projects-201801.csv`) |
| Cleaning | Kept successful, failed and canceled campaigns. Removed placeholder 1970 dates, invalid countries and invalid durations. 370,216 of 378,661 rows kept |
| Tools | Power BI and DAX (model and dashboard), MySQL (optional load) |
| Model | Star schema: campaign facts with category, country and date dimensions |
| Measures | Success rate, funding %, goal and pledge totals, last-year and year-over-year change (25 DAX measures) |
| Bands | Goals grouped into 7 bands and campaign lengths into 5, for the heatmap matrix |

**Definitions**
- *Successful:* the campaign reached its funding goal by the deadline.
- *Success rate:* successful campaigns divided by all campaigns. Failed and canceled campaigns both count as not successful.
- *Funding %:* amount pledged divided by the goal.

**Dashboard design:** KPI cards with year-over-year change, a heatmap matrix of success rate by goal band and campaign length (with a written insight), a decomposition tree for exploring success by goal, duration, category and country, trend charts, and a year slicer.

## 5. Key findings

### 5.1 Overall picture

| Metric | Value |
|---|---|
| Campaigns | 370,216 |
| Total funding goals | $16.69B |
| Total pledged | $3.39B |
| Success rate | 36.2% |
| Average campaign length | 34.2 days (median 30; 45% run exactly 30 days) |

Of the campaigns, 133,851 succeeded, 197,614 failed and 38,751 were canceled. Goals added up to about five times the money actually pledged.

### 5.2 Goal size is the strongest factor

| Goal size | Success rate | Share of campaigns |
|---|---|---|
| Under $1K | 51% | 12.7% |
| $1K to $5K | 45% | 30.0% |
| $5K to $10K | 37% | 18.5% |
| $10K to $25K | 31% | 20.2% |
| $25K to $50K | 23% | 8.9% |
| $50K to $100K | 16% | 5.4% |
| $100K+ | 8% | 4.3% |

- Success falls steadily with every step up in goal size.
- The median goal of successful campaigns was **$3,840**, against **$7,500** for failed ones.

### 5.3 Campaign length: 31 to 45 days is the sweet spot

- **31 to 45 days has the highest success rate in every goal band from $1K up.** Longer campaigns do worse.
- For goals under $1K, shorter campaigns (up to 15 days) do slightly better, at 59.4% against 55.4%.
- Example: for $10K to $25K goals, success is 38.9% at 31 to 45 days, against 30.0% at 16 to 30 days and 20.8% at 46 to 60 days.

### 5.4 Best and worst combinations

| | Goal | Length | Success rate |
|---|---|---|---|
| Best | Under $1K | Up to 15 days | **59.4%** |
| Worst | $100K+ | Up to 15 days | **2.4%** |

Ambitious goals need the right length: for $25K to $50K goals, success is 30.1% at 31 to 45 days but 13.9% at 46 to 60 days.

### 5.5 Categories

| Highest | Success | Lowest | Success |
|---|---|---|---|
| Dance | 63% | Technology | 20% |
| Theater | 60% | Journalism | 22% |
| Comics | 55% | Crafts | 24% |

At subcategory level (500+ campaigns), Anthologies, Dance and Indie Rock exceed 64%, while **Apps (6%), Mobile Games (9%) and Web (9%)** are the hardest.

### 5.6 Countries and regions

- The **United States (38%)** and **United Kingdom (36%)** lead among the larger markets, followed by France (32%) and Canada (29%).
- **Italy (16%), Austria (19%), the Netherlands (22%) and Spain (22%)** are lowest.
- By region: North America 37%, Europe 32%, Oceania 27%.
- Small markets such as Hong Kong (564 campaigns) should be read with care.

### 5.7 Trend over time

| Launch year | Campaigns | Success rate |
|---|---|---|
| 2009 to 2013 | 1K to 45K per year | 43% to 47% |
| 2014 | 66,723 | 31.6% |
| 2015 | 74,198 | 28.3% |
| 2016 | 56,194 | 33.2% |
| 2017 | 49,185 | 37.5% |

The success rate dropped sharply in 2014 and 2015, when the number of campaigns jumped, and has partly recovered since.

### 5.8 Failed campaigns are rarely close

- Failed campaigns raised a median of **1.7% of their goal**, and 73% raised under 10%.
- Only 0.6% of failed campaigns reached 75% of their goal.
- Successful campaigns typically overfunded: the median reached 117% of the goal.

Most campaigns either clear their goal or stay far from it. Near misses are rare.

## 6. Recommendations

**For creators**
1. **Set a realistic goal.** It is the strongest factor, and half of all successful campaigns asked for under $3,840.
2. **Plan 31 to 45 days** for most goals, and avoid campaigns longer than 45 days.
3. **Check your category's baseline** before launch. A Technology or Apps project should plan for much lower odds than Dance or Theater.
4. **Test early.** Since failed campaigns rarely come close, validate demand before launching.

**For platforms and advisors**
1. Offer guidance that suggests a goal and length based on similar past campaigns.
2. Give extra support to the weakest categories (Technology, Apps, Journalism, Crafts).
3. Investigate why success rates fell in 2014 and 2015 and what helped the recovery.

## 7. Limitations

- The data ends in January 2018, so the findings are historical.
- Goal size and length are **associated** with success. The analysis does not show that they cause it. For example, experienced creators may choose different goals and lengths.
- Success means reaching the funding goal. It does not measure whether the project was delivered.
- Small groups (some countries and subcategories) give unreliable rates, so the dashboard has a `SuccessRate(min 100)` measure that hides groups under 100 campaigns.
- The model also contains **simulated** pledge-level data (a generated `F_Pledges` table) for backer-behaviour analysis. **None of the findings above use it.**

## 8. Next steps

- Add a backer-behaviour page (one-time vs. repeat backers, average pledge, pledge size by category). It should be labelled as simulated data.
- Add a time-to-goal page showing the funding curve.
- Add category deep-dives with the goal and length matrix by category.
- Add a what-if tool where a creator enters a goal and length and sees the historical success rate.

## 9. Skills demonstrated

Data cleaning and preparation, star-schema data modelling, DAX (time intelligence, ratio and year-over-year measures, calculated columns), dashboard and KPI design, data storytelling, and clearly stating the limits of a dataset.
