# Spotify-Churn
# Spotify User Engagement & Churn Analysis

This project explores how user behavior differs between Free and Premium Spotify users, with a focus on engagement patterns and churn. The analysis follows a practical workflow using **Excel for data preparation** and **SQL for analysis**, similar to how product and analytics teams operate in real environments.

---

## Project Overview

The goal of this analysis is to understand which user behaviors are most closely associated with churn and how those behaviors differ across subscription types. The findings can be used to inform retention strategies and highlight features that encourage long-term usage.

Key questions explored include:

* How does engagement differ between Free and Premium users?
* Are churn rates meaningfully different by subscription type?
* What behavioral signals appear before a user churns?
* How do ads, offline listening, and demographics relate to retention?

---

## Business Context

User retention is a core driver of growth for streaming platforms. While Premium users generate direct revenue, Free users represent both churn risk and conversion opportunity. Understanding how listening behavior, ad exposure, and feature usage relate to churn helps product and marketing teams prioritize the most impactful improvements.

---

## Dataset

**Table:** `spotify_data_project`

The dataset contains user-level listening and behavior metrics.

Key fields used in the analysis:

* `user_id`
* `subscription_type` (Free / Premium)
* `listening_time`
* `songs_played_per_day`
* `skip_rate`
* `ads_listened_per_week`
* `offline_listening`
* `is_churned`
* `age`
* `country`
* `device_type`

Churn is inferred based on inactivity, and the data represents a snapshot rather than a full user lifecycle.

---

## Data Preparation (Excel)

Excel was used to review and prepare the raw data before analysis. This step focused on making the dataset consistent and reliable rather than adding new logic.

Preparation steps included:

* Removing duplicate user records
* Standardizing text fields such as subscription type and device
* Ensuring numeric columns were correctly typed
* Creating simple helper fields (e.g., age groups) for later analysis

Once cleaned, the data was loaded into the SQL environment.

---

## SQL Analysis

All analysis was performed in SQL. Queries were written to directly answer specific business questions, with a focus on clarity and reproducibility. Representative queries are shown below; full scripts are available in the `/sql` directory.

---

### Engagement by Subscription Type

This query compares overall engagement between Free and Premium users.

```sql
SELECT
  subscription_type,
  COUNT(*) AS users,
  ROUND(AVG(listening_time), 1) AS avg_listening_time,
  ROUND(AVG(songs_played_per_day), 1) AS avg_songs_per_day,
  ROUND(AVG(skip_rate), 2) AS avg_skip_rate
FROM spotify_data_project
GROUP BY subscription_type;
```

Premium users consistently show higher listening time, play more songs per day, and skip fewer tracks, indicating stronger engagement.

---

### Churn Rate by Subscription Type

To understand whether engagement differences translate into retention, churn rates were calculated by subscription tier.

```sql
SELECT
  subscription_type,
  COUNT(*) AS total_users,
  SUM(CASE WHEN is_churned = 1 THEN 1 ELSE 0 END) AS churned_users,
  ROUND(
    100.0 * SUM(CASE WHEN is_churned = 1 THEN 1 ELSE 0 END) / COUNT(*),
    2
  ) AS churn_rate_pct
FROM spotify_data_project
GROUP BY subscription_type;
```

Free users churn at a noticeably higher rate than Premium users, reinforcing the link between paid features and retention.

---

### Churned vs Retained User Behavior

This comparison looks at how behavior differs between users who churned and those who did not.

```sql
SELECT
  is_churned,
  ROUND(AVG(listening_time), 1) AS avg_listening_time,
  ROUND(AVG(skip_rate), 2) AS avg_skip_rate,
  ROUND(AVG(ads_listened_per_week), 1) AS avg_ads
FROM spotify_data_project
GROUP BY is_churned;
```

Churned users listen significantly less overall, while skip rates remain similar. This suggests disengagement, rather than frustration, is a primary churn signal.

---

### Ad Exposure and Churn (Free Users)

Free users were grouped into ad exposure buckets to evaluate whether ad load impacts retention.

