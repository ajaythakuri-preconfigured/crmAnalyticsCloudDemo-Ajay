# 🚀 Tableau Next Build Guide — DTC Sales Dashboard
## Based on Live Org Audit: pr1786449020763.my.salesforce.com
**Org:** pr1786449020763.my.salesforce.com
**User:** ajay.thakuri+partner@preconfigured.ai
**Audited:** 2026-09-28
**Prepared By:** Claude AI

---

## 📊 Live Org Audit Findings

### ✅ What's Ready
| Item | Status | Detail |
|------|--------|--------|
| Salesforce CRM Opportunity | ✅ Live | 120 records, $8,676,000 total |
| Account DLM (Data Cloud) | ✅ Live | 40 records — ssot__Account__dlm |
| User DLM (Data Cloud) | ✅ Live | 14 records — ssot__User__dlm |
| Key Opportunity fields | ✅ Confirmed | Amount, StageName, CloseDate, IsClosed, IsWon, OwnerId, AccountId |
| Key Account fields | ✅ Confirmed | Industry, BillingCountry, BillingState, Type |

### ❌ Critical Gap Found
| Item | Status | Action Required |
|------|--------|----------------|
| **Opportunity DMO in Data Cloud** | ❌ MISSING | Must ingest Opportunity object into Data Cloud before Semantic Model can be built |

> ⚠️ **Important:** The user said "everything is set up" — but Opportunity is NOT yet in Data Cloud as a DMO. Only Account and User DLMs exist. **This must be fixed first (Step 0) before any Tableau Next work.**

---

## 📐 Validated KPI Baseline Numbers
*(These are the exact numbers your Tableau Next dashboard should show — use for validation)*

| KPI | Value | Records |
|-----|-------|---------|
| **Total Pipeline** | $8,676,000 | 120 |
| **Open Pipeline** | $2,892,000 | 40 |
| **Won Revenue** | $2,892,000 | 40 |
| **Lost Revenue** | $2,892,000 | 40 |
| **Win Rate %** | **50.0%** | 40 won / 80 closed |

### Data Breakdown
| Dimension | Breakdown |
|-----------|-----------|
| **Years** | 2025 (21 records), 2026 (99 records) |
| **Countries** | Germany $4,059,000 · France $1,899,000 · UK $1,449,000 · Netherlands $1,269,000 |
| **Industries** | Retail $7,317,000 (105 records) · Transportation $1,359,000 (15 records) |
| **Stages** | Identified · Qualified · Proposal or Trial · Negotiation · Won · Closed Lost |
| **Owners** | Shyamsunder Mule (primary owner) |

### Cumulative Won Revenue (Time Chart Validation)
| Month | Monthly Won | Cumulative |
|-------|------------|------------|
| Nov 2025 | $675,000 | $675,000 |
| Dec 2025 | $465,000 | $1,140,000 |
| Jan 2026 | $735,000 | $1,875,000 |
| Feb 2026 | $918,000 | $2,793,000 |
| Aug 2026 | $99,000 | **$2,892,000** |

---

## 🗺️ STEP-BY-STEP BUILD GUIDE

---

## 🔴 STEP 0 — Fix Data Cloud Gap: Ingest Opportunity (30–60 min)
*Do this FIRST — nothing else works without it.*

### 0.1 Open Data Cloud Setup
```
Salesforce Setup → Search "Data Cloud" → Data Cloud Setup
```

### 0.2 Configure Salesforce CRM Connector (SFDC_LOCAL)
```
Data Cloud → Data Sources → New → Salesforce CRM (SFDC_LOCAL)
Connector name: Salesforce CRM
→ Click Connect
```

### 0.3 Ingest Opportunity Object
```
Data Cloud → Data Streams → New Data Stream
Source: Salesforce CRM (SFDC_LOCAL)
Object: Opportunity

Select these fields:
  ✅ Id
  ✅ Name
  ✅ Amount
  ✅ StageName
  ✅ CloseDate
  ✅ IsClosed
  ✅ IsWon
  ✅ OwnerId
  ✅ AccountId
  ✅ ForecastCategory
  ✅ Type
  ✅ LeadSource
  ✅ CreatedDate
  ✅ FiscalYear
  ✅ FiscalQuarter

Refresh schedule: Daily at 6:00 AM
Category: Sales Order (or Sales)
→ Save & Run
```

