# Visuals Guide - Copilot Credits Dashboard

## Dashboard Overview

This dashboard provides visibility into Copilot Credit consumption across multiple Power Platform environments, tracking usage by AI Builder functions, Copilot Studio agents, Power Apps, and Power Automate.

## Recommended Dashboard Layout

### Page 1: Executive Summary

```
┌──────────────┬──────────────┬─────────────────────────────┐
│ Environment ▼│ AI Builder  ▼│  ○────Date Slider─────○     │
├──────────┬───┴──────┬───────┴──────┬───────────────────────┤
│  32.60   │   459    │    100%      │         16            │
│ Total    │  Total   │  Success     │   Unique Models       │
│ Credits  │  Events  │  Rate %      │   Used                │
├──────────┴──────────┼──────────────┴───────────────────────┤
│                     │                                       │
│ Consumption by      │   Top Models by Credits               │
│ Source (Donut)      │   (Horizontal Bar)                    │
│                     │                                       │
├─────────────────────┴───────────────────────────────────────┤
│                                                             │
│           Credit Consumption Trend by Date                  │
│                    (Line Chart)                             │
└─────────────────────────────────────────────────────────────┘
```

### Page 2: Consumption Analysis

```
┌──────────────┬──────────────┬─────────────────────────────┐
│ Environment ▼│ AI Builder  ▼│  ○────Date Slider─────○     │
├─────────────────────────────┬───────────────────────────────┤
│                             │                               │
│   Credits by Template       │   Monthly Credits by Source   │
│   & Month (Matrix)          │   (Stacked Column)            │
│                             │                               │
├───────────────────────┬─────┴───────────────────────────────┤
│                       │                                     │
│ Credits by Template   │   Decomposition Tree                │
│ (Horizontal Bar)      │   (Drill-down analysis)             │
│                       │                                     │
└───────────────────────┴─────────────────────────────────────┘
```

### Page 3: Cost & Capacity

```
┌──────────────┬──────────────┬─────────────────────────────┐
│ Environment ▼│ AI Builder  ▼│  ○────Date Slider─────○     │
├──────────────┴──────────────┼─────────────────────────────┤
│  KPI Cards (Optional)       │   Credits This Month        │
│  - Daily Avg                │   (Gauge)                   │
│  - Projected Month-End      │                             │
├─────────────────────────────┴─────────────────────────────┤
│                                                           │
│   Monthly Credits by Source (Stacked Column)              │
│                                                           │
├───────────────────────────────────────────────────────────┤
│                                                           │
│   Running Total Credits (Area Chart)                      │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## Visual Specifications

### 1. KPI Cards (Summary)

#### Total Copilot Credits Card
- **Visual**: Card
- **Value**: `[Total Copilot Credits]`
- **Conditional Formatting**: 
  - Green: < 75% of allocation
  - Yellow: 75-90% of allocation
  - Red: > 90% of allocation

#### Month-over-Month Change Card
- **Visual**: Card
- **Value**: `[Credits MoM Change %]`
- **Format**: Show as percentage with trend indicator
- **Conditional Formatting**: 
  - Green arrow down if negative (reducing usage)
  - Red arrow up if positive (increasing usage)

### 2. Credit Consumption Trend

- **Visual**: Line Chart with Area
- **X-Axis**: Date (DateTable[Date])
- **Y-Axis**: `[Total Copilot Credits]`
- **Secondary Line**: `[7-Day Moving Avg Credits]` (for smoothed trend)
- **Reference Line**: Monthly allocation threshold

```dax
-- Add reference line for monthly allocation
Monthly Allocation = 10000 -- Update with your value
```

### 3. Consumption by Source (Donut Chart)

- **Visual**: Donut Chart
- **Legend**: `ConsumptionSourceName`
- **Values**: `[Total Copilot Credits]`
- **Colors**:
  - Power Automate: #0066FF
  - Power Apps: #742774
  - API: #00A36C
  - Copilot Studio: #FF6B35

### 4. Top Models by Credits (Bar Chart)

- **Visual**: Horizontal Bar Chart
- **Y-Axis**: `AIModels_Combined[msdyn_name]`
- **X-Axis**: `[Total Copilot Credits]`
- **Top N Filter**: Show Top 10 by `[Total Copilot Credits]`
- **Data Labels**: On

### 5. Credits Matrix (As from Community Guide)

- **Visual**: Matrix
- **Rows**: 
  - Level 1: `AITemplates_Combined[msdyn_uniquename]`
  - Level 2: `AIModels_Combined[msdyn_name]`
- **Columns**: `DateTable[YearMonth]`
- **Values**: `[Total Copilot Credits]`
- **Conditional Formatting**: Data bars

### 6. Budget Gauge

- **Visual**: Gauge
- **Value**: `[Credits This Month]`
- **Minimum**: 0
- **Maximum**: Your monthly allocation
- **Target**: 80% of allocation (warning threshold)
- **Color Coding**:
  - Green: 0-75%
  - Yellow: 75-90%
  - Red: 90-100+%

### 7. Usage Heatmap

- **Visual**: Matrix
- **Rows**: `DateTable[DayName]`
- **Columns**: Hour (extracted from processingdate)
- **Values**: `[Total Events]`
- **Conditional Formatting**: Background color scale
- **Purpose**: Identify peak usage times

---

## Enhanced Cost Visuals

### 8. Cumulative Cost Tracker

- **Visual**: Area Chart
- **X-Axis**: Date
- **Y-Axis**: `[Running Total Credits]`
- **Add Constant Line**: Monthly budget threshold
- **Forecast**: Enable Power BI built-in forecasting (30 days)

### 9. Cost Breakdown Table

- **Visual**: Table
- **Columns**:
  - Model Name
  - Model Type
  - Events Count
  - Credits Consumed
  - Estimated Cost
  - % of Total
- **Sorting**: By Credits Consumed (descending)

### 10. Anomaly Detection

Enable anomaly detection on the trend line:
1. Select line chart
2. Analytics pane > Find Anomalies
3. Sensitivity: Medium
4. This highlights unusual spikes in consumption

---

## Slicers

### Date Range Slicer
- **Visual**: Slicer
- **Field**: `DateTable[Date]`
- **Style**: Between (date range)

### Model Type Slicer
- **Visual**: Slicer
- **Field**: `AITemplates_Combined[msdyn_uniquename]`
- **Style**: Dropdown or List

### Consumption Source Slicer
- **Visual**: Slicer
- **Field**: `ConsumptionSourceName`
- **Style**: Buttons

---

## Conditional Formatting Rules

### Credit Status Background
Apply to cards and key metrics:

```
If [Credit Status] = "Critical" then #FFCCCC (light red)
If [Credit Status] = "Warning" then #FFEECC (light orange)
Else #CCFFCC (light green)
```

### Trend Arrows
For Month-over-Month change:
- Up arrow (▲) with red: Consumption increased
- Down arrow (▼) with green: Consumption decreased
- Horizontal (→) with gray: No significant change

---

## Report Filters

### Page-Level Filters
- Date range: Last 3 months by default
- Exclude test/quick test events: `QuickTest = FALSE` or `NULL`

### Visual-Level Filters
- Exclude blank model names for model analysis
- Filter to successful events only for consumption analysis (optional)

---

## Refresh Schedule

For scheduled refresh in Power BI Service:
- **Recommended**: Daily refresh
- **Time**: Early morning (before business hours)
- **Data latency**: AI Event data typically has 1-2 hour delay
