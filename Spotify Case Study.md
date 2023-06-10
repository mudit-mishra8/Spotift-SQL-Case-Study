![alt text](https://helios-i.mashable.com/imagery/articles/050bXhdmNaL9bDGAINptcrL/hero-image.fill.size_1248x702.v1617973265.jpg)

## Introduction

Spotify is a digital music, podcast, and video streaming service that gives users access to millions of songs and other content from artists all over the world. User activities such as app installations, in-app purchases, and location data can be useful in making data-driven decisions. In this case study, we will analyze a sample dataset of user activity on Spotify using MySQL.

## Dataset

The dataset contains records of user activity. The columns are:
- `user_id`: The unique identifier of a user.
- `event_name`: The name of the event (e.g., app-installed, app-purchase).
- `event_date`: The date of the event.
- `country`: The country from which the user is using the app.

Here's how the dataset looks like:

| user_id | event_name   | event_date | country  |
|---------|--------------|------------|----------|
| 1       | app-installed | 2022-01-01 | India    |
| 1       | app-purchase  | 2022-01-02 | India    |
| 2       | app-installed | 2022-01-01 | USA      |
| 3       | app-installed | 2022-01-01 | USA      |
| 3       | app-purchase  | 2022-01-03 | USA      |
| 4       | app-installed | 2022-01-03 | India    |
| 4       | app-purchase  | 2022-01-03 | India    |
| 5       | app-installed | 2022-01-03 | SL       |
| 5       | app-purchase  | 2022-01-03 | SL       |
| 6       | app-installed | 2022-01-04 | Pakistan |
| 6       | app-purchase  | 2022-01-04 | Pakistan |


## Questions and Analysis

### Question 1: How many users installed the app each day?

```sql
SELECT
    event_date,
    COUNT(distinct user_id) AS installations
FROM
    activity
WHERE
    event_name = 'app-installed'
GROUP BY
    event_date
ORDER BY
    event_date;
```

**Expected Output:**

```plaintext
| event_date | installations |
|------------|---------------|
| 2022-01-01 |             3 |
| 2022-01-03 |             2 |
| 2022-01-04 |             1 |
```

**Analysis:**

There were 3 installations on January 1st, 2 installations on January 3rd, and 1 installation on January 4th. This shows how the installations are distributed over the days.

### Question 2: How many users made a purchase each day?

```sql
SELECT
    event_date,
    COUNT(distinct user_id) AS purchases
FROM
    activity
WHERE
    event_name = 'app-purchase'
GROUP BY
    event_date
ORDER BY
    event_date;
```

**Expected Output:**

```plaintext
| event_date | purchases |
|------------|-----------|
| 2022-01-02 |         1 |
| 2022-01-03 |         3 |
| 2022-01-04 |         1 |
```

**Analysis:**

On January 2nd, there was 1 purchase, while on January 3rd there were 3 purchases, and on January 4th there was 1 purchase. This can provide insights into which days have higher purchase activity.

### Question 3: Find the total active users each day.

```sql
SELECT
    event_date,
    COUNT(DISTINCT user_id) AS active_users
FROM
    activity
GROUP BY
    event_date
ORDER BY
    event_date;
```

**Expected Output:**

```plaintext
| event_date | active_users |
|------------|--------------|
| 2022-01-01 |            3 |
| 2022-01-02 |            1 |
| 2022-01-03 |            3 |
| 2022-01-04 |

            1 |
```

**Analysis:**

The daily active users vary. The highest number of active users was on the 1 and 3 january in 2022.

### Question 4: Find the total active users each week.

```sql
SELECT
    YEAR(event_date) AS year,
    WEEK(event_date) AS week_number,
    MIN(DATE(event_date)) AS week_start_date,
    COUNT(DISTINCT user_id) AS weekly_active_users
FROM
    activity
GROUP BY
    YEAR(event_date),
    WEEK(event_date)
ORDER by
    YEAR(event_date),
    WEEK(event_date);
```

**Expected Output:**

```plaintext
| year | week | week_start_date | weekly_active_users |
|------|------|-----------------|---------------------|
| 2022 |    0 |      2022-01-01 |                   3 |
| 2022 |    1 |      2022-01-02 |                   5 |
```

**Analysis:**

There were 5 active users in the first week of 2022 and 2 active users in the second week. This can help to understand weekly trends in user activity.

### Question 5: For each date, find the total number of users who made the purchase and installed the app on the same date.

```sql
WITH cte AS (
    SELECT
        event_date,
        user_id
    FROM
        activity
    GROUP BY
        event_date,
        user_id
    HAVING
        COUNT(DISTINCT event_name) = 2
)
SELECT
    event_date,
    COUNT(*) AS users_installed_purchased_same_day
FROM
    cte
GROUP BY
    event_date;
```

**Expected Output:**

```plaintext
| event_date | users_installed_purchased_same_day |
|------------|------------------------------------|
| 2022-01-03 |                                  2 |
| 2022-01-04 |                                  1 |
```

**Analysis:**

On the 3rd of January, 2 users both installed the app and made a purchase. On the 4th of January, 1 user did the same. 

### Question 6: What percentage of users made a purchase in India, USA, and other countries?

```sql
WITH country_purchases AS (
    SELECT
        CASE WHEN country IN ('India', 'USA') THEN country ELSE 'Others' END AS country_group,
        COUNT(DISTINCT user_id) AS purchase_count
    FROM
        activity
    WHERE
        event_name = 'app-purchase'
    GROUP BY
        country_group
),
total_purchases AS (
    SELECT
        SUM(purchase_count) AS total
    FROM
        country_purchases
)
SELECT
    country_group,
    (purchase_count / total) * 100 AS percentage
FROM
    country_purchases,
    total_purchases;
```

**Expected Output:**

```plaintext
| country_group | percentage |
|---------------|------------|
| India         |      40.0  |
| Others        |      40.0  |
| USA           |      20.0  |
```

**Analysis:**

40% of purchases were made in India, another 40% in other countries, and 20% in the USA. This can help in understanding market penetration in different regions.

### Question 7: Among all the users who installed the app on a given day, how many made an in-app purchase the next day? Show the day-wise result.

```sql
WITH cte_user_info AS (
    SELECT
        cte1_installed.user_id AS user_id,
        cte1_installed.event_date AS installed_date,
        cte1_purchased.event_date AS purchased_date
    FROM
        activity cte1_installed
    JOIN
        activity cte1_purchased
    ON
        cte1_installed.user_id = cte1_purchased.user_id
    WHERE
        cte1_installed.event_name = 'app-installed'
        AND cte1_purchased.event_name = 'app-purchase'
        AND DATEDIFF(cte1_purchased.event_date, cte1_installed.event_date) = 1
),
cte_to_order AS (
    SELECT
        purchased_date AS event_date,
        COUNT(DISTINCT user_id) AS installed_this_date_purchased_next
    FROM
        cte_user_info
    GROUP BY
        purchased_date
    UNION
    SELECT
        DISTINCT event_date,
        0
    FROM
        activity
    WHERE
        event_date NOT IN (SELECT purchased_date FROM cte_user_info)
)
SELECT
    *
FROM
    cte_to_order
ORDER BY
    event_date;
```

**Expected Output:**

```plaintext
| event_date | installed_this_date_purchased_next |
|------------|------------------------------------|
| 2022-01-01 |                                  0 |
| 2022-01-02 |                                  1 |
| 2022-01-03 |                                  0 |
| 2022-01-04 |                                  0 |
```

**Analysis:**

On January 2nd , one user made an in-app purchase the day after installing the app.

### Question 8: Calculate the average number of days it took for users to make a purchase after installing the app.

```sql
WITH purchase_dates AS (
    SELECT
        user_id,
        MIN(CASE WHEN event_name = 'app-installed' THEN event_date END) as install_date,
        MIN(CASE WHEN event_name = 'app-purchase' THEN event_date END) as purchase_date
    FROM
        activity
    WHERE
        event_name IN ('app-installed', 'app-purchase')
    GROUP BY
        user_id
)
SELECT
    AVG(DATEDIFF(purchase_date, install_date)) AS avg_days_to_purchase
FROM
    purchase_dates
WHERE
    purchase_date IS NOT NULL;
```

**Expected Output:**

```plaintext
| avg_days_to_purchase |
|----------------------|
|                  0.6 |
```

**Analysis:**

On average, it took users approximately 0.6 days to make a purchase after installing the app.

### Question 9: Rank countries by the number of app installations and purchases.

```sql
WITH country_ranks AS (
    SELECT
        country,
        SUM(CASE WHEN event_name = 'app-installed' THEN 1 ELSE 0 END) as installations,
        SUM(CASE WHEN event_name = 'app-purchase' THEN 1 ELSE 0 END) as purchases
    FROM
        activity
    GROUP BY
        country
)
SELECT
    country,
    installations,
    purchases,
    DENSE_RANK() OVER (ORDER BY installations DESC) as installation_rank,
    DENSE_RANK() OVER (ORDER BY purchases DESC) as purchase_rank
FROM
    country_ranks
ORDER BY
    installations DESC,
    purchases DESC;
```

**Expected Output:**

```plaintext
| country  | installations | purchases | installation_rank | purchase_rank |
|----------|---------------|-----------|-------------------|---------------|
| India    |             2 |         2 |                 1 |             1 |
| USA      |             2 |         1 |                 1 |             2 |
| SL       |             1 |         1 |                 2 |             2 |
| Pakistan |             1 |         1 |                 2 |             2 |
```

**Analysis:**

India and the USA are leading in terms of app installations with India also being the top country for in-app purchases. Other countries have comparatively fewer installations and purchases. 

## Conclusion

Through this case study, we have used MySQL queries to analyze user activity data for Spotify. By examining installations, purchases, and conversion rates, Spotify can make data-driven decisions to enhance user experience and optimize marketing strategies. Additionally, the insights regarding active users and their behavior can be instrumental in identifying trends and patterns in user engagement.


```python

```
