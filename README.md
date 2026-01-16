# Spotify-Churn
# 🎧 Spotify User Engagement & Churn Analysis

> **End-to-end data analysis project** using **Excel for data cleaning**, **SQL for querying**, and **Power BI for visualization** to understand user engagement patterns and churn behavior across Spotify subscription types.

---

## 🔍 Project Overview

**Goal:** Identify behavioral differences between Free and Premium users and determine which factors are most strongly associated with user churn.

**Key Questions Answered:**

* How do engagement and skip behavior differ between Free and Premium users?
* How does churn vary by subscription type?
* What behavioral patterns distinguish churned vs non-churned users?
* Does ad exposure impact churn for Free users?
* Is offline listening associated with retention?
* How does churn vary by age group and country?
* Which high-engagement user segments show the strongest retention?

---

## 🧠 Business Context

Spotify’s growth depends on **retaining users and converting Free users to Premium**. Understanding how engagement, ads, offline listening, and demographics relate to churn helps:

* Product teams prioritize features
* Marketing teams target retention campaigns
* Leadership evaluate subscription strategy

This analysis simulates a **real product analytics use case**.

---

## 📁 Dataset

**Table:** `spotify_data_project`

**Key Columns:**

* `user_id` – unique user identifier
* `subscription_type` – Free or Premium
* `listening_time` – average daily listening minutes
* `songs_played_per_day` – engagement intensity
* `skip_rate` – percentage of skipped songs
* `ads_listened_per_week` – ad exposure (Free users)
* `offline_listening` – offline usage flag
* `is_churned` – churn indicator (1 = churned)
* `age`, `country`, `device_type`

**Assumptions & Limitations:**

* Churn is inferred from inactivity
* Dataset represents a snapshot in time
* Ads only apply to Free users

---

## 🧹 Excel: Data Cleaning & Formatting

Excel was used for **initial inspection and preparation** before loading into SQL.

**Steps Performed:**

* Removed duplicate user records
* Standardized text fields (subscription type, device type)
* Converted dates and numeric fields to consistent formats
* Created helper columns (age bands, engagement checks)

**Outcome:** Clean, analysis-ready dataset loaded into the SQL environment.

---

## 🗄️ SQL: Analysis & Querying

SQL is the **core analysis layer** for this project. Each section below maps a **business question → SQL logic → insight**. Only representative queries are shown to keep the README readable; full scripts live in the `/sql` folder.

---

### 1️⃣ Engagement by Subscription Type

**Business Question:**
Do Premium users show higher engagement and better listening behavior than Free users?

**SQL Used:**

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

**Insight:**
Premium users listen longer, play more songs per day, and skip fewer tracks than Free users, indicating substantially higher engagement.

---

### 2️⃣ Churn Rate by Subscription Type

**Business Question:**
Does churn differ between Free and Premium users?

**SQL Used:**

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

**Insight:**
Free users churn at a significantly higher rate than Premium users, reinforcing the retention value of Premium features.

---

### 3️⃣ Behavior of Churned vs Non-Churned Users

**Business Question:**
How does behavior differ between churned and retained users?

**SQL Used:**

```sql
SELECT
  is_churned,
  ROUND(AVG(listening_time), 1) AS avg_listening_time,
  ROUND(AVG(skip_rate), 2) AS avg_skip_rate,
  ROUND(AVG(ads_listened_per_week), 1) AS avg_ads
FROM spotify_data_project
GROUP BY is_churned;
```

**Insight:**
Churned users listen far less, while skip rates remain similar. This suggests churn is driven more by disengagement than frustration.

---

### 4️⃣ Ad Exposure Impact (Free Users)

**Business Question:**
Does higher ad exposure increase churn among Free users?

**SQL Used:**

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

**Insight:**
Higher ad exposure is associated with higher churn and lower listening time, indicating ad load may negatively impact retention.

---

### 5️⃣ Offline Listening & Retention

**Business Question:**
Is offline listening associated with lower churn?

**SQL Used:**

```sql
SELECT
  offline_listening,
  COUNT(*) AS users,
  ROUND(AVG(CAST(is_churned AS FLOAT)) * 100, 2) AS churn_rate_pct,
  ROUND(AVG(listening_time), 1) AS avg_listening_time
FROM spotify_data_project
GROUP BY offline_listening;
```

**Insight:**
Users who use offline listening churn at significantly lower rates, making this one of the strongest retention signals in the dataset.

---

### 6️⃣ Churn by Age Group & Country

**Business Question:**
How does churn vary across demographics and regions?

**SQL Used:**

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

**Insight:**
Churn varies meaningfully by age group and country, highlighting the need for region- and demographic-specific retention strategies.

---

### 7️⃣ High-Engagement Retained Users

**Business Question:**
What does the ideal retained user look like?

**SQL Used:**

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

**Insight:**
High-engagement retained users are predominantly Premium subscribers and cluster around specific device types, defining Spotify’s most valuable segment.

---

### 2️⃣ Churn Rate by Subscription Type

**Insight:** Free users churn at a **significantly higher rate** than Premium users.

This supports the hypothesis that Premium features contribute to retention.

---

### 3️⃣ Behavior of Churned vs Non-Churned Users

**Insight:**

* Churned users listen substantially less
* Skip rates remain similar
* Ad exposure alone does not fully explain churn

This suggests churn is driven more by **low engagement** than frustration behavior.

---

### 4️⃣ Ad Exposure Impact (Free Users Only)

Users were bucketed into **Low, Medium, and High ad exposure** groups.

**Insights:**

* Higher ad exposure is associated with **higher churn rates**
* Users with high ad exposure also show lower listening time

Ad load appears to negatively impact retention for Free users.

---

### 5️⃣ Offline Listening & Retention

**Key Finding:**

> Users who enable offline listening churn at **significantly lower rates**.

Offline listening is one of the **strongest retention signals** observed.

---

### 6️⃣ Churn by Age Group & Country

Users were grouped into age buckets across countries.

**Insights:**

* Younger age groups tend to churn more
* Churn patterns vary noticeably by region

This highlights opportunities for **region- and age-specific retention strategies**.

---

### 7️⃣ High-Engagement Retained Users

Filtered for users who:

* Listen ≥ 120 minutes/day
* Skip < 20%
* Are not churned

**Insight:**

* Premium users dominate this segment
* Certain device types show consistently higher engagement

This defines Spotify’s **ideal retained user profile**.

---

## 🔑 Key Insights Summary

* Premium users are significantly more engaged and churn less
* Low engagement is the strongest churn indicator
* High ad exposure increases churn risk for Free users
* Offline listening is strongly correlated with retention
* Younger users and certain regions show higher churn

---

## 💡 Recommendations

* Encourage Free users to adopt Premium features tied to retention (offline listening)
* Reduce ad load or optimize ad experience for highly engaged Free users
* Target retention campaigns at low-engagement users early
* Customize strategies by age group and region

---

## 📂 Repository Structure

```
spotify-churn-analysis/
│
├── data/              # Cleaned Excel files
├── sql/               # SQL analysis scripts
├── powerbi/           # Power BI dashboard (.pbix)
├── visuals/           # Dashboard screenshots
├── README.md
```

---

## 🚀 How to Reproduce

1. Review cleaned Excel file in `/data`
2. Load data into SQL database
3. Run SQL scripts in `/sql`
4. Open Power BI file and refresh data

---

## 👤 Author

**Nick Santos**
Aspiring Data Analyst | SQL • Excel • Power BI

---

⭐ *This project demonstrates practical, business-focused analytics using industry-standard tools.*
