# Power BI Setup Guide - Copilot Credits Dashboard

## Prerequisites

- Power BI Desktop (latest version)
- Admin or Maker access to your Power Platform environment(s)
- Environment URL(s) from Power Platform Admin Center

## Get Your Environment URL(s)

1. Navigate to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/)
2. Select **Environments** > Select your environment > **Details**
3. Copy the **Environment URL** (e.g., `https://org12345.crm.dynamics.com/`)
4. Repeat for additional environments if needed

---

## Option A: Quick Start with Template (Recommended)

The easiest way to get started is using the included Power BI template file.

### Step 1: Open the Template

1. Download `Copilot AI Credits v1.pbit` from this repository
2. Double-click to open in Power BI Desktop

### Step 2: Enter Your Environment URLs

When prompted, enter your environment URL(s):
- **Single environment**: `https://yourorg.crm.dynamics.com/`
- **Multiple environments**: `https://org1.crm.dynamics.com/,https://org2.crm.dynamics.com/`

> **Important**: Include `https://` at the beginning and `/` at the end of each URL. Separate multiple URLs with commas (no spaces).

### Step 3: Authenticate

1. Sign in with your organizational credentials when prompted
2. Wait for data to load (may take a few minutes depending on data volume)

### Step 4: Save Your Report

1. Go to **File** → **Save As**
2. Save as a `.pbix` file for your ongoing use

**That's it!** Your dashboard is ready. The template includes all queries, relationships, calculated columns, measures, and visuals.

---

## Option B: Manual Setup (Build from Scratch)

If you prefer to build the report yourself or want to understand how it works, follow these detailed steps.

### Step 1: Initial Connection to Dataverse

> **Important**: Use the **Common Data Service (Legacy)** connector for the initial connection.

1. Open **Power BI Desktop**
2. Click **Get Data** > **More...**
3. Search for **Common Data Service (Legacy)** and select it
4. Enter your first Environment URL:
   - Include `https://` at the beginning
   - Include `/` at the end
   - Example: `https://yourorg.crm.dynamics.com/`
5. Sign in with your organizational credentials
6. Select these tables:
   - `msdyn_aievent` - AI consumption events
   - `msdyn_aimodel` - AI model definitions
   - `msdyn_aitemplate` - AI templates (prebuilt/custom types)
   - `msdyn_aiconfiguration` - Model configurations
   - `systemuser` - User details for creator/owner mapping
7. Click **Transform Data** to open Power Query Editor

### Step 2: Create Multi-Environment Parameter

This allows pulling data from one or more environments.

1. In Power Query Editor: **Home** → **Manage Parameters** → **New Parameter**
2. Configure:
   - **Name**: `EnvironmentURLs`
   - **Description**: `Comma-separated list of Dataverse environment URLs`
   - **Required**: Yes
   - **Type**: Text
   - **Current Value**: Your URL(s), e.g., `https://org1.crm.dynamics.com/,https://org2.crm.dynamics.com/`
3. Click **OK**

### Step 3: Create Environment List Query

1. Right-click in Queries pane → **New Query** → **Blank Query**
2. Name it `EnvironmentList`
3. Open **Advanced Editor** and paste:

```powerquery
let
    // Split the parameter by comma and trim whitespace
    URLList = Text.Split(EnvironmentURLs, ","),
    TrimmedList = List.Transform(URLList, each Text.Trim(_)),
    // Remove empty entries
    FilteredList = List.Select(TrimmedList, each _ <> ""),
    // Convert to table
    ToTable = Table.FromList(FilteredList, Splitter.SplitByNothing(), {"EnvironmentURL"}),
    // Add a friendly name column (extract org name from URL)
    AddEnvName = Table.AddColumn(ToTable, "EnvironmentName", each 
        let 
            url = [EnvironmentURL],
            afterHttps = Text.AfterDelimiter(url, "://"),
            orgName = Text.BeforeDelimiter(afterHttps, ".")
        in 
            orgName
    )
in
    AddEnvName
```

### Step 4: Create Data Retrieval Functions

Create these functions to fetch data from each environment:

### fnGetAIEvents
Right-click → **New Query** → **Blank Query** → Name: `fnGetAIEvents` → **Advanced Editor**:

