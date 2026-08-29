# Marketing Campaign Performance (Multi-Source Reconciliation)

## The Brief (simulated client request)
> "We run ads on Google and Facebook and want to know which campaigns are actually driving revenue, not just clicks. Marketing keeps asking for more budget on Facebook — is that justified? Also, our sales team says some conversions aren't showing up as tied to any campaign, which worries them."

## Final Result

Link: https://app.powerbi.com/view?r=eyJrIjoiYjBiMjIyNjgtYmE2OS00YmNhLTllMmMtN2Q1NmUxZDlkZDRjIiwidCI6IjcwYTI4NTIyLTk2OWItNDUxZi1iZGIyLWFiZmVhM2FhYTViZiIsImMiOjl9

Images:
![Page 1](/images/image_1.png)
![Page 2](/images/image_2.png)

## Data
Three separate exports, deliberately inconsistent with each other — the core challenge of this project:
- **google_ads_performance.csv** — clean column names, standard date format
- **facebook_ads_performance.csv** — different column names for the same concepts (`report_date`, `campaign`, `impressions_count`, `link_clicks`, `amount_spent_usd`), and dates in `MM/DD/YYYY` instead of `YYYY-MM-DD`
- **crm_conversions.csv** — hand-typed campaign names from the sales team, inconsistent abbreviations/spellings, plus a set of conversions genuinely not tied to any paid campaign at all (organic search, direct traffic, referrals, walk-ins)

## Skills Used
- SQL: `CASE WHEN` + `LIKE` pattern matching to standardize inconsistent campaign name spellings across three independent sources
- Multi-source `FULL OUTER JOIN`, and — critically — understanding **join-key uniqueness** and the row-multiplication  risk of joining tables on a non-unique column
- Column renaming and schema alignment (`ALTER TABLE`) across sources with different naming conventions


## My Interpretation & Key Decisions
- Standardized campaign names across all three sources so they could be compared on equal footing.
- Treated conversions with no matching campaign (organic, direct, referral, walk-in) as their own legitimate category — **not** a cleaning problem to force-match, since they represent real, valid business events that never touched a paid campaign.
- When a campaign ran on **both** Google and Facebook simultaneously, and CRM only recorded the campaign name (not which platform drove the conversion), I could not cleanly split that revenue between platforms. Rather than fabricate a split, I stated this as a known limitation of the data and recommended the client add platform-level tracking to conversions going forward.

## Key Challenges & Fixes
- **Row-multiplication bug (the big one)**: an early version of the combined table joined Google, Facebook, and CRM data directly on `campaign` — but `campaign` isn't a unique key in any of these tables (each campaign has many daily ad rows and many individual conversions). Joining on a non-unique key doesn't match rows 1-to-1; it matches *every* row on one side to *every* matching row on the other, multiplying the result. Verified this by simulating the join on a single campaign: 29 real conversions became 27,550 rows after the join, inflating summed revenue by roughly 950x. **Fixed by aggregating each source down to one row per campaign first** (`SUM`/`COUNT` inside a subquery, grouped by campaign), then joining those clean summaries — eliminating the possibility of 

## Final Insight
Facebook Ads shows a meaningfully lower cost for a comparable share of engagement relative to Google (roughly 70% of Google's clicks at under half the spend), suggesting efficient reach — but exact revenue attribution by platform isn't possible with the current CRM tracking when a campaign runs on both platforms at once. Recommended the client add platform-level conversion tracking to answer the budget question with precision going forward.

## Warning
To open this file correctly, go to Power BI settings and change path to your local path as by default project uses my local path.
