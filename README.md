# Copilot Credits Usage Dashboard for Power BI

A comprehensive Power BI dashboard for tracking and analyzing Microsoft Copilot Credits consumption across your Power Platform environments, with additional cost-related data from Dataverse.

## Key Features

- **Multi-Environment Support** - Connect to multiple Power Platform environments with a single parameter and aggregate credit consumption across your organization
- **Copilot Credit Tracking** - Monitor consumption from AI Builder, Copilot Studio Agents (MCS), Power Apps, Power Automate, and API calls
- **Interactive Analysis** - Drill down by environment, AI model/function, date range, and consumption source using slicers and decomposition trees
- **Cost & Capacity Forecasting** - Track running totals, monthly trends, and projected consumption with gauge visuals
- **Template-Ready** - Export as .pbit template for easy distribution; users simply enter their environment URLs

## Overview

This dashboard tracks:

1. **Copilot Credits consumption** - AI Builder models, prompts, and Copilot Studio agent usage
2. **Multi-source breakdown** - Credits from API, MCS (Copilot Studio), Power Apps, and Power Automate
3. **Time-based analysis** - Monthly trends, running totals, and consumption forecasting
4. **Model-level details** - Per-model and per-template type credit consumption

## Data Sources

### Primary Dataverse Tables (Combined from Multiple Environments)

| Table Name | System Name | Purpose |
|------------|-------------|---------|
| AI Events Combined | `AIEvents_Combined` | Aggregated AI consumption events from all environments |
| AI Models Combined | `AIModels_Combined` | AI model definitions with environment context |
| AI Templates Combined | `AITemplates_Combined` | AI template types (prebuilt vs custom) |
| AI Configuration Combined | `AIConfiguration_Combined` | AI model configurations |
| System Users Combined | `SystemUsers_Combined` | User details for ownership mapping |

### Key Columns in AIEvents_Combined

| Column | Description |
|--------|-------------|
| `CopilotCreditConsumption` | Calculated column extracting credits from JSON |
| `msdyn_eventdata` | JSON containing detailed consumption data |
| `ProcessingDate` | Converted date for DateTable relationship |
| `msdyn_consumptionsource_display` | Source (API, MCS, PowerApps, PowerAutomation) |
| `Environment` | Source environment name |
| `ModelKey` | Composite key for multi-environment relationships |

## Quick Start

See [SETUP_GUIDE.md](SETUP_GUIDE.md) for detailed Power BI setup instructions.

## Project Structure

```
CP Credits Dash/
├── README.md                 # This file
├── SETUP_GUIDE.md           # Power BI connection and setup guide
├── DAX_CALCULATIONS.md       # DAX formulas and measures
├── DATA_MODEL.md            # Data model relationships
└── VISUALS_GUIDE.md         # Recommended visualizations
```

## Dashboard Pages

1. **Executive Summary** - KPI cards, trend line, consumption by source donut, top models bar chart
2. **Consumption Analysis** - Matrix by template/month, stacked column by source, decomposition tree
3. **Cost & Capacity** - Gauge for budget tracking, running total area chart, monthly breakdown

## Credit Currency Transition

As of November 2026, Microsoft is ending AI Builder credits. This dashboard tracks consumption regardless of currency type—the underlying Dataverse tables and event structure remain unchanged.

| Timeline | What's Happening |
|----------|------------------|
| **Now - Nov 2026** | Dual-mode: AI Builder credits consumed first, then fallback to Copilot Credits |
| **Nov 2026** | Seeded AI Builder credits from premium licenses end |
| **Post Nov 2026** | All consumption uses Copilot Credits; AI Builder add-on credits continue until contract expiration |

**Key Points:**
- No automatic conversion from AI Builder credits to Copilot Credits
- The `msdyn_aievents` table continues logging all AI consumption events
- This dashboard will continue to work — it tracks events and consumption, not the currency type

For full details, see [End of AI Builder credits](https://learn.microsoft.com/en-us/ai-builder/endofaibcredits).

## References

- [Monitor AI Builder models and prompts activity](https://learn.microsoft.com/ai-builder/activity-monitoring)
- [AI Builder consumption report](https://learn.microsoft.com/ai-builder/administer-consumption-report)
- [Manage Copilot Studio credits and capacity](https://learn.microsoft.com/power-platform/admin/manage-copilot-studio-messages-capacity)
- [Dataverse capacity-based storage details](https://learn.microsoft.com/power-platform/admin/capacity-storage)
- [End of AI Builder credits](https://learn.microsoft.com/en-us/ai-builder/endofaibcredits)