### 0.4 Map to DMO
```
After Data Stream is created:
Data Cloud → Data Model → Find "Opportunity" object
Map fields:
  Opportunity.Amount        → SalesOrder.TotalAmount (or keep as custom)
  Opportunity.IsClosed      → custom field
  Opportunity.IsWon         → custom field
  Opportunity.CloseDate     → SalesOrder.OrderedDate
  Opportunity.OwnerId       → SalesOrder.OwnedById
  Opportunity.AccountId     → SalesOrder.SoldToCustomerId
→ Save & Publish
```

### 0.5 Verify Ingestion
```
Data Cloud → Data Explorer → Search "Opportunity"
Confirm: 120 records visible
Confirm fields: Amount, IsClosed, IsWon, CloseDate are populated
```

> ✅ Once you see 120 Opportunity records in Data Explorer — Step 0 is done. Move to Step 1.

---

## 🧠 STEP 1 — Build the Semantic Model (2–3 hrs)

### 1.1 Navigate to Tableau Next
```
App Launcher → Tableau Next
(or: pr1786449020763.my.salesforce.com/analytics)
→ Click "New" → "Semantic Model"
```

### 1.2 Create Semantic Model
```
Name:        DTC Sales Analytics
Description: Sales performance semantic model for pipeline, won, and lost revenue analysis.
             Powers DTC Sales dashboard and Tableau Agent queries.
Data Space:  Default
→ Next
```

### 1.3 Connect Data Source
```
Source: Data Cloud
Object: Opportunity (the DMO you just created in Step 0)
→ Add
```

### 1.4 Define the 5 Metrics

**METRIC 1 — Total Pipeline Amount**
```
Click "+ Add Metric"
Name:          Total Pipeline Amount
Display Label: Total Pipeline
Formula:       SUM(Amount)
Format:        Currency | 0 decimal places | Prefix: $
Description:   Total value of all opportunities including open, won, and lost.
               Use to understand full pipeline size.
→ Save
```
*Expected value: $8,676,000*

---

**METRIC 2 — Open Pipeline Amount**
```
Click "+ Add Metric"
Name:          Open Pipeline Amount
Display Label: Open Opportunities
Formula:       SUM(Amount)
Filter:        IsClosed = false
Format:        Currency | 0 decimal places | Prefix: $
Description:   Sum of amount for opportunities that are still open and actively being worked.
→ Save
```
*Expected value: $2,892,000*

---

**METRIC 3 — Won Revenue**
```
Click "+ Add Metric"
Name:          Won Revenue
Display Label: Won Opportunities
Formula:       SUM(Amount)
Filter:        IsWon = true
Format:        Currency | 0 decimal places | Prefix: $
Description:   Total closed-won revenue. Primary measure of sales success.
               Ask: 'What is our won revenue?' or 'Show won revenue by quarter'
→ Save
```
*Expected value: $2,892,000*

---

**METRIC 4 — Lost Revenue**
```
Click "+ Add Metric"
Name:          Lost Revenue
Display Label: Lost Opportunities
Formula:       SUM(Amount)
Filter:        IsWon = false AND IsClosed = true
Format:        Currency | 0 decimal places | Prefix: $
Description:   Total value of closed-lost opportunities. High values indicate
               conversion issues. Compare against Won Revenue to assess win rate.
→ Save
```
*Expected value: $2,892,000*

---

**METRIC 5 — Win Rate % (Bonus)**
```
Click "+ Add Metric"
Name:          Win Rate Percentage
Display Label: Win Rate %
Formula:       [Won Revenue] / ([Won Revenue] + [Lost Revenue]) * 100
Format:        Percent | 1 decimal place | Suffix: %
Description:   Percentage of closed deals that were won.
               Target: >50%. Current: 50.0%
→ Save
```
*Expected value: 50.0%*

---

### 1.5 Define Dimensions

Add each dimension by clicking "+ Add Dimension":

