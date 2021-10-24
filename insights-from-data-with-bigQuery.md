# Insights from Data with BigQuery: Challenge Lab

Table: `bigquery-public-data.covid19_open_data.covid19_open_data`

## Query 1: Total Confirmed Cases

```sql
SELECT SUM(cumulative_confirmed) AS total_cases_worldwide FROM  `bigquery-public-data.covid19_open_data.covid19_open_data` WHERE date="2020-04-15"
```

## Query 2: Worst Affected Areas

```sql
WITH states AS (
    SELECT SUM(cumulative_deceased) AS deaths, 
    subregion1_name 
FROM 
    bigquery-public-data.covid19_open_data.covid19_open_data 
WHERE country_name="United States of America" AND date="2020-04-10"
    AND subregion1_name is not null
    GROUP BY subregion1_name
    Having deaths > 100
)
SELECT count(*) AS count_of_states FROM states;
```

## Query 3: Identifying Hotspots

```sql
SELECT subregion1_name AS state, SUM(cumulative_confirmed) AS total_confirmed_cases FROM bigquery-public-data.covid19_open_data.covid19_open_data 
WHERE country_code="US" AND date="2020-10-10"
GROUP BY subregion1_name
ORDER BY total_confirmed_cases DESC ;
```

## Query 4: Fatality Ratio

```sql
SELECT 
    SUM(cumulative_confirmed) AS total_confirmed_cases, 
    SUM(cumulative_deceased) AS total_deaths, 
    (SUM(cumulative_deceased) / SUM(cumulative_confirmed))*100 AS case_fatality_ratio 
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE country_name="Italy" AND date BETWEEN "2020-04-01" AND "2020-04-30"
```

## Query 5: Identifying specific day

```sql
SELECT date FROM bigquery-public-data.covid19_open_data.covid19_open_data 
WHERE country_name="Italy" AND cumulative_deceased > 10000 
ORDER BY date LIMIT 1;
```

## Query 6: Finding days with zero net new cases

```sql
WITH india_cases_by_date AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="India"
    AND date between '2020-02-21' and '2020-03-15'
  GROUP BY
    date
  ORDER BY
    date ASC
 )
, india_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases
FROM india_cases_by_date
)
SELECT COUNT(date)
FROM india_previous_day_comparison
WHERE net_new_cases = 0;
```

## Query 7: Doubling rate

```sql
WITH us_cases_by_date AS (
  SELECT
    date,
    SUM( cumulative_confirmed ) AS cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="United States of America"
    AND date between '2020-03-22' and '2020-04-20'
  GROUP BY
    date
  ORDER BY
    date ASC
 )

, us_previous_day_comparison AS
(SELECT
  date,
  cases,
  LAG(cases) OVER(ORDER BY date) AS previous_day,
  cases - LAG(cases) OVER(ORDER BY date) AS net_new_cases,
  (cases - LAG(cases) OVER(ORDER BY date))*100/LAG(cases) OVER(ORDER BY date) AS percentage_increase
FROM us_cases_by_date
)
SELECT
  Date,
  cases AS Confirmed_Cases_On_Day,
  previous_day AS Confirmed_Cases_Previous_Day,
  percentage_increase AS Percentage_Increase_In_Cases
FROM
  us_previous_day_comparison
WHERE
  percentage_increase > 10;
```

## Query 8: Recovery rate

```sql
SELECT country_name AS country, 
SUM(cumulative_recovered) AS recovered_cases, 
SUM(cumulative_confirmed) AS confirmed_cases,
(SUM(cumulative_recovered) / SUM(cumulative_confirmed))*100 AS recovery_rate
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date = "2020-05-10"
GROUP BY country
HAVING confirmed_cases > 50000
ORDER BY recovery_rate DESC
LIMIT 10
```

## Query 9: CDGR - Cumulative Daily Growth Rate

```sql
WITH
  france_cases AS (
  SELECT
    date,
    SUM(cumulative_confirmed) AS total_cases
  FROM
    `bigquery-public-data.covid19_open_data.covid19_open_data`
  WHERE
    country_name="France"
    AND date IN ('2020-01-24',
      '2020-05-10')
  GROUP BY
    date
  ORDER BY
    date)
, summary as (
SELECT
  total_cases AS first_day_cases,
  LEAD(total_cases) OVER(ORDER BY date) AS last_day_cases,
  DATE_DIFF(LEAD(date) OVER(ORDER BY date),date, day) AS days_diff
FROM
  france_cases
LIMIT 1
)

select first_day_cases, last_day_cases, days_diff, POWER(last_day_cases/first_day_cases,1/days_diff)-1 as cdgr
from summary
```

## Create a Datastudio report

```sql
SELECT 
date,
SUM(cumulative_confirmed) as country_cases,
SUM(cumulative_deceased) AS country_deaths
FROM `bigquery-public-data.covid19_open_data.covid19_open_data`
WHERE date BETWEEN "2020-03-15" AND "2020-04-30" AND country_code = "US"
GROUP BY date
```

Go to `Data Studio` -> New Report -> Bigquery -> Authorize
Paste above query in `custom query` field, and add to report.