```powerquery
(EnvironmentURL as text, EnvironmentName as text) as table =>
let
    FullURL = if Text.StartsWith(EnvironmentURL, "https://") then EnvironmentURL else "https://" & EnvironmentURL,
    CleanURL = if Text.EndsWith(FullURL, "/") then FullURL else FullURL & "/",
    Source = Cds.Entities(CleanURL, [ReorderColumns=null, UseFormattedValue=null]),
    entities = Source{[Group="entities"]}[Data],
    AIEventTable = entities{[EntitySetName="msdyn_aievents"]}[Data],
    AddEnvironment = Table.AddColumn(AIEventTable, "Environment", each EnvironmentName)
in
    AddEnvironment
```

### fnGetAIModels
```powerquery
(EnvironmentURL as text, EnvironmentName as text) as table =>
let
    FullURL = if Text.StartsWith(EnvironmentURL, "https://") then EnvironmentURL else "https://" & EnvironmentURL,
    CleanURL = if Text.EndsWith(FullURL, "/") then FullURL else FullURL & "/",
    Source = Cds.Entities(CleanURL, [ReorderColumns=null, UseFormattedValue=null]),
    entities = Source{[Group="entities"]}[Data],
    AIModelTable = entities{[EntitySetName="msdyn_aimodels"]}[Data],
    AddEnvironment = Table.AddColumn(AIModelTable, "Environment", each EnvironmentName)
in
    AddEnvironment
```

### fnGetAITemplates
```powerquery
(EnvironmentURL as text, EnvironmentName as text) as table =>
let
    FullURL = if Text.StartsWith(EnvironmentURL, "https://") then EnvironmentURL else "https://" & EnvironmentURL,
    CleanURL = if Text.EndsWith(FullURL, "/") then FullURL else FullURL & "/",
    Source = Cds.Entities(CleanURL, [ReorderColumns=null, UseFormattedValue=null]),
    entities = Source{[Group="entities"]}[Data],
    AITemplateTable = entities{[EntitySetName="msdyn_aitemplates"]}[Data],
    AddEnvironment = Table.AddColumn(AITemplateTable, "Environment", each EnvironmentName)
in
    AddEnvironment
```

### fnGetAIConfiguration
```powerquery
(EnvironmentURL as text, EnvironmentName as text) as table =>
let
    FullURL = if Text.StartsWith(EnvironmentURL, "https://") then EnvironmentURL else "https://" & EnvironmentURL,
    CleanURL = if Text.EndsWith(FullURL, "/") then FullURL else FullURL & "/",
    Source = Cds.Entities(CleanURL, [ReorderColumns=null, UseFormattedValue=null]),
    entities = Source{[Group="entities"]}[Data],
    AIConfigTable = entities{[EntitySetName="msdyn_aiconfigurations"]}[Data],
    AddEnvironment = Table.AddColumn(AIConfigTable, "Environment", each EnvironmentName)
in
    AddEnvironment
```

### fnGetSystemUsers
```powerquery
(EnvironmentURL as text, EnvironmentName as text) as table =>
let
    FullURL = if Text.StartsWith(EnvironmentURL, "https://") then EnvironmentURL else "https://" & EnvironmentURL,
    CleanURL = if Text.EndsWith(FullURL, "/") then FullURL else FullURL & "/",
    Source = Cds.Entities(CleanURL, [ReorderColumns=null, UseFormattedValue=null]),
    entities = Source{[Group="entities"]}[Data],
    UserTable = entities{[EntitySetName="systemusers"]}[Data],
    AddEnvironment = Table.AddColumn(UserTable, "Environment", each EnvironmentName)
in
    AddEnvironment
```

### Step 5: Create Combined Queries

Create a combined query for each table that aggregates data from all environments.

### AIEvents_Combined
```powerquery
let
    EnvList = EnvironmentList,
    GetAllData = Table.AddColumn(EnvList, "Data", each 
        try fnGetAIEvents([EnvironmentURL], [EnvironmentName]) otherwise null
    ),
    FilterValid = Table.SelectRows(GetAllData, each [Data] <> null and Table.RowCount([Data]) > 0),
    Result = if Table.RowCount(FilterValid) = 0 
        then #table({"EnvironmentURL", "EnvironmentName", "NoData"}, {})
        else 
            let
                SampleColumns = Table.ColumnNames(FilterValid{0}[Data]),
                ExpandData = Table.ExpandTableColumn(FilterValid, "Data", SampleColumns)
            in
                ExpandData
in
    Result
```

