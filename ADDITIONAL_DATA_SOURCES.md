# Additional Cost Data Sources for Enhanced Dashboard

## Overview

Beyond the core `msdyn_aievent` data, you can integrate additional data sources to create a comprehensive cost management dashboard.

---

## 1. Power Platform Admin Center Reports

### AI Builder Consumption Report

Download from: **PPAC > Resources > Capacity > Add-ons > Download Reports**

| Column | Description |
|--------|-------------|
| Date | Consumption date |
| UserId | Dataverse User ID |
| EnvironmentId | Environment GUID |
| EnvironmentName | Friendly name |
| AIConsumption | Credits consumed |
| IsTrial | Trial vs. production |

**Connection Method**: Import Excel file and refresh manually, or automate with Power Automate flow that downloads and stores in SharePoint/Dataverse.

### Copilot Studio Credits Report

View in: **PPAC > Licensing > Copilot Studio**

Data available:
- Prepaid capacity allocation
- Pay-as-you-go consumption
- Per-environment breakdown
- Per-agent consumption

---

## 2. Dataverse Storage Cost Tables

### Monitor Storage Consumption

Connect to these additional Dataverse tables:

```
Environment Capacity (if accessible via admin API)
- Database storage (GB)
- File storage (GB)
- Log storage (GB)
```

### Cost Calculation Example

```dax
Estimated Storage Cost = 
VAR DatabaseGB = [Database Storage GB]
VAR FileGB = [File Storage GB]
VAR LogGB = [Log Storage GB]
-- Example rates (update with your pricing)
VAR DatabaseRate = 40 -- per GB/month
VAR FileRate = 2 -- per GB/month
VAR LogRate = 10 -- per GB/month
RETURN
    (DatabaseGB * DatabaseRate) + (FileGB * FileRate) + (LogGB * LogRate)
```

---

## 3. Azure Cost Management Integration

If using pay-as-you-go billing, connect to Azure Cost Management:

### Power BI Connector
- Use **Azure Cost Management** connector
- Filter to Power Platform/Dynamics 365 meters
- Relevant meter categories:
  - `Microsoft.PowerPlatform/enterprisePolicies`
  - `Dynamics 365/PowerApps`
  - `Microsoft Copilot Studio`

### Key Meters for Copilot Credits

| Meter Name | Description |
|------------|-------------|
| Copilot Studio Credits | Agent and flow consumption |
| AI Builder Credits | Legacy AI Builder usage |
| Dataverse Database | Database storage |
| Dataverse File | File/attachment storage |
| Dataverse Log | Audit log storage |

---

## 4. Center of Excellence (CoE) Starter Kit

If you've deployed the CoE Starter Kit, leverage these tables:

### Available Tables

| Table | Purpose |
|-------|---------|
| `admin_Environment` | Environment inventory |
| `admin_Flow` | Flow inventory |
| `admin_PowerAppsApp` | App inventory |
| `admin_Maker` | Maker information |
| `admin_CoESettings` | Configuration |

### Enhanced Queries

Join AI Event data with CoE data to:
- Identify which flows are driving consumption
- Group consumption by business unit/maker
- Track consumption trends by app/flow

---

## 5. Custom Tracking Table

Create a custom Dataverse table to store allocation and budget data:

### Schema: Credit Budget Tracking

| Column | Type | Description |
|--------|------|-------------|
| Month | Date | Budget month |
| Environment | Lookup | Environment reference |
| AllocatedCredits | Number | Monthly allocation |
| BudgetAmount | Currency | Dollar budget |
| AlertThreshold | Number | % threshold for alerts |
| Owner | Lookup | Budget owner |

### DAX Integration

```dax
Budget Remaining = 
VAR CurrentMonth = EOMONTH(TODAY(), 0)
VAR Allocated = 
    CALCULATE(
        SUM(CreditBudget[AllocatedCredits]),
        CreditBudget[Month] = CurrentMonth
    )
VAR Consumed = [Credits This Month]
RETURN
    Allocated - Consumed
```

---

## 6. Power Automate API Data

### Flow Run History

Query flow runs to correlate with AI consumption:

```
GET https://api.flow.microsoft.com/providers/Microsoft.ProcessSimple/environments/{environmentId}/flows/{flowId}/runs
```

Useful fields:
- `startTime`, `endTime`
- `status`
- `triggerName`

### Create Correlation Table

Join flow runs with AI events by timestamp to identify which flows generate the most AI consumption.

---

## 7. Microsoft Graph API - License Data

### Get License Information

```
GET https://graph.microsoft.com/v1.0/subscribedSkus
```

Track:
- Total purchased licenses
- Assigned vs. available
- License expiration dates

### Copilot Credit Entitlements

Create a reference table mapping license SKUs to Copilot Credit entitlements:

| License SKU | Credits/Month |
|-------------|---------------|
| Power Automate Premium | Varies |
| Copilot Studio | 25,000 |
| Dynamics 365 Sales Enterprise | Varies |

---

## 8. Combined Cost Dashboard Example

### Cost Summary Table Structure

```dax
-- Create a comprehensive cost table
CostSummary = 
SUMMARIZECOLUMNS(
    DateTable[YearMonth],
    "AI Builder Credits", [Total AI Builder Credits],
    "Copilot Credits", [Total Copilot Credits],
    "Est AI Cost", [Total Copilot Credits] * 0.01, -- Example rate
    "Storage Cost", [Estimated Storage Cost],
    "Total Monthly Cost", [Est AI Cost] + [Estimated Storage Cost]
)
```

---

## 9. Alerting with Power Automate

Create automated alerts when:
- Consumption exceeds 75%/90% of allocation
- Unusual spikes detected (>2x daily average)
- New high-consumption models appear

### Example Flow Trigger

```json
{
  "trigger": "When consumption threshold exceeded",
  "condition": "Credits This Month > (Allocation * 0.75)",
  "action": "Send email to admin team"
}
```

---

## 10. Governance Dashboard Extension

Add a governance page tracking:

1. **Policy Compliance**
   - Models without DLP policy
   - Unattached resources

2. **Security Metrics**
   - Failed authentication attempts
   - API usage from unknown sources

3. **Efficiency Metrics**
   - Credits per successful prediction
   - Error rate by model
   - Unused allocated capacity

---

## Implementation Priority

| Priority | Data Source | Effort | Value |
|----------|-------------|--------|-------|
| 1 | msdyn_aievent (core) | Low | High |
| 2 | PPAC Consumption Report | Low | High |
| 3 | Azure Cost Management | Medium | High |
| 4 | CoE Starter Kit tables | Medium | Medium |
| 5 | Custom budget tracking | Medium | Medium |
| 6 | Graph API licenses | High | Low |

Start with priorities 1-2 for immediate value, then expand as needed.
