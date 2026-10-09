# Crowdfunding Campaign Performance Dashboard

An interactive Power BI dashboard that analyses **370,000 Kickstarter campaigns (2009 to 2018)** to answer one question: *what goal size and campaign length give a project the best chance of success?*

![Dashboard preview](images/dashboard.png)

## Key numbers

| Metric | Value |
|---|---|
| Campaigns analysed | 370,216 |
| Total funding goals | $16.69B |
| Total pledged | $3.39B |
| Overall success rate | 36.2% |
| Average campaign length | 34.2 days (median 30) |

## Business questions

1. What goal size and campaign length give the best chance of success?
2. Which categories and countries perform best?
3. How has the success rate changed over time?

## Key findings

- **Goal size is the strongest factor.** Success falls from 51% for goals under $1K to 8% for goals of $100K+.
- **31 to 45 days is the sweet spot.** It has the highest success rate in every goal band from $1K up. Longer campaigns do worse.
- **Best combination:** goals under $1K with 15 days or fewer succeed 59.4% of the time. **Worst:** $100K+ goals with 15 days or fewer, at 2.4%.
- **Category matters.** Dance (63%), Theater (60%) and Comics (55%) lead. Technology (20%), Journalism (22%) and Crafts (24%) trail.
- **Failed campaigns rarely come close.** 73% of failed campaigns raised under 10% of their goal.
- Success rates dropped sharply around 2014 to 2015 and partly recovered afterwards.

These are patterns in historical data, not guarantees. A campaign does not succeed because of its length alone.

## Dashboard features

- KPI cards with sparklines and year-over-year change
- Heatmap matrix of success rate by goal band and campaign length, with a written insight
- Decomposition tree to explore success by goal band, duration, category and country
- Success rate trend by year, quarter and month
- Year slicer (2009 to 2017)

## Data

**Source:** the public [Kickstarter Projects](https://www.kaggle.com/datasets/kemical/kickstarter-projects) dataset on Kaggle (`ks-projects-201801.csv`). Please check the dataset page for its license and credit the original author.

**Cleaning** (`scripts/generate_data.py`):
- Kept successful, failed and canceled campaigns
- Removed placeholder 1970 launch dates, invalid country codes and invalid durations
- 370,216 of 378,661 rows kept
- Converted all amounts to USD using the dataset's `usd_pledged_real` and `usd_goal_real` fields

**Simulated pledge data.** The Kaggle file only has campaign totals. To support backer-behaviour analysis, the script also generates an individual-pledges table (`F_Pledges`) for 5,000 campaigns with 500 or fewer backers. Amounts add up exactly to each campaign's real pledged total, and the timing follows a launch-rush, mid-campaign and final-surge pattern. **This table is simulated and is not real backer data.** The findings above use only the real campaign data.

## Data model

A star schema with two fact tables and four dimensions.

| Table | Type | Description |
|---|---|---|
| `F_Campaigns` | Fact | One row per campaign: goal, pledged, backers, dates, outcome |
| `F_Pledges` | Fact (simulated) | One row per generated pledge |
| `D_Category` | Dimension | Main category and subcategory |
| `D_Country` | Dimension | Country and region |
| `D_Backer` | Dimension (simulated) | Generated backers |
| `D_Date` | Dimension | Calendar table |

Relationships (many to one): `F_Campaigns` to `D_Category`, `D_Country` and `D_Date` (on launch date); `F_Pledges` to `F_Campaigns`, `D_Backer` and `D_Date`.

## DAX measures

| Measure | Logic |
|---|---|
| TotalCampaigns | Count of campaigns |
| GoalAmount | Sum of `goal_usd` |
| PledgeAmount | Sum of `pledged_usd` |
| SuccessRate | Successful campaigns divided by total campaigns |
| YoY % | (Current year - previous year) divided by previous year, using time intelligence |
| Goal Band, Duration Band | Calculated columns that group goals and campaign lengths into bands for the matrix |

## Repository structure

```
crowdfunding-campaign-performance-dashboard/
├── README.md
├── powerbi/
│   └── CrowdFundingCampaignPerformanceDashboard.pbix
├── scripts/
│   └── generate_data.py      # cleaning and pledge generation
├── sql/
│   └── schema.sql            # optional MySQL schema and load script
├── data/
│   └── README.md             # where to download the raw file
├── images/
│   └── dashboard.png
└── docs/
    └── linkedin_carousel.pdf
```

The raw Kaggle file (58 MB) is not stored in this repository. Download it from Kaggle.

## How to reproduce

1. Download `ks-projects-201801.csv` from Kaggle and place it in `data/`.
2. Install the libraries: `pip install pandas numpy`
3. Generate the tables:
   ```
   python scripts/generate_data.py --input data/ks-projects-201801.csv --outdir data/processed
   ```
4. (Optional) Load the tables into MySQL by editing the file paths in `sql/schema.sql` and running it.
5. Open the `.pbix` file in Power BI Desktop, go to **Transform data > Data source settings**, and point the sources to `data/processed/`.
6. Click **Refresh**.

Options for the script: `--sample` (campaigns with pledge detail, default 5000), `--max-backers` (default 500) and `--seed` (default 42, for repeatable output).

## Limitations

- The data ends in January 2018, so results are historical.
- Success is based on reaching the funding goal. It does not measure whether the project was delivered.
- Goal and campaign length are associated with success. This analysis does not show that they cause it.
- Pledge-level data is simulated (see above).

## Tools

Power BI, DAX, Python (pandas, NumPy), MySQL (optional)

## Roadmap

- Backer-behaviour page: one-time vs. repeat backers, average pledge, pledge size by category
- Time-to-goal page: funding curve and when pledges arrive
- Category deep-dives

## Author

**Maitreyee** - Power BI and Full Stack Developer (Laravel, MySQL, Vue.js)

LinkedIn: *add your profile link*