### AIModels_Combined
```powerquery
let
    EnvList = EnvironmentList,
    GetAllData = Table.AddColumn(EnvList, "Data", each 
        try fnGetAIModels([EnvironmentURL], [EnvironmentName]) otherwise null
    ),
    FilterValid = Table.SelectRows(GetAllData, each [Data] <> null and Table.RowCount([Data]) > 0),
    Result = if Table.RowCount(FilterValid) = 0 
        then #table({"EnvironmentURL", "EnvironmentName", "NoData"}, {})
        else 
            let
                SampleColumns = Table.ColumnNames(FilterValid{0}[Data]),
                ExpandData = Table.ExpandTableColumn(FilterValid, "Data", SampleColumns)
            in
                ExpandData
in
    Result
```

### AITemplates_Combined
```powerquery
let
    EnvList = EnvironmentList,
    GetAllData = Table.AddColumn(EnvList, "Data", each 
        try fnGetAITemplates([EnvironmentURL], [EnvironmentName]) otherwise null
    ),
    FilterValid = Table.SelectRows(GetAllData, each [Data] <> null and Table.RowCount([Data]) > 0),
    Result = if Table.RowCount(FilterValid) = 0 
        then #table({"EnvironmentURL", "EnvironmentName", "NoData"}, {})
        else 
            let
                SampleColumns = Table.ColumnNames(FilterValid{0}[Data]),
                ExpandData = Table.ExpandTableColumn(FilterValid, "Data", SampleColumns)
            in
                ExpandData
in
    Result
```

### AIConfiguration_Combined
```powerquery
let
    EnvList = EnvironmentList,
    GetAllData = Table.AddColumn(EnvList, "Data", each 
        try fnGetAIConfiguration([EnvironmentURL], [EnvironmentName]) otherwise null
    ),
    FilterValid = Table.SelectRows(GetAllData, each [Data] <> null and Table.RowCount([Data]) > 0),
    Result = if Table.RowCount(FilterValid) = 0 
        then #table({"EnvironmentURL", "EnvironmentName", "NoData"}, {})
        else 
            let
                SampleColumns = Table.ColumnNames(FilterValid{0}[Data]),
                ExpandData = Table.ExpandTableColumn(FilterValid, "Data", SampleColumns)
            in
                ExpandData
in
    Result
```

### SystemUsers_Combined
```powerquery
let
    EnvList = EnvironmentList,
    GetAllData = Table.AddColumn(EnvList, "Data", each 
        try fnGetSystemUsers([EnvironmentURL], [EnvironmentName]) otherwise null
    ),
    FilterValid = Table.SelectRows(GetAllData, each [Data] <> null and Table.RowCount([Data]) > 0),
    Result = if Table.RowCount(FilterValid) = 0 
        then #table({"EnvironmentURL", "EnvironmentName", "NoData"}, {})
        else 
            let
                SampleColumns = Table.ColumnNames(FilterValid{0}[Data]),
                ExpandData = Table.ExpandTableColumn(FilterValid, "Data", SampleColumns)
            in
                ExpandData
in
    Result
```

### Step 6: Disable Original Queries (Optional)

If you want to use only the combined queries:
1. Right-click each original table (msdyn_AIEvent, msdyn_AIModel, etc.)
2. Select **Enable Load** to uncheck it (disables loading to model)
3. Keep them for reference or delete if not needed

### Step 7: Close & Apply

Click **Close & Apply** to load data into the model.

### Step 8: Create Composite Keys (Required for Multi-Environment)

When pulling data from multiple environments, the same model/template IDs may exist in different environments. Create composite keys to ensure unique relationships.

### In Power Query Editor:

**AIEvents_Combined** - Add Custom Column:
- Name: `ModelKey`
- Formula: `[msdyn_aimodelid] & "|" & [EnvironmentName]`

**AIModels_Combined** - Add Two Custom Columns:
- Name: `ModelKey`
- Formula: `[msdyn_aimodelid] & "|" & [EnvironmentName]`

- Name: `TemplateKey`  
- Formula: `[msdyn_templateid] & "|" & [EnvironmentName]`

**AITemplates_Combined** - Add Custom Column:
- Name: `TemplateKey`
- Formula: `[msdyn_aitemplateid] & "|" & [EnvironmentName]`

Click **Close & Apply** after adding all columns.

