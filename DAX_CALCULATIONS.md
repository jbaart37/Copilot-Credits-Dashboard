# DAX Calculations for Copilot Credits Dashboard

## Calculated Columns

### 1. Copilot Credit Consumption (Add to AIEvents_Combined)

Extracts Copilot Credit consumption from the JSON `msdyn_eventdata` field:

```dax
CopilotCreditConsumption = 
VAR jsonText = [msdyn_eventdata]
VAR startPos = FIND("""consumption"":", jsonText, 1, -1)
RETURN
    IF(
        startPos > 0,
        VALUE(
            SUBSTITUTE(
                MID(
                    jsonText,
                    startPos + LEN("""consumption"":"),
                    FIND("}", jsonText & "}", startPos) - startPos - LEN("""consumption"":")
                ),
                "}", ""
            )
        ),
        0
    )
```

### 5. Consumption Source Friendly Name (Optional - Add to AIEvents_Combined)

More user-friendly labels (use `msdyn_consumptionsource_display` column exists already):

```dax
ConsumptionSourceFriendly = 
SWITCH(
    [msdyn_consumptionsource_display],
    "MCS", "Copilot Studio Agents",
    "API", "API Calls",
    "PowerApps", "Power Apps",
    "PowerAutomation", "Power Automate",
    [msdyn_consumptionsource_display]
)
```

---

## Measures

### Core Consumption Measures

#### Total Legacy AI Builder Credits
```dax
Total AI Builder Credits = 
SUM(AIEvents_Combined[msdyn_creditconsumed])
```

#### Total Copilot Credits
```dax
Total Copilot Credits = 
SUM(AIEvents_Combined[CopilotCreditConsumption])
```

#### Combined Total Credits
```dax
Total Credits Consumed = 
[Total AI Builder Credits] + [Total Copilot Credits]
```

#### Event Count
```dax
Total Events = 
COUNTROWS(AIEvents_Combined)
```

#### Success Rate
```dax
Success Rate % = 
DIVIDE(
    CALCULATE(COUNTROWS(AIEvents_Combined), AIEvents_Combined[msdyn_processingstatus_display] = "Processed"),
    COUNTROWS(AIEvents_Combined),
    0
) * 100
```

---

### Time Intelligence Measures

#### Credits This Month
```dax
Credits This Month = 
CALCULATE(
    [Total Copilot Credits],
    DATESMTD(DateTable[Date])
)
```

#### Credits Last Month
```dax
Credits Last Month = 
CALCULATE(
    [Total Copilot Credits],
    DATEADD(DATESMTD(DateTable[Date]), -1, MONTH)
)
```

#### Credits Month-over-Month Change
```dax
Credits MoM Change % = 
VAR CurrentMonth = [Credits This Month]
VAR PreviousMonth = [Credits Last Month]
RETURN
    DIVIDE(CurrentMonth - PreviousMonth, PreviousMonth, 0) * 100
```

#### Credits YTD
```dax
Credits YTD = 
CALCULATE(
    [Total Copilot Credits],
    DATESYTD(DateTable[Date])
)
```

#### Running Total Credits
```dax
Running Total Credits = 
CALCULATE(
    [Total Copilot Credits],
    FILTER(
        ALL(DateTable[Date]),
        DateTable[Date] <= MAX(DateTable[Date])
    )
)
```

#### Daily Average Credits
```dax
Daily Avg Credits = 
DIVIDE(
    [Total Copilot Credits],
    DISTINCTCOUNT(DateTable[Date]),
    0
)
```

#### 7-Day Moving Average
```dax
7-Day Moving Avg Credits = 
AVERAGEX(
    DATESINPERIOD(DateTable[Date], MAX(DateTable[Date]), -7, DAY),
    [Total Copilot Credits]
)
```

---

### Model & Template Analysis

#### Unique Models Used
```dax
Unique Models Used = 
DISTINCTCOUNT(AIEvents_Combined[msdyn_aimodelid])
```

#### Top Consuming Model
```dax
Top Consuming Model = 
CALCULATE(
    MAX(AIModels_Combined[msdyn_name]),
    TOPN(1, ALL(AIModels_Combined), [Total Copilot Credits], DESC)
)
```

#### Credits by Model Type (Use with AITemplates_Combined[msdyn_uniquename])
```dax
Credits by Model Type = 
CALCULATE(
    [Total Copilot Credits],
    ALLEXCEPT(AITemplates_Combined, AITemplates_Combined[msdyn_uniquename])
)
```

---

### Source Analysis

#### Credits from Power Automate
```dax
Credits Power Automate = 
CALCULATE(
    [Total Copilot Credits],
    AIEvents_Combined[msdyn_consumptionsource] = 0
)
```

#### Credits from Power Apps
```dax
Credits Power Apps = 
CALCULATE(
    [Total Copilot Credits],
    AIEvents_Combined[msdyn_consumptionsource] = 1
)
```

#### Credits from API
```dax
Credits API = 
CALCULATE(
    [Total Copilot Credits],
    AIEvents_Combined[msdyn_consumptionsource] = 2
)
```

#### Credits from Copilot Studio
```dax
Credits Copilot Studio = 
CALCULATE(
    [Total Copilot Credits],
    AIEvents_Combined[msdyn_consumptionsource] = 3
)
```

---

### Cost Estimation Measures

> **Note**: Update the rate below based on your organization's pricing

#### Estimated Monthly Cost (Example: $0.50 per Copilot Credit)
```dax
Estimated Monthly Cost = 
[Credits This Month] * 0.50
```

#### Cost vs Budget
```dax
Cost vs Budget % = 
VAR Budget = 1000 -- Update with your monthly budget
VAR ActualCost = [Estimated Monthly Cost]
RETURN
    DIVIDE(ActualCost, Budget, 0) * 100
```

#### Projected Month-End Credits
```dax
Projected Month-End Credits = 
VAR DaysElapsed = DAY(TODAY())
VAR DaysInMonth = DAY(EOMONTH(TODAY(), 0))
VAR AvgDailyCredits = DIVIDE([Credits This Month], DaysElapsed, 0)
RETURN
    AvgDailyCredits * DaysInMonth
```

---

### User Analysis (requires systemuser table)

#### Unique Users
```dax
Unique Users = 
DISTINCTCOUNT(AIEvents_Combined[_createdby_value])
```

#### Credits per User
```dax
Credits per User = 
DIVIDE(
    [Total Copilot Credits],
    [Unique Users],
    0
)
```

---

### Conditional Formatting Values

#### Credit Status
```dax
Credit Status = 
VAR MonthlyCredits = [Credits This Month]
VAR Budget = 10000 -- Your monthly allocation
RETURN
    SWITCH(
        TRUE(),
        MonthlyCredits >= Budget * 0.9, "Critical",
        MonthlyCredits >= Budget * 0.75, "Warning",
        "Normal"
    )
```

#### Credit Status Color
```dax
Credit Status Color = 
SWITCH(
    [Credit Status],
    "Critical", "#FF4444",
    "Warning", "#FFA500",
    "#00AA00"
)
```
