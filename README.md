# idaho_elk_harvest_analysis

## Project Overview: 
  A data analysis project exploring 24 years of elk hunting and harvest data from the Idaho Department of Fish and Game. This project analyzes elk harvest data from the Idaho department of Fish and Game (2001–2024) to identify trends in hunting success, regional differences, and potential external impacts such as predator reintroduction and tag demand. The goal is to uncover patterns that could inform wildlife management and hunting strategies.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Key Questions](#key-questions)
- [Tools Used](#tools-used)
- [Summary of Findings](#summary-of-findings)
- [Repository Structure](#repository-structure)

---

This project analyzes elk harvest data from the Idaho Department of Fish and Game spanning from 2001 to 2024. With 24 years of data across multiple hunting units, seasons, and harvest methods, the dataset offered a rich opportunity to identify long-term trends and regional differences in elk hunting success.

The analysis focuses on harvest rates over time, the performance of individual hunting units, shifts in the age of elk being harvested, and how different hunting methods and seasons compare in terms of success. The broader aim is to surface patterns that could inform wildlife management decisions and help hunters make smarter choices about where and how they hunt.

## Key Questions: 

How have elk harvest rates changed over time (2001–2024)?

Which hunting units are hunters most likely to be successful in?

Is there evidence of a shift in harvest toward more mature elk (e.g., spike vs. six-point)?

Which take method or season type yields the highest success rates?

When factoring in both days invested and success rate, which hunting units offer the best return on effort?

## Tools Used: 
- Google Sheets - Initial data handling and light cleaning
- MySQL - Data cleaning, transformation, and analysis
- Tableau - Data visualization and dashboard creation

## Summary of Findings

**Harvest Trends Over Time:** Over 291,642 elk were harvested in Idaho from 2001–2024. Early years were notably low, possibly reflecting the effects of wolf reintroduction into central Idaho in 1995, with wolf populations expanding significantly through the early 2000s and potentially impacting elk behavior and survival during that period. Harvest rates have trended upward over time and have stabilized in the low to mid 13,000s in recent years, suggesting improved herd management by Idaho Fish and Game.

**Top Performing Hunting Units:** Region 4 leads in total harvest with 16,333 elk, followed by Region 10A and Region 6 — all located in northern Idaho where dense National Forest land and high elevation create ideal elk habitat. However high harvest volume does not equal high individual success. Region 13, despite having only about 18 hunters on average, posts a 77.8% success rate — the highest in the state. Access to Region 13 is naturally restricted by private property and very limited road access, meaning less competition and dramatically better individual odds.

**Shifts in Harvest Composition:** Over 24 years spike harvest rates have declined from around 32–34% to 20–22% while six point or greater rates have risen from the high 20s to the high 30s and low 40s. This inverse relationship indicates that more bulls are surviving long enough to reach trophy maturity, hunters are being more selective with their harvests, and that Idaho Fish and Game's herd management practices are working over the long term.

**Most Successful Harvest Methods:** Muzzleloader seasons produce the highest individual success rate at 20.58% despite having the fewest hunters, likely due to favorable season timing during the elk rut and a smaller hunter pool. Any Weapon seasons draw the most participants and the highest total harvest but sit in the middle for individual success, confirming that more hunters means more competition and lower individual odds.

**Best Units by Effort-to-Success Ratio:** Region 41 is the standout, hunters average just 9 days with a 50% success rate, making it the most efficient unit in the dataset. Region 4, despite leading in total harvest, requires a combined 35,550 hunter days and returns only 10.43% individual success. Popularity does not equal efficiency.

## Repository Structure
```
📁 Idaho-Elk-Harvest-Analysis
├── README.md               ← You are here
├── data_cleaning.md        ← Data cleaning process and queries
├── sql_analysis.md         ← Key questions, queries, and answers
├── visualizations.md       ← Tableau dashboard summaries and link
└── works_cited.md          ← Data sources and credits
```

---

**[View Interactive Dashboards on Tableau Public](https://public.tableau.com/app/profile/shannon.joyce5716/vizzes)**

---

*Data sourced from the [Idaho Department of Fish and Game](https://idfg.idaho.gov). See [works_cited.md](works_cited.md) for full attribution.*
