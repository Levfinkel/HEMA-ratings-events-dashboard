# HEMA-ratings-events-dashboard
A description of a dashboard about fencing events, based on the data from HEMA ratings
## HEMA Tournaments — Tableau dashboard write-up

### Project overview

This project turns raw event statistics from **HEMA Ratings** into an exploratory Tableau dashboard. The goal was to answer several questions:

- **How has the HEMA tournament scene changed over time and across regions?**
- **How did covid influence HEMA?**
- **What do “typical” tournaments look like in terms of size, number of fights, and fights per fighter?**

To support both “big picture” and “pick a specific event” exploration, I built two dashboards:

1. **Main dashboard** — trends and composition (time, geography, weapon categories, seasonality)
2. **Fights & Fighters dashboard** — distributions of events by the number of fights, fighters and their ratio to each other.

---

## Data acquisition and preparation

### Source

I received an **Excel export** from **Petter Vegsund Brodin (HEMA Ratings)** containing tournament event statistics. The structure is *event/division level*: one event can have multiple rows (e.g., Mixed Longsword, Women’s Longsword), each row containing:

- tournament identifiers (Event ID, Event Name, date, country/city etc.)
- division/category name
- number of fights
- number of fighters (entries)

The important caveat here, is that for now I only have data from HEMA ratings, which means that any HEMA event that took place, but wasn't registered there, doesn't appear in my data. 
### Key modeling decision

Because the dataset is division-level, many “per tournament” metrics must be **aggregated to Event ID** first (to avoid double counting). In Tableau, I used **FIXED LOD calculations** to create event-level totals, e.g.:

- total fights per tournament = `{ FIXED [Event ID] : SUM([Fights]) }`
- total entries per tournament = `{ FIXED [Event ID] : SUM([Fighters]) }`

Those event-level totals become the basis for the “per tournament” histograms and buckets.

---

## Tableau build process

### 1) Connect and model

- Connected Tableau to the provided Excel file.
- Verified row granularity (division rows per event).
- Built calculated fields to produce **event-level** totals and derived metrics (entries, fights, fights per fighter).

### 2) Create worksheets

I created several worksheets, each focused on one question. These were later assembled into dashboards.

---

## Main dashboard — what each workbook shows

<img width="1890" height="802" alt="image" src="https://github.com/user-attachments/assets/946e28d7-13c7-4636-bd42-b98a98a59ce9" />


### Events by continents (Treemap)

A treemap showing how events are distributed across broad regions (e.g., Europe, Americas, Asia/Oceania). The most interesting insight here is that the number of touranments in Europe is higher, that in Americas, even thought US vastly overperforms any single European country, even the large ones.

Here is how countries are ddistributed among the groups (grouping is subjective):

| Region | Countries |
|---|---|
| Asia and Oceania | Afghanistan, Australia, China, Hong Kong, Indonesia, New Zealand, Singapore, South Korea, Taiwan |
| Americas | Argentina, Brazil, Canada, Chile, Mexico, Peru, United States |
| Europe + | Austria, Belgium, Bulgaria, Croatia, Czech Republic, Denmark, Finland, France, Georgia, Germany, Greece, Hungary, Iceland, Ireland, Israel, Italy, Latvia, Lithuania, Netherlands, Norway, Poland, Portugal, Romania, Serbia, Slovakia, Spain, Sweden, Switzerland, Turkey, Ukraine, United Kingdom |
| Russia + | Belarus, Kazakhstan, Russia |

### Events by year (trend line/area)

A time series of events per year to show growth and changes in activity. This one shows, that HEMA scene recovered from covid quite well, and 2023 held more regestered events, than 2019. Unfortunately, I don't have the data to distinguish the actual recovery of the sport from the rise of poplularity of HEMA ratings.

### Events by season (bars)

Seasonality analysis using a custom “season” calculation based on event date. This shows which part of the year tends to host more events.

### Events by categories of weapons (Treemap)

A treemap of events by weapon category/division group, showing what formats dominate in the dataset.

Important notice: Each division here is considered as a separate event i.e., if a given tournament had both a longsword and a saber competitions, both saber and longsword will have their counts updated.

### Filtering 
The dashboard can be filtered either by the event country (list on the right), or by any of the charts.

---

## Fights & Fighters dashboard — distributions + event locator
<img width="1885" height="796" alt="Fights and fighers dashboard" src="https://github.com/user-attachments/assets/06ca1cb8-dc73-414b-a3c0-b6337222d251" />


This dashboard focuses on “what a typical tournament looks like” using **distributions** (histograms/buckets), and adds a feature to quickly locate a specific tournament inside those distributions. 
**Each entry is considered as a separate fighter**, because I don't have the data about individual fighters.

### Fights per fighter in an event 

A bucketed distribution of “fights per fighter” at event level. Because the raw data is division-level, this metric is derived from event totals. 

### Number of fighters in an event 

A histogram of tournament size based on **event entries summed across divisions** (event-level total fighters). Buckets are custom (e.g., 0–20, 21–40 …) to keep the view readable while still showing tail behavior.

### Number of fights in an event 

A histogram of event fight totals, bucketed (e.g., 0–40, 41–80 …) to show typical tournament workload and the long tail of very large events.

### Country filter

Just like the main dashboard, the country filter is available here to compare distributions by geography.

---

## Red dot functionality — “Where does this tournament land?”

A key piece of interactivity is the **red dot**: after selecting a tournament in a search bar on the right, the dashboard places a dot **on top of the bar** representing the bucket where that tournament belongs.

### Why this was non-trivial

Because the dataset contains multiple rows per tournament (one per division), a naïve “selected tournament” mark can accidentally create multiple dots. Also, if you simply plot the selected tournament value, it often lands at y=1 rather than **on top of the bar height**.

### How it works (conceptually)

1. **Find the selected event** (via a parameter or set selection).
2. Determine **which bucket** that event falls into (based on event-level totals).
3. Compute **the bar height for that bucket** (count of tournaments in that bucket).
4. Plot a dot at *(bucket, bar height)* using a **dual axis** chart.

### Implementation approach (high level)

- Built an event-level selection flag (LOD by Event ID) so selection is stable even with division rows.
- Created a dot measure that returns the **bucket count** only for the selected event.
- Used a **dual axis**: bars show `COUNTD(Event ID)`, the dot layer shows the dot measure.
- Synchronized axes and customized the dot tooltip so it displays only the relevant information.

Result: the user can type/select a tournament, and instantly see whether it is “typical” or an outlier in terms of fighters, fights, and fights-per-fighter.

---

## Outcome

The final dashboards allow:

- macro exploration (year/region/season/category composition),
- distribution analysis (what’s “normal” for tournament size and intensity),
- and a practical “lookup” mode (select an event → see where it sits in the distributions),
    
    with country filtering available throughout.
    
