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

---

## Query 5: Success Rate by Take Method

**Question:** Which take method has the highest success rate?
```sql
SELECT take_method,
    ROUND(AVG(success_rate), 2) AS avg_success_rate,
    ROUND(AVG(hunters), 0) AS avg_hunters,
    ROUND(SUM(harvest), 0) AS total_harvest
FROM elk_harvest
WHERE take_method != 'All Weapons Combined'
GROUP BY take_method
ORDER BY avg_success_rate DESC;
```

**Note:** The `!=` operator excludes `All Weapons Combined` so we are comparing only the individual weapon types: Archery, Any Weapon, and Muzzleloader.

**Result:**

| Take Method | Avg Success Rate | Avg Hunters | Total Harvest | Highlight |
|-------------|-----------------|-------------|---------------|-----------|
| **Muzzleloader** | **20.58%** | 105 | 20,384 | ⬆️ Highest Success Rate |
| Any Weapon | 17.60% | 595 | 198,538 | ⬆️ Highest Total Harvest |
| **Archery** | **14.54%** | 269 | 74,000 | ⬇️ Lowest Success Rate |

**Summary:** Muzzleloader hunters achieve the highest success rate despite using the most technically challenging equipment. This is likely due to two factors: muzzleloader seasons often fall during the elk rut when bulls are more active and visible, and the hunter pool tends to skew toward more experienced individuals. Any Weapon seasons draw the most hunters and the highest total harvest, but individual odds are lower due to competition. Archery, while requiring close-range skill, draws a dedicated community but yields the lowest individual success rate.

---

## Query 6: Six Point Rate by Region

**Question:** Which region units have the highest and lowest average rates of six point or greater bulls harvested?
```sql
SELECT region_unit,
    ROUND(AVG(six_point_plus_rate), 2) AS avg_six_point_rate,
    ROUND(AVG(hunters), 0) AS avg_hunters,
    ROUND(AVG(success_rate), 2) AS avg_success_rate
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY region_unit
ORDER BY avg_six_point_rate DESC;
```

**Why this query:** Including `avg_hunters` and `avg_success_rate` alongside the six point rate gives a fuller picture of each unit in a single query.

**Result:**

| Region Unit | Avg Six Point Rate | Avg Hunters | Avg Success Rate | Highlight |
|-------------|-------------------|-------------|-----------------|-----------|
| **42** | **100%** | 3 | 20.00 | ⬆️ Highest |
| **40** | **100%** | 14 | 10.50 | ⬆️ Highest |
| **54** | **100%** | 121 | 8.99 | ⬆️ Highest |
| 70 | 72.34% | 204 | 9.31 | |
| 73A | 71.08% | 124 | 6.85 | |
| — | — | — | — | |
| 3 | 17.51% | 2672 | 10.62 | |
| 21A | 16.81% | 689 | 25.92 | |
| 10A | 15.32% | 3437 | 18.29 | |
| **5** | **9.8%** | 1666 | 16.29 | ⬇️ Lowest |

**Summary:** This result reframes the earlier finding about BLM-managed units. Regions 40 and 42 had among the lowest total harvests, which initially suggested poor elk habitat — but their 100% six point rates tell a different story. These are almost certainly limited entry trophy units where very few tags are issued. If you do harvest, it is a trophy bull. Idaho elk units fall into three distinct categories:

| Region Type | Example | Strategy |
|-------------|---------|----------|
| High volume hunting | Region 4 | Most elk harvested, but low individual odds |
| Best individual odds | Region 13 | Small exclusive unit, 77.8% success rate |
| Trophy quality bulls | Regions 40, 42 | 100% six point rate, very few tags issued |

---

## Query 7: Six Point Rate Over Time

**Question:** How have six point or greater harvest rates trended over the 24-year period?
```sql
SELECT year,
    ROUND(AVG(six_point_plus_rate), 2) AS avg_six_point_rate,
    ROUND(AVG(success_rate), 2) AS avg_success_rate,
    ROUND(SUM(harvest), 0) AS total_harvest
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY year
ORDER BY year ASC;
```

**Why this query:** Including success rate and total harvest alongside the six point rate shows whether all three metrics move together or independently over time.

**Summary:** Six point rates have generally improved from the low 30s in the early 2000s to the high 30s and low 40s in recent years, suggesting Idaho Fish and Game's herd management is successfully allowing more bulls to reach trophy maturity. A notable dip to 27.68% in 2006 stands out — this coincides with a period of significant wolf pack expansion in Idaho, which may have impacted mature bull survival. The peak years of 2015 and 2019 are worth investigating further for any corresponding management changes or favorable habitat conditions.

---

## Query 8: Days Hunted Efficiency

**Question:** Which region gives you the best return on time invested?
```sql
SELECT region_unit,
    ROUND(AVG(days_hunted), 0) AS avg_days_hunted,
    ROUND(AVG(success_rate), 2) AS avg_success_rate
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY region_unit
ORDER BY avg_days_hunted ASC;
```

**Result:**

| Region Unit | Avg Days Hunted | Avg Success Rate | Highlight |
|-------------|----------------|-----------------|-----------|
| **41** | **9** | **50.0%** | ⬆️ Most Efficient |
| 42 | 21 | 20.0% | |
| 38 | 34 | NULL | |
| 18 | 41 | 16.70% | |
| 40 | 56 | 10.5% | |
| — | — | — | |
| 39 | 16,348 | 14.02% | |
| 6 | 20,187 | 16.67% | |
| 1 | 21,325 | 11.82% | |
| 10A | 26,036 | 18.29% | |
| **4** | **35,550** | **10.43%** | ⬇️ Least Efficient |

**Summary:** Region 41 is the standout — hunters average just 9 days with a 50% success rate, making it by far the most efficient unit in the dataset. Region 4, despite being the top producer by total harvest, requires a combined 35,550 hunter days and returns only 10.43% success. More hunters in a unit means more competition and more days spent searching. Popularity does not equal efficiency — for a hunter focused on maximizing their odds relative to time invested, smaller limited entry units offer a dramatically better return.

---

## Query 9: Spike Rate Trends

**Question:** Are younger bulls (spikes) increasing or decreasing as a share of the harvest over time?
```sql
SELECT year,
    ROUND(AVG(spike_rate), 2) AS avg_spike_rate
FROM elk_harvest
WHERE take_method = 'All Weapons Combined'
GROUP BY year
ORDER BY year ASC;
```

**Summary:** Spike rates have declined steadily from around 32–34% in the early 2000s to 20–22% in recent years. Paired with the rising six point rates from Query 7, this tells a clear and consistent story:

| Metric | Early Years (2001–2005) | Recent Years (2019–2024) |
|--------|------------------------|--------------------------|
| Spike rate | ~32–34% | ~20–22% |
| Six point rate | ~27–31% | ~38–42% |

As spike rates go down, six point rates go up — and 2006 ties it together perfectly, showing the highest spike rate (34.4%) and the lowest six point rate (27.68%) in the same year. That inverse relationship confirms the trend is real and sustained. Fewer young bulls are being harvested, more bulls are surviving long enough to reach trophy maturity, and Idaho Fish and Game's management approach appears to be working over the long term.

---

*For data visualizations of these findings, see [visualizations.md](visualizations.md).*
