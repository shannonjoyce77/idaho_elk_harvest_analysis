# SQL Analysis

This document covers the key questions asked of the data, the queries used to answer them, and a summary of what the results reveal. All queries using aggregate harvest data filter to `take_method = 'All Weapons Combined'` to avoid double counting across weapon types.

---

## Table of Contents

- [How many total elk were harvested across all years?](#query-1-total-elk-harvested)
- [What was the total elk harvest by year?](#query-2-harvest-by-year)
- [Which regions had the highest and lowest total harvest?](#query-3-harvest-by-region)
- [What is the average success rate by region unit?](#query-4-average-success-rate-by-region)
- [Which take method has the highest success rate?](#query-5-success-rate-by-take-method)
- [Which region units have the highest and lowest six point or greater rates?](#query-6-six-point-rate-by-region)
- [How have six point rates trended over time?](#query-7-six-point-rate-over-time)
- [Which region gives you the best return on time invested?](#query-8-days-hunted-efficiency)
- [Are younger bulls increasing or decreasing over time?](#query-9-spike-rate-trends)

---

## Query 1: Total Elk Harvested

**Question:** How many total elk were harvested across all years?
```sql
SELECT SUM(harvest) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined';
```

**Result:** 291,642 total elk harvested from 2001–2024.

**Why this query:** Within the `take_method` column the data includes `All Weapons Combined`, `Any Weapon`, `Archery`, and `Muzzleloader`. Using `All Weapons Combined` ensures we are not double counting, as it already totals all accepted harvest methods for each region unit and year.

**Summary:** Over 291,000 elk were harvested in Idaho across the 24-year study period, establishing a strong baseline for all further analysis.

---

## Query 2: Harvest by Year

**Question:** What was the total elk harvest by year?
```sql
SELECT year, SUM(harvest) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY year
ORDER BY year ASC;
```

**Result:**

| Year | Total Harvest |
|------|--------------|
| 2001 | 8,589 |
| 2002 | 7,315 |
| 2003 | 10,846 |
| 2004 | 12,306 |
| 2005 | 16,647 |
| 2006 | 12,870 |
| 2007 | 12,676 |
| 2008 | 10,811 |
| 2009 | 10,475 |
| 2010 | 11,789 |
| 2011 | 9,866 |
| 2012 | 10,241 |
| 2013 | 9,853 |
| 2014 | 12,886 |
| 2015 | 15,048 |
| 2016 | 13,172 |
| 2017 | 13,276 |
| 2018 | 13,473 |
| 2019 | 13,804 |
| 2020 | 15,050 |
| 2021 | 12,775 |
| 2022 | 12,991 |
| 2023 | 11,713 |
| 2024 | 13,170 |

To find the 24-year average annual harvest:
```sql
SELECT ROUND(AVG(annual_total), 0) AS avg_annual_harvest
FROM (
    SELECT year, SUM(harvest) AS annual_total
    FROM elk_harvest
    WHERE take_method = 'All Weapons Combined'
    GROUP BY year
) AS yearly_totals;
```

**Result:** 12,152 elk per year on average.

**Summary:** 2001 and 2002 were notably low harvest years, possibly reflecting the effects of wolf reintroduction on elk populations during that period. 2005 saw a significant spike, and 2020 produced a surprisingly high harvest year despite being a COVID year — likely because more people turned to outdoor activities when other options were shut down. Excluding the early 2000s, recent years consistently exceed the 24-year average, suggesting improved herd management or increased hunting pressure over time.
---

## Query 3: Harvest by Region

**Question:** Which regions had the highest and lowest total harvest?
```sql
SELECT region_unit, SUM(harvest) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY region_unit
ORDER BY total_harvest DESC;
```

**Result:**

| Region Unit | Total Harvest | Highlight |
|-------------|--------------|-----------|
| **4** | **16,333** | ⬆️ Highest |
| 10A | 14,944 | |
| 6 | 13,119 | |
| 39 | 12,441 | |
| 1 | 11,153 | |
| — | — | |
| 47 | 4 | |
| 40 | 4 | |
| 11 | 2 | |
| 41 | 1 | |
| 42 | 1 | |
| **18** | **1** | ⬇️ Lowest |

To filter results by a specific year:
```sql
SELECT region_unit, SUM(harvest) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
AND year = 2001
GROUP BY region_unit
ORDER BY total_harvest DESC;
```

**Summary:** The top performing units — 4, 10A, 6, 39, and 1 — are largely concentrated in northern Idaho, an area known for dense National Forest land, high elevation, and historically strong elk populations. Region 4 covers the Coeur d'Alene National Forest, Region 1 includes the Selkirk Mountains, and Region 10A sits near a large reservoir. The lowest performing units (18, 40, 41, 42) are managed by the Bureau of Land Management (BLM), which tends to be lower elevation and more arid — less ideal elk habitat than National Forest land. Harvest success strongly correlates with habitat type.

---

## Query 4: Average Success Rate by Region

**Question:** Which region units have the highest and lowest average success rates?
```sql
SELECT region_unit, ROUND(AVG(success_rate), 2) AS avg_success_rate
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY region_unit
ORDER BY avg_success_rate DESC;
```

**Result:**

| Region Unit | Avg Success Rate | Highlight |
|-------------|-----------------|-----------|
| **13** | **77.8%** | ⬆️ Highest |
| 41 | 50.0% | |
| 21A | 25.92% | |
| 30 | 25.21% | |
| 14 | 24.64% | |
| — | — | |
| 71 | 8.9% | |
| 62A | 8.58% | |
| 11 | 7.10% | |
| 2 | 6.86% | |
| **73A** | **6.85%** | ⬇️ Lowest |

To see hunter numbers and success rates side by side for the top harvest regions:
```sql
SELECT region_unit,
    ROUND(AVG(hunters), 0) AS avg_hunters,
    ROUND(AVG(success_rate), 2) AS avg_success_rate,
    ROUND(SUM(harvest), 0) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY region_unit
ORDER BY total_harvest DESC
LIMIT 10;
```

**Summary:** Success rate and total harvest tell very different stories. Region 4 leads in total elk harvested but averages around 10% individual success, while Region 13 has a 77.8% success rate with only about 18 hunters on average. The most popular hunting regions are not the most successful ones — hunter competition is a key factor. Region 13 is likely a limited entry unit where tags are restricted by Idaho Fish and Game to protect hunting quality, which would explain both the low hunter numbers and exceptional success rate.
