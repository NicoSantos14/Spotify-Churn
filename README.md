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
* `age`, `country`, `device_type`

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
#Project Summary

I chose this project because understanding user engagement and churn is a critical challenge for any subscription-based service like Spotify. It allows an analyst to explore real-world patterns in user behavior and apply practical data skills such as data cleaning, segmentation, and SQL analysis. This project also demonstrates the ability to link behavioral metrics to actionable business insights, which is a key component of product-focused data analytics.

The findings highlight several important insights: Premium users are more engaged and less likely to churn, low engagement is the strongest predictor of churn, and offline listening strongly correlates with retention. Additionally, ad exposure impacts Free users’ churn, and patterns vary by age and region. Collectively, these results convey the importance of monitoring user behavior and tailoring retention strategies to both subscription type and demographic segments.

#Proposed Solution

To address customer churn, the platform can implement targeted retention strategies based on the findings. For Free users, reducing ad overload or personalizing ad experiences could decrease churn. Encouraging offline listening through feature promotion may improve retention across both Free and Premium users. Additionally, engagement-based interventions, such as recommending playlists or sending reminders to less active users, could help increase listening time and reduce churn risk. Tailoring campaigns by age and region ensures that retention efforts are optimized for the most at-risk segments.
## Author

Nicolas Nunez
Data Analyst | SQL • Excel
