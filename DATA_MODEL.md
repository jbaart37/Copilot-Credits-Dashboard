# Data Model Reference

## Entity Relationship Diagram

```
┌─────────────────────────────┐
│     msdyn_aitemplate        │
│─────────────────────────────│
│ msdyn_aitemplateid (PK)     │
│ msdyn_uniquename            │◄──────────────────┐
│ msdyn_name                  │                   │
│ msdyn_templateversion       │                   │
└─────────────────────────────┘                   │
                                                  │ Many-to-One
┌─────────────────────────────┐                   │
│       msdyn_aimodel         │                   │
│─────────────────────────────│                   │
│ msdyn_aimodelid (PK)        │◄──────────┐       │
│ msdyn_templateid (FK)       │───────────┼───────┘
│ msdyn_name                  │           │
│ createdby (FK)              │───────────┼───────────────┐
│ createdon                   │           │               │
│ msdyn_modeltype             │           │               │
└─────────────────────────────┘           │               │
                                          │               │
                               Many-to-One│               │ Many-to-One
                                          │               │
┌─────────────────────────────┐           │               │
│       msdyn_aievent         │           │               │
│─────────────────────────────│           │               │
│ msdyn_aieventid (PK)        │           │               │
│ msdyn_aimodelid (FK)        │───────────┘               │
│ msdyn_creditconsumed        │                           │
│ msdyn_eventdata (JSON)      │                           │
│ msdyn_processingdate        │                           │
│ msdyn_consumptionsource     │                           │
│ msdyn_processingstatus      │                           │
│ msdyn_automationname        │                           │
│ _createdby_value            │                           │
└─────────────────────────────┘                           │
                                                          │
┌─────────────────────────────┐                           │
│        systemuser           │                           │
│─────────────────────────────│                           │
│ systemuserid (PK)           │◄──────────────────────────┘
│ fullname                    │
│ domainname                  │
│ internalemailaddress        │
│ businessunitid              │
└─────────────────────────────┘

┌─────────────────────────────┐
│        DateTable            │
│─────────────────────────────│
│ Date (PK)                   │◄──── Linked to msdyn_aievent[msdyn_processingdate]
│ Year                        │
│ Month                       │
│ MonthName                   │
│ Quarter                     │
│ YearMonth                   │
│ WeekNum                     │
│ DayOfWeek                   │
│ DayName                     │
└─────────────────────────────┘
```

## Table Details

### msdyn_aievent (AI Event)

The primary fact table containing all AI consumption events.

| Column | Type | Description |
|--------|------|-------------|
| `msdyn_aieventid` | GUID | Primary key |
| `msdyn_aimodelid` | GUID | Foreign key to msdyn_aimodel |
| `msdyn_creditconsumed` | Integer | Legacy AI Builder credits |
| `msdyn_eventdata` | Text (JSON) | Contains Copilot Credit data |
| `msdyn_processingdate` | DateTime | When the event occurred |
| `msdyn_consumptionsource` | OptionSet | 0=PowerAutomate, 1=PowerApps, 2=API, 3=MCS |
| `msdyn_processingstatus` | OptionSet | 0=Success, 1=Failed |
| `msdyn_datatype` | String | Type of data processed |
| `msdyn_datainfo` | String | Details about processed data |
| `msdyn_automationname` | String | Name of triggering automation |
| `msdyn_automationlink` | URL | Link to the automation |

### msdyn_aimodel (AI Model)

Dimension table containing AI model definitions.

| Column | Type | Description |
|--------|------|-------------|
| `msdyn_aimodelid` | GUID | Primary key |
| `msdyn_name` | String | Model name |
| `msdyn_templateid` | GUID | Foreign key to msdyn_aitemplate |
| `createdby` | GUID | Foreign key to systemuser |
| `createdon` | DateTime | Creation date |
| `msdyn_modeltype` | OptionSet | Model type |

### msdyn_aitemplate (AI Template)

Dimension table containing AI template types.

| Column | Type | Description |
|--------|------|-------------|
| `msdyn_aitemplateid` | GUID | Primary key |
| `msdyn_uniquename` | String | Template unique name (e.g., "DocumentProcessing") |
| `msdyn_name` | String | Display name |
| `msdyn_templateversion` | String | Template version |

### Template Unique Names (msdyn_uniquename values)

| Unique Name | Description |
|-------------|-------------|
| `TextRecognition` | OCR / Text recognition |
| `DocumentProcessing` | Document automation (Invoice, Receipt, etc.) |
| `ObjectDetection` | Object detection in images |
| `TextCategory` | Category classification |
| `TextEntityExtraction` | Entity extraction |
| `Prediction` | Prediction models |
| `ImageDescription` | Image description |
| `TextGeneration` | GPT/Text generation prompts |
| `SentimentAnalysis` | Sentiment analysis |
| `KeyPhraseExtraction` | Key phrase extraction |
| `LanguageDetection` | Language detection |

## Additional Cost-Related Tables to Consider

### For Extended Cost Analysis

You can supplement the Dataverse data with additional data sources:

1. **Power Platform Admin Center Reports**
   - AI Builder Consumption Report (Excel download)
   - Copilot Studio Credits Report
   - Dataverse Capacity Report

2. **Azure Cost Management** (if using pay-as-you-go)
   - Copilot Studio meters
   - Dataverse storage meters

3. **CoE Starter Kit Tables** (if deployed)
   - Environment inventory
   - Flow inventory
   - App inventory

## Handling Deleted Models

When an `msdyn_aimodelid` in `msdyn_aievent` doesn't match any record in `msdyn_aimodel`, the model was deleted. Create a placeholder:

```dax
Model Name Display = 
VAR ModelName = RELATED(msdyn_aimodel[msdyn_name])
RETURN
    IF(ISBLANK(ModelName), "[Deleted Model]", ModelName)
```