| Dimension Name | Field | Type | Notes |
|---------------|-------|------|-------|
| Close Date | CloseDate | Date | Enable: Year, Quarter, Month, Day |
| Stage | StageName | Text | Values: Identified, Qualified, Proposal or Trial, Negotiation, Won, Closed Lost |
| Opportunity Type | Type | Text | Picklist |
| Lead Source | LeadSource | Text | Picklist |
| Forecast Category | ForecastCategory | Text | Picklist |
| Fiscal Year | FiscalYear | Number | |
| Fiscal Quarter | FiscalQuarter | Number | |
| Is Closed | IsClosed | Boolean | |
| Is Won | IsWon | Boolean | |

**Join Account fields (for Industry, Country):**
```
In Semantic Model → Relationships → Add Join
Join: Opportunity.AccountId = Account.Id (ssot__Account__dlm)
Then add these dimensions from Account:
  Account Industry    → ssot__PrimaryIndustry__c   (Label: Industry)
  Account Country     → ssot__BillContactAddressId__c (or BillingCountry via lookup)
  Account Name        → ssot__Name__c
  Account Type        → ssot__AccountType__c
```

**Join User fields (for Owner Name):**
```
Join: Opportunity.OwnerId = User.Id (ssot__User__dlm)
Add dimension:
  Owner Name → User.Name
```

### 1.6 Test with Tableau Agent
Type these questions — all must return correct answers:
```
"What is our total pipeline?"           → Should show $8,676,000
"Show me won revenue"                   → Should show $2,892,000
"What is the win rate?"                 → Should show 50%
"Show pipeline by industry"             → Should show Retail + Transportation
"Show won revenue by month"             → Should show Nov 2025 through Aug 2026
```
> ✅ If all 5 answer correctly → Semantic Model is solid. Move to Step 2.

---

## 🎨 STEP 2 — Build the Dashboard (2–3 hrs)

### 2.1 Create Dashboard
```
Tableau Next → New → Dashboard
Name:        DTC Sales
Description: Sales pipeline performance — Total, Open, Won, and Lost opportunities
Workspace:   [Your workspace]
→ Create
```

### 2.2 Add Title + Year Filter (Header Row)
```
ADD TEXT WIDGET:
  Content:     DTC Sales
  Font size:   24px Bold
  Position:    Top-left

ADD FILTER CONTROL:
  Type:         Dimension Filter
  Dimension:    Close Date (Year)
  Style:        Dropdown or Segmented buttons (Year)
  Default:      All Years (or 2026 as default)
  Position:     Top-right
  Apply to:     ALL visualizations on dashboard
```

### 2.3 Build 4 KPI Metric Tiles (Row 2)
Place 4 tiles side by side:

**TILE 1 — Total Pipeline**
```
Add: Metric Tile
Metric:       Total Pipeline Amount
Label:        TOTAL
Color:        Dark blue / neutral
Size:         25% width
Period compare: vs. Prior Year (optional)
```

**TILE 2 — Open Pipeline**
```
Add: Metric Tile
Metric:       Open Pipeline Amount
Label:        Open Opportunities
Color:        Blue (#4A90D9 or similar)
Size:         25% width
```

**TILE 3 — Won Revenue**
```
Add: Metric Tile
Metric:       Won Revenue
Label:        Won Opportunities
Color:        Green (#43B263)
Size:         25% width
```

**TILE 4 — Lost Revenue**
```
Add: Metric Tile
Metric:       Lost Revenue
Label:        Lost Opportunities
Color:        Red (#E74C3C)
Size:         25% width
```

### 2.4 Build Cumulative Time Chart (Row 3) ⚠️ Most Complex
```
Add: Line Chart (or Area Chart)

Configure:
  Metric:     Won Revenue
  Dimension:  Close Date → Month
  Filter:     IsWon = true (built into metric — no extra filter needed)
  
Apply Table Calculation:
  Click the Won Revenue pill on Y-axis
  → Table Calculation → Running Total
  → Compute along: Month (date axis)
  → This creates the cumulative effect

Expected result:
  Nov 2025: $675,000
  Dec 2025: $1,140,000
  Jan 2026: $1,875,000
  Feb 2026: $2,793,000
  Aug 2026: $2,892,000 (flat line)

Chart title: "Cumulative Won Revenue Over Time"
```