```sql
WITH ad_buckets AS (
  SELECT
    CASE
      WHEN ads_listened_per_week <= 5 THEN 'Low Ads'
      WHEN ads_listened_per_week BETWEEN 6 AND 15 THEN 'Medium Ads'
      ELSE 'High Ads'
    END AS ad_exposure,
    is_churned,
    listening_time
  FROM spotify_data_project
  WHERE subscription_type = 'Free'
)
SELECT
  ad_exposure,
  COUNT(*) AS users,
  ROUND(AVG(CAST(is_churned AS FLOAT)) * 100, 2) AS churn_rate_pct,
  ROUND(AVG(listening_time), 1) AS avg_listening_time
FROM ad_buckets
GROUP BY ad_exposure
ORDER BY churn_rate_pct DESC;
```

Higher ad exposure is associated with both lower listening time and higher churn among Free users.

---

### Offline Listening and Retention

Offline listening usage was evaluated as a potential retention signal.

```sql
SELECT
  offline_listening,
  COUNT(*) AS users,
  ROUND(AVG(CAST(is_churned AS FLOAT)) * 100, 2) AS churn_rate_pct,
  ROUND(AVG(listening_time), 1) AS avg_listening_time
FROM spotify_data_project
GROUP BY offline_listening;
```

Users who use offline listening churn at substantially lower rates, making this one of the strongest retention indicators in the dataset.

---

### Churn by Age Group and Country

Users were segmented by age group and country to surface demographic patterns.

```sql
WITH age_buckets AS (
  SELECT
    country,
    CASE
      WHEN age < 18 THEN '<18'
      WHEN age BETWEEN 18 AND 24 THEN '18–24'
      WHEN age BETWEEN 25 AND 34 THEN '25–34'
      WHEN age BETWEEN 35 AND 44 THEN '35–44'
      ELSE '45+'
    END AS age_group,
    is_churned
  FROM spotify_data_project
)
SELECT
  country,
  age_group,
  ROUND(AVG(CAST(is_churned AS FLOAT)) * 100, 2) AS churn_rate_pct
FROM age_buckets
GROUP BY country, age_group
ORDER BY churn_rate_pct DESC;
```

Churn varies across regions and age groups, suggesting that retention strategies should not be one-size-fits-all.

---

### High-Engagement Retained Users

To define the most valuable retained users, the analysis filtered for high listening time, low skip rates, and no churn.

```sql
SELECT
  subscription_type,
  device_type,
  COUNT(*) AS users,
  ROUND(AVG(listening_time), 1) AS avg_listening_time,
  ROUND(AVG(skip_rate), 2) AS avg_skip_rate
FROM spotify_data_project
WHERE
  listening_time >= 120
  AND skip_rate < 20
  AND is_churned = 0
GROUP BY subscription_type, device_type
ORDER BY users DESC;
```

These users are predominantly Premium subscribers and tend to cluster around specific device types.

---

## Key Takeaways

* Premium users are more engaged and significantly less likely to churn
* Low engagement is the clearest signal of future churn
* High ad exposure increases churn risk among Free users
* Offline listening is strongly associated with retention
* Demographic and regional differences matter

---

## Repository Structure

```
spotify-churn-analysis/
├── data/   # Cleaned Excel files
├── sql/    # SQL analysis scripts
├── README.md
```

---
# Project Summary

I selected this project to explore user engagement and churn—two of the most critical challenges for subscription-based platforms such as Spotify. The analysis provides an opportunity to investigate real-world user behavior while applying essential data analytics skills, including data cleaning, user segmentation, and SQL-based analysis. More importantly, the project demonstrates how behavioral metrics can be translated into actionable business insights, a core competency in product-focused data analytics.

The analysis revealed several key findings. Premium users show significantly higher engagement levels and lower churn rates compared to Free users. Low engagement emerged as the strongest predictor of churn, while offline listening displayed a strong positive relationship with user retention. Among Free users, higher ad exposure was associated with increased churn risk, and behavioral patterns varied notably across different age groups and geographic regions. Together, these insights underscore the importance of closely monitoring user behavior and developing tailored retention strategies based on subscription type and demographic characteristics.

# Proposed Solution

To reduce customer churn, the platform should adopt targeted, data-driven retention strategies aligned with these findings. For Free users, minimizing ad fatigue through reduced frequency or more personalized advertising experiences may help improve retention. Promoting offline listening features could further enhance engagement and retention for both Free and Premium users. Additionally, engagement-based interventions—such as personalized playlist recommendations or timely re-engagement notifications for less active users—can encourage increased listening activity and lower churn risk. Finally, tailoring retention campaigns by age group and region ensures that interventions are focused on the most vulnerable user segments, maximizing their overall impact.
## Author

Nicolas Nunez
Data Analyst | SQL • Excel