### Step 9: Configure Relationships

In Power BI Desktop **Model** view, establish these relationships using composite keys:

### Required Relationships

1. **AIEvents_Combined → AIModels_Combined**
   - From: `AIEvents_Combined[ModelKey]`
   - To: `AIModels_Combined[ModelKey]`
   - Cardinality: Many-to-One (*:1)
   - Cross-filter: Single

2. **AIModels_Combined → AITemplates_Combined**
   - From: `AIModels_Combined[TemplateKey]`
   - To: `AITemplates_Combined[TemplateKey]`
   - Cardinality: Many-to-One (*:1)
   - Cross-filter: Single

3. **DateTable → AIEvents_Combined**
   - From: `DateTable[Date]`
   - To: `AIEvents_Combined[ProcessingDate]`
   - Cardinality: One-to-Many (1:*)
   - Cross-filter: Single

### Step 10: Create Date Table

In Power BI Desktop (not Power Query), **Modeling** tab → **New table**:

```dax
DateTable = 
VAR MinDate = MINX(AIEvents_Combined, DATEVALUE(LEFT([msdyn_processingdate], FIND(" ", [msdyn_processingdate]) - 1)))
VAR MaxDate = MAXX(AIEvents_Combined, DATEVALUE(LEFT([msdyn_processingdate], FIND(" ", [msdyn_processingdate]) - 1)))
RETURN
ADDCOLUMNS(
    CALENDAR(MinDate, MaxDate),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "MonthName", FORMAT([Date], "MMMM"),
    "Quarter", QUARTER([Date]),
    "YearMonth", FORMAT([Date], "YYYY-MM"),
    "WeekNum", WEEKNUM([Date]),
    "DayOfWeek", WEEKDAY([Date]),
    "DayName", FORMAT([Date], "dddd")
)
```

### Step 11: Add Calculated Columns to AIEvents_Combined

Select **AIEvents_Combined** in Data pane, then **Modeling** → **New column**:

### Processing Date (for DateTable relationship)
```dax
ProcessingDate = DATEVALUE(LEFT([msdyn_processingdate], FIND(" ", [msdyn_processingdate]) - 1))
```

### Copilot Credit Consumption

```dax
CopilotCreditConsumption = 
VAR jsonText = AIEvents_Combined[msdyn_eventdata]
VAR searchPattern = """consumption"":"
VAR startPos = FIND(searchPattern, jsonText, 1, -1)
RETURN
    IF(
        startPos > 0,
        VAR afterPattern = MID(jsonText, startPos + LEN(searchPattern), 20)
        VAR endPos = FIND("}", afterPattern, 1, LEN(afterPattern))
        VAR numText = LEFT(afterPattern, endPos - 1)
        RETURN VALUE(numText),
        0
    )
```

### Step 12: Verify Data Access

After loading data, verify in **Table view**:

1. **Row count** in `AIEvents_Combined` - Should show data from all environments
2. **Environment column** - Should show different environment names
3. **Consumption source values** - Should include values 0-3 (PowerAutomation, PowerApps, API, MCS)
4. **CopilotCreditConsumption** - Should extract values from JSON (may be 0 if using free/test calls)

## Creating a Template (.pbit)

To share as a template:

1. **File** → **Export** → **Power BI template**
2. Add description with setup instructions
3. Save as `.pbit` file

When users open the template:
- They'll be prompted for the `EnvironmentURLs` parameter
- They enter their URL(s) and the data loads automatically

## Troubleshooting

### Tables not visible
- Verify admin/maker permissions on the environment
- Check if AI Builder activity monitoring is enabled

### No data returned
- Verify the environment URL format (include `https://` and trailing `/`)
- Check that AI features have been used in that environment

### Function errors
- Use `Cds.Entities` for the data source (not `CommonDataService.Database`)
- Ensure EntitySetName uses plural form (e.g., `msdyn_aievents` not `msdyn_aievent`)

### Copilot Credits showing 0
- Copilot Credit tracking was added October 2025
- Test/quick test events may show 0 consumption
- Preview models may not consume credits

## Next Steps

- See [DAX_CALCULATIONS.md](DAX_CALCULATIONS.md) for all measures and calculations
- See [VISUALS_GUIDE.md](VISUALS_GUIDE.md) for recommended visualizations
- See [DATA_MODEL.md](DATA_MODEL.md) for entity relationships