### 2.5 Configure Cross-Filter Interactions
```
Dashboard Settings → Interactions

Rule 1: Date Filter → applies to ALL tiles + chart
Rule 2: Clicking chart data point → filters all KPI tiles
Rule 3: KPI tiles → display only (no broadcast)

How to set:
  Click each visualization → "Use as Filter" → ON
  This means clicking any bar/point in the chart will filter the KPIs
```

### 2.6 Add Navigation Links (Row 4 or footer)
```
ADD BUTTON/LINK — "Opportunity Details >"
  Action type:    Navigate to Dashboard
  Target:         Opportunity Details dashboard (if it exists)
  Style:          Text link or outlined button

ADD BUTTON/LINK — "Regional Sales >"
  Action type:    Navigate to Dashboard
  Target:         Regional Sales dashboard (if it exists)
  Style:          Text link or outlined button
```

### 2.7 Mobile Layout
```
Dashboard → View → Mobile
Reorder widgets (top to bottom):
  1. Title "DTC Sales"
  2. Year Filter
  3. Total KPI tile (full width)
  4. Open KPI tile (full width)
  5. Won KPI tile (full width)
  6. Lost KPI tile (full width)
  7. Cumulative chart (full width)
  8. Navigation links
→ Save mobile layout
```

---

## 🚀 STEP 3 — Enable New Capabilities (45 min)

### 3.1 Proactive Alert on Won Revenue
```
Tableau Next → Metrics → Won Revenue
→ Click "Set Alert"
Condition:   Won Revenue drops below $500,000 in any month
Notify:      [your email] + Slack channel (if connected)
Frequency:   Real-time / Daily check
→ Save Alert
```

### 3.2 Tableau Agent on Dashboard
```
On the DTC Sales dashboard:
Settings → Enable Tableau Agent
→ Toggle ON

Users can now type:
"Compare won revenue 2025 vs 2026"
"Which country has the best win rate?"
"Show me pipeline by stage for Germany"
```

### 3.3 Slack Integration (if Slack is connected)
```
Tableau Next → Metrics → Won Revenue
→ Share to Slack → Select #sales-ops channel
→ Set: Weekly snapshot every Monday 9:00 AM
→ Pin metric to channel
```

### 3.4 Subscribe Stakeholders
```
Dashboard → Share → Schedule
Frequency: Weekly (every Monday)
Recipients: [key stakeholders]
Format: PDF snapshot
→ Save subscription
```

---

## ✅ STEP 4 — Validate & Go Live (1 hr)

### 4.1 Number Validation Checklist
Open the dashboard and verify against these baseline values:

| KPI | Expected | Actual (your dashboard) | ✅/❌ |
|-----|----------|------------------------|-------|
| Total Pipeline | $8,676,000 | | |
| Open Pipeline | $2,892,000 | | |
| Won Revenue | $2,892,000 | | |
| Lost Revenue | $2,892,000 | | |
| Win Rate % | 50.0% | | |

### 4.2 Filter Validation
```
✅ Select year "2026" → all tiles update to 2026 values
✅ Select year "2025" → all tiles update to 2025 values
✅ Clear filter → returns to full totals ($8,676,000)
✅ Click a month on the chart → KPI tiles filter to that month
```

### 4.3 Tableau Agent Validation
```
Type these in the dashboard's Agent panel:
✅ "What is total pipeline?"              → $8,676,000
✅ "Show won revenue by country"          → Germany, France, UK, Netherlands
✅ "Which industry has the most revenue?" → Retail ($7,317,000)
✅ "What is the win rate?"                → 50%
✅ "Compare 2025 vs 2026 won revenue"     → 2025: $1,140,000 | 2026: $1,752,000
```

### 4.4 Go Live
```
✅ All validations pass
✅ Share dashboard with users
✅ Deprecate / archive old CRM Analytics version
✅ Notify team of new dashboard URL
```

---

## 📋 Summary: What You Build vs. What You Get

