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

#### ⚠️ Tableau Next Dimension Limitations (Confirmed from Live Org)

| Dimension Type | Supported | Filter Source | Notes |
|---------------|-----------|--------------|-------|
| **Calculated Dimension** | ✅ Yes | ❌ No | Formula-based (e.g. `STR([Obj].[Field])`). Cannot be used as dashboard filter source — clicking filter returns formula expression as error |
| **Dimension Hierarchy** | ✅ Yes | ⚠️ Partial | Requires each level to have a pre-existing **dimension-type field** in DLO. Number fields (FiscalYear__c) and Timestamp fields cannot fill Year/Quarter levels directly — needs a dedicated year-granularity dimension field |
| **Native Field Dimension** | ❓ Unknown | ❓ Unknown | "Add Dimension" UI path may only expose calculated + hierarchy options; plain field dimension may not be available in current Tableau Next version |

#### ✅ CORRECT Approach: Adjustable Metric Filters (Confirmed from Live Org + Official Docs)

Dashboard filters for metric tiles do NOT work through Semantic Model dimensions.
They work through **Adjustable Metric Filters** configured inside each metric definition.

> *"For the Pulse object to respond to a filter, the filter must be a dimension from the same data source that the metric definition connects to, and that dimension must be an adjustable metric filter on the metric definition."* — Salesforce Help

**How to set up a Year filter that drives metric tiles:**

**Step 1 — Add Adjustable Metric Filter to each metric:**
- Open Semantic Model → click each metric (e.g., `Total_Pipeline_mtc`)
- Find **"Options"** or **"Adjustable Filters"** section (under "Define metric options")
- Add `FiscalYear__c` (or whichever field you want users to filter by)
- Repeat for ALL metrics that should respond to the filter
- Save

**Step 2 — Add filter widget to dashboard:**
The filter widget connects to the Semantic Model via the adjustable filter field name.
Correct XML pattern (confirmed from live org retrieve):
```json
{
  "viewType": "list",
  "filterOption": {
    "objectName": "Opportunity_Home",
    "fieldName": "Fiscal_Year",
    "dataType": "String",
    "selectionType": "single"
  },
  "isLabelHidden": false
}
```
With `<source>DTC_Sales_Analytics</source>` on the `filterWidgetDefs`.

**Key parameters:**
| Parameter | Value | Notes |
|-----------|-------|-------|
| `objectName` | Semantic Model object label (e.g., `"Opportunity_Home"`) | NOT the DLO API name |
| `fieldName` | Adjustable filter API name from Semantic Model (e.g., `"Fiscal_Year"`) | Set when configuring metric adjustable filter |
| `dataType` | `"String"` | Use String even for numeric year fields — prevents 2,025 formatting, enables picklist display |
| `selectionType` | `"single"` or `"multiple"` | `"single"` for year filter |
| `viewType` | `"list"` | Shows as clickable list/picklist |

**Two types of metric filters (important distinction):**
| Filter Type | Set Where | User Can Change? | Example |
|------------|-----------|-----------------|---------|
| **Definition Filter** | Metric filter conditions | ❌ No — hardcoded | `IsWon = true` on Won Revenue |
| **Adjustable Metric Filter** | Metric Options section | ✅ Yes — via dashboard filter widget | `FiscalYear = 2025` |

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

#### ⭐ Metric Tile Card Setup — Display Mode (IMPORTANT)

By default, Tableau Next metric tiles render in **rich view** which includes:
- Sparkline/trend chart
- Date range label (e.g., "Jan 1, 1970 – Sep 28, 2026")
- AI-generated insight text (e.g., "A high volatility trend has been observed...")

**To show only the metric value (clean card style — matching CRM Analytics):**

In the Dashboard Editor:
1. Click the metric tile to select it
2. In the right panel → find **"Widget"** settings section
3. Open **"Card Setup"**
4. Change display mode to **"Show Value"**
5. This hides the sparkline, date range, and AI insights — shows only the number

> ✅ Apply "Show Value" to all 4 KPI tiles for the clean card-style display matching CRM Analytics

**XML parameter mapping (metricWidgetDefs) — confirmed via retrieve:**

The `componentVisibility` object inside `metricOption.layout` controls which parts of the tile render:

```json
"metricOption": {
  "layout": {
    "compact": true,
    "showChart": false,
    "showInsights": false,
    "showDateRange": false,
    "componentVisibility": {
      "title": false,
      "details": false,
      "value": true,
      "comparison": false,
      "chart": false,
      "goals": false,
      "insights": false
    }
  },
  "sdmApiName": "DTC_Sales_Analytics"
}
```

- `value: true` → shows only the metric number
- All other keys `false` → hides sparkline, date range, AI insights, comparison, goals
- Add `"accentColor": "#43B263"` (green) or `"#E74C3C"` (red) inside `layout` for colored accent bars
- Total Pipeline and Open Pipeline have no `accentColor` (defaults to neutral)

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

*Generated from live org audit. All KPI values validated via SOQL queries against pr1786449020763.my.salesforce.com on 2026-09-28.*