| Before (CRM Analytics) | After (Tableau Next) |
|------------------------|---------------------|
| Stale CSV data (6 years old) | Live Salesforce data (daily refresh) |
| 120 static records | 120 records + grows automatically |
| 4 KPI tiles (no comparison) | 4 KPI tiles + YoY % comparison |
| No natural language | Tableau Agent fully enabled |
| No alerts | Proactive alerts on Won Revenue |
| No Slack | Weekly Slack snapshots |
| SAQL window function | Running Total table calculation |
| Static pillbox filter | Dynamic year filter with current-year default |
| No security | Row-level security via Data Cloud policy |
| Win Rate: not built | Win Rate % metric built in |

---

## ⏱️ Total Effort (Revised — Data Cloud gap found)

| Step | Task | Time |
|------|------|------|
| Step 0 | Ingest Opportunity into Data Cloud | 30–60 min |
| Step 1 | Build Semantic Model (5 metrics + dimensions) | 2–3 hrs |
| Step 2 | Build Dashboard (all widgets) | 2–3 hrs |
| Step 3 | Enable Tableau Agent + Alerts + Slack | 45 min |
| Step 4 | Validate + Go Live | 1 hr |
| **Total** | | **~7–9 hours (1–1.5 days)** |

---

## 🧠 Hard-Won Lessons: AnalyticsVisualization XML Deployment

> These lessons were learned during the Olympus Opportunities CRM Analytics → Tableau Next migration. They apply to any AnalyticsVisualization or AnalyticsDashboard deployed via SF CLI.

### Lesson 1: visualSpecification JSON — All style properties must be INSIDE `style`

The `visualSpecification` is a base64-encoded JSON blob. A critical mistake is placing properties like `axis`, `fit`, `headers`, `showDataPlaceholder`, `marks` (style), `referenceLines` at the **top level**. They must all be nested inside `style`:

```json
{
  "layout": "Vizql",
  "rows": ["F2"],
  "columns": ["F1"],
  "marks": { ... },
  "style": {
    "axis": { ... },           ✅ INSIDE style
    "fit": "Standard",         ✅ INSIDE style
    "headers": { ... },        ✅ INSIDE style
    "marks": { ... },          ✅ INSIDE style (style.marks ≠ top-level marks)
    "showDataPlaceholder": false,  ✅ INSIDE style
    "referenceLines": {},      ✅ INSIDE style
    "lines": { ... },          ✅ REQUIRED
    "fonts": { ... }           ✅ REQUIRED
  }
}
```

### Lesson 2: `style.lines` and `style.fonts` are REQUIRED

Omitting either causes: `"Value required for [lines]"` or `"Value required for [fonts]"`. Always include them:

```json
"fonts": {
  "actionableHeaders": {"size": 13, "color": "--slds-g-color-palette-electric-blue-40"},
  "headers": {"size": 13, "color": "--slds-g-color-palette-neutral-20"},
  "fieldLabels": {"size": 13, "color": "--slds-g-color-palette-neutral-20"},
  "marks": {"size": 13, "color": "--slds-g-color-palette-neutral-20"},
  "markLabels": {"size": 13, "color": "--slds-g-color-palette-neutral-20"},
  "legendLabels": {"size": 13, "color": "--slds-g-color-palette-neutral-20"},
  "axisTickLabels": {"size": 13, "color": "--slds-g-color-palette-neutral-20"}
},
"lines": {
  "fieldLabelDividerLine": {"color": "--slds-g-color-palette-neutral-80"},
  "separatorLine": {"color": "--slds-g-color-palette-neutral-80"},
  "axisLine": {"color": "--slds-g-color-palette-neutral-80"},
  "zeroLine": {"color": "--slds-g-color-palette-neutral-80"}
}
```

### Lesson 3: Discrete dimensions require `style.headers.fields.{fieldKey}`

Any text/discrete dimension (Stage, Opportunity_Type1, Opportunity_Name1, etc.) used in `rows` or `columns` requires an entry in `style.headers.fields`:

```json
"style": {
  "headers": {
    "fields": {
      "F1": {
        "isVisible": true,
        "hiddenValues": [],
        "showMissingValues": false,
        "textDirection": "Horizontal"
      }
    }
  }
}
```

Without this: `"headers.fields" style is required for the "Stage" ("F1") field.`

**Continuous dimensions (e.g., `Close_Date1` with `DatePartYear`) do NOT need this entry.**

### Lesson 4: Discrete dimensions must NOT have an `axis` entry

For a discrete text dimension placed in `rows` (horizontal bar layout), do NOT add a `style.axis.fields.{fieldKey}` entry. Only the **measure** (continuous field in columns) should have an axis entry.

```json
"style": {
  "axis": {
    "fields": {
      "F2": { ... }   ✅ Only the measure gets an axis entry
      // F1 (discrete dimension) should NOT be here
    }
  }
}
```

### Lesson 5: Sort order format in `viewSpecification`

For sorting by a measure in a visualization with a discrete dimension in rows, the `viewSpecification` sort must reference the **dimension field** with `byField` pointing to the measure:

```json
// CORRECT — sort dimension F1 descending by measure F2
"sortOrders": {
  "fields": {"F1": {"type": "Nested", "order": "Descending", "byField": "F2"}},
  "rows": [], "columns": []
}

// WRONG — don't sort by the measure field directly
"sortOrders": {
  "fields": {"F2": {"direction": "Descending"}},
  "rows": [], "columns": []
}
```

### Lesson 6: SM field names are auto-generated from the source dataset column names

When a Semantic Model is built on a Wave/Data Cloud CSV dataset, field names are derived from the original column names, **not** from DLO API names. To discover real field names:

```bash
sf project retrieve start --metadata "AnalyticsVisualization:*" --target-org tableauNextOrg
# Then decode the base64 visualSpecification to see real objectName and fieldName values
```

Example for Olympus: `objectName: OlympusOpportunities_DataCloud_csv`  
Real field names: `Amount`, `Close_Date1`, `Stage`, `Opportunity_Type1`, `Opportunity_Name1`, `Opportunity_Owner`

### Lesson 7: Dashboard deploy fails if referenced vizes don't exist in the org

Even if a viz deploy "Succeeded" in a previous session, verify it still exists before deploying the dashboard:

```bash
sf project retrieve start --metadata "AnalyticsVisualization:MyViz" --target-org myOrg
# If: "Entity of type 'AnalyticsVisualization' named 'MyViz' cannot be found"
# → Redeploy the viz first, then deploy the dashboard
```

### Lesson 8: Always deploy vizes and dashboard in the same `sf project deploy start` call or verify existence first

When deploying a dashboard that references new visualizations, either:
- **Option A**: Deploy all vizes + dashboard in a single CLI call (dependency order respected)
- **Option B**: Deploy vizes first, verify they exist via retrieve, then deploy dashboard

Salesforce sometimes fails to persist newly-created AnalyticsVisualization assets if a bundle deploy fails partway through.

### Lesson 9: KPI tiles in Tableau Next can use visualization type (not metric type)

If a Semantic Model doesn't support metric creation via metadata (e.g., dimensions can't be added via CLI), use a `type: visualization` widget pointing to a single-value viz:

```xml
<type>visualization</type>
<vizWidgetDefs>
    <analyticsVisualization>Sum_of_Amount_Number</analyticsVisualization>
    <parameters>{...}</parameters>
</vizWidgetDefs>
```

### Lesson 10: Dashboard filter widget format

Working filter widget structure:

```xml
<filterWidgetDefs>
    <initialValues>null</initialValues>
    <parameters>{
        "viewType": "toggle",
        "filterOption": {
            "objectName": "OlympusOpportunities_DataCloud_csv",
            "fieldName": "Close_Date1",
            "dataType": "Date",
            "selectionType": "multiple"
        },
        "receiveFilterSource": {"filterMode": "all", "widgetIds": [], "publishers": []},
        "receiveParameterSource": {"parameterMode": "all", "publishers": []}
    }</parameters>
    <source>New_Semantic_Model_9fc</source>
</filterWidgetDefs>
<label>Close Date</label>
<type>filter</type>
<widgetName>filter_close_date</widgetName>
```

`source` = SM API name (not workspace). `objectName` = original Wave dataset label.

---

*Generated from live org audit. All KPI values validated via SOQL queries against pr1786449020763.my.salesforce.com on 2026-09-28.*
*Olympus Opportunities migration lessons added 2026-09-30.*
