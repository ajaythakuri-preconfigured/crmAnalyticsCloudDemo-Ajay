# Migrating Dashboards from CRM Analytics to Tableau Next
## A Practical Step-by-Step Guide

> Based on real hands-on migration of two dashboards:
> - **DTC Sales Dashboard** — DTC Sales pipeline analytics
> - **Olympus Opportunities Dashboard** — Revenue, Top Opps, Type Mix, Account Mix
>
> Author: Ajay Thakuri | Date: October 2026

---

## Overview

Migrating from CRM Analytics to Tableau Next is **not a lift-and-shift**.
The two platforms have fundamentally different data architectures, and dashboards
cannot be exported from one and imported into the other.

The migration is a **rebuild** — you recreate each dashboard component from scratch
in Tableau Next, pointed at the equivalent data in Data Cloud via a Semantic Model.

### High-Level Migration Flow

```
CRM Analytics                         Tableau Next
─────────────────────────────────────────────────────────
App (e.g. Olympus)          →    Workspace
Dataset (e.g. OlympusOpps)  →    Data Lake Object (DLO) → Semantic Model
Lens / Widget               →    Visualization (.uaviz-meta.xml)
Dashboard                   →    Dashboard (.uadash-meta.xml)
SAQL query                  →    VizQL / SM field configuration
```

---

## Step 1 — Audit Your CRM Analytics Dashboard

Before touching Tableau Next, fully document what you are migrating.

### 1a. List every widget on the dashboard

For each widget record:
- Widget type (KPI, bar chart, donut, table, filter, text, parameter)
- Data source (which dataset)
- Fields used (dimensions and measures)
- Aggregations (Sum, Count, Average, etc.)
- Sort order
- Filters applied

### 1b. Run a SOQL query to confirm field names

CRM Analytics dataset field names map to Salesforce object fields.
Run this against your CRM Analytics org to confirm the exact field names
and see real data:

```sql
-- DTC Sales example
SELECT Id, Name, Amount, CloseDate, StageName, Type,
       Owner.Name, Account.Name, Account.Type, Account.BillingCountry
FROM Opportunity

-- Or for a specific dataset object — check via the dataset API
```

> **Why this matters**: CRM Analytics field names (e.g. `Amount`, `StageName`)
> will become different names in Tableau Next's Semantic Model
> (e.g. `Amount2`, `Stage2`). You need to know your starting point.

### 1c. Screenshot every dashboard state

Take screenshots of:
- The full dashboard (default state)
- Each individual widget in isolation
- Any filter interactions
- Mobile layout if applicable

> We used `Olympus_Opportunities_CRM Analytics.png` as our reference throughout
> the Olympus migration.

---

## Step 2 — Set Up Data Cloud

Tableau Next analytics runs on top of **Salesforce Data Cloud**.
Your CRM Analytics datasets must exist as Data Lake Objects (DLOs) in Data Cloud.

### 2a. Ingest data into Data Cloud

Options:
- **Salesforce CRM Connector** — sync standard objects (Opportunity, Account, Contact)
  directly from your Salesforce org into Data Cloud
- **CSV upload** — for flat files (as used in the Olympus migration: `OlympusOpportunities_DataCloud_csv`)
- **Cloud storage connector** — AWS S3, Azure Blob, Google Cloud Storage
- **MuleSoft / third-party connectors** — for external systems

> In our Olympus migration, the data was ingested as a CSV into a DLO called
> `Opportunity_Home__dll`.

### 2b. Verify the DLO exists and has data

In Data Cloud setup:
1. Go to **Data Cloud → Data Lake Objects**
2. Confirm your DLO is listed (e.g. `Opportunity_Home__dll`)
3. Preview the data — check field names as they appear in the DLO
   (these will differ from both SOQL names and final SM names)

### 2c. Note the DLO API name

You will need the exact DLO API name for the Semantic Model step.

> In our project: `Opportunity_Home__dll`

---

## Step 3 — Create the Tableau Next Workspace

A Workspace in Tableau Next is the equivalent of a CRM Analytics App.
All assets (Semantic Model, vizes, dashboards) must belong to the same workspace.

### 3a. Create the workspace in the UI

1. Go to **Tableau Next → Workspaces**
2. Click **New Workspace**
3. Give it a meaningful name (e.g. `Olympus Opportunities Ajay`)
4. Note the **API name** that gets assigned (e.g. `Olympus_Workspace_Ajay`)

> **Critical rule**: Every viz and dashboard you create must belong to this workspace.
> A viz CANNOT use a Semantic Model from a different workspace — this causes
> the runtime error `"We couldn't find the Dashboard: [name]"`.

### 3b. Retrieve the workspace metadata

```bash
sf project retrieve start \
  --metadata "AnalyticsWorkspace:Olympus_Workspace_Ajay" \
  --target-org tableauNextOrg
```

Confirm the workspace XML references your DLO and SM correctly:

```xml
<AnalyticsWorkspace>
    <masterLabel>Olympus Opportunities Ajay</masterLabel>
    <workspaceAssetRelationships>
        <asset>Opportunity_Home__dll</asset>
        <assetType>MktDataLakeObject</assetType>
        <assetUsageType>Referenced</assetUsageType>
        <workspace>Olympus_Workspace_Ajay</workspace>
    </workspaceAssetRelationships>
    <workspaceAssetRelationships>
        <asset>New_Semantic_Model_9fc2</asset>
        <assetType>SemanticModel</assetType>
        <assetUsageType>Created</assetUsageType>
        <workspace>Olympus_Workspace_Ajay</workspace>
    </workspaceAssetRelationships>
</AnalyticsWorkspace>
```

---

## Step 4 — Create the Semantic Model (UI Only)

The Semantic Model (SM) is the most critical step and **must be done in the UI**.
The `AnalyticsSemanticModel` metadata type is NOT in the SFDX registry and cannot
be deployed via `sf project deploy start`.

### 4a. Create the SM in the Tableau Next UI

1. Inside your Workspace, click **New → Semantic Model**
2. Select your DLO as the data source (e.g. `Opportunity_Home__dll`)
3. Add the fields you need — match them to your CRM Analytics dataset fields
4. Set field types: Dimension (text, date) or Measure (number)
5. Set display labels to match your CRM Analytics field labels
6. **Save** — this assigns the SM an API name (e.g. `New_Semantic_Model_9fc2`)

### 4b. Discover the SM field names — CRITICAL

Tableau Next assigns **global org-wide numbered suffixes** to every SM field.
The suffix increments each time any SM in the org exposes the same underlying field.
This means your SM field names will be different from:
- The SOQL field names
- The CRM Analytics dataset field names
- What you might expect

**The only reliable way to discover the exact field names is:**

**Option A — Deploy a test viz and read the error**
```bash
# Create a test viz XML pointing to your SM with a guessed field name
# Deploy it:
sf project deploy start \
  --metadata "AnalyticsVisualization:Test_Viz_Discovery" \
  --target-org tableauNextOrg

# If the field name is wrong, you get:
# "F1 isn't a valid field in the semantic model"
# → Try a different suffix number and redeploy
```

**Option B — Create a viz in the UI and retrieve it**
1. In the Tableau Next UI, open your workspace
2. Create a new viz, drag fields onto the canvas from the data panel
3. Save the viz with a recognisable name (e.g. `Test_Viz_Ajay`)
4. Retrieve it via CLI:
```bash
sf project retrieve start \
  --metadata "AnalyticsVisualization:Test_Viz_Ajay" \
  --target-org tableauNextOrg
```
5. Read the retrieved XML — the `<fieldName>` elements contain the exact SM field names

> **From our Olympus migration — confirmed field names for `New_Semantic_Model_9fc2`:**
>
> | CRM Analytics Field | Tableau Next SM Field |
> |---|---|
> | `Amount` | `Amount2` |
> | `Opportunity_Name` | `Name2` |
> | `Close_Date` | `Close_Date3` |
> | `StageName` | `Stage2` |
> | `Opportunity_Type` | `Type1` |
> | `Owner_Name` | `Owner_Name2` |

### 4c. Build a field name map

Create a mapping table before building any vizes:

```
CRM Analytics field  →  SOQL field  →  Tableau Next SM field
Amount               →  Amount      →  Amount2
Opportunity Name     →  Name        →  Name2
Close Date           →  CloseDate   →  Close_Date3
Stage                →  StageName   →  Stage2
Opportunity Type     →  Type        →  Type1
Owner Name           →  Owner.Name  →  Owner_Name2
Account Type         →  Account.Type  →  (discover via test viz)
Billing Country      →  Account.BillingCountry → (discover via test viz)
```

---

## Step 5 — Recreate Visualizations (One by One)

Each CRM Analytics widget becomes a separate `.uaviz-meta.xml` file in Tableau Next.

### 5a. The golden rule — ALWAYS use new API names

> **This is the single most important lesson from our migration.**

If you use the same API name as an existing viz that was ever associated with
a different SM, the Tableau Next platform will **silently revert** the viz back
to the original SM after deploy — even if the deploy reports `Status: Succeeded`.

**Always create completely new API names for every viz you build for a new workspace.**

❌ Do not reuse: `Olympus_Rev_by_Year`, `Olympus_Type_Mix`, `Sum_of_Amount_Number`
✅ Create new: `Rev_by_Year_Ajay_V2`, `Opp_Type_Mix_Ajay_V2`, `KPI_Revenue_Ajay_V2`

### 5b. Verify SM binding after every deploy

After deploying a new viz, immediately retrieve it and check the `<dataSource>`:

```bash
sf project retrieve start \
  --metadata "AnalyticsVisualization:KPI_Revenue_Ajay_V2" \
  --target-org tableauNextOrg

# Then check:
grep "dataSource\|objectName\|fieldName" \
  force-app/main/default/analyticsVisualizations/KPI_Revenue_Ajay_V2.uaviz-meta.xml
```

Expected output:
```
<dataSource>New_Semantic_Model_9fc2</dataSource>   ✅ Correct
<objectName>Opportunity_Home1</objectName>          ✅ Correct
<fieldName>Amount2</fieldName>                      ✅ Correct
```

If you see `<dataSource>New_Semantic_Model_9fc</dataSource>` (wrong SM) — the viz
was created in the org under a different API name before. Delete it, pick a new API
name, and deploy again.

### 5c. Viz XML template

Use this structure for every viz:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AnalyticsVisualization xmlns="http://soap.sforce.com/2006/04/metadata"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <analyticsWorkspace>YOUR_WORKSPACE_API_NAME</analyticsWorkspace>
    <dataSource>YOUR_SM_API_NAME</dataSource>
    <fields>
        <displayCategory>Discrete</displayCategory>
        <fieldKey>F1</fieldKey>
        <fieldName>Stage2</fieldName>          <!-- SM field name with suffix -->
        <objectName>Opportunity_Home1</objectName>
        <role>Dimension</role>
        <type>Field</type>
    </fields>
    <fields>
        <displayCategory>Continuous</displayCategory>
        <fieldKey>F2</fieldKey>
        <fieldName>Amount2</fieldName>
        <function>Sum</function>
        <objectName>Opportunity_Home1</objectName>
        <role>Measure</role>
        <type>Field</type>
    </fields>
    <masterLabel>Human Readable Label</masterLabel>
    <version>67.14</version>
    <views>
        <fullName>VIZ_API_NAME_default</fullName>
        <isOriginal>true</isOriginal>
        <masterLabel>default</masterLabel>
        <version>67.14</version>
        <viewSpecification>{ sort/filter JSON here }</viewSpecification>
    </views>
    <visualSpecification>BASE64_ENCODED_CHART_CONFIG</visualSpecification>
    <workspaceAssetRelationships>
        <asset xsi:nil="true"/>
        <assetType>AnalyticsVisualization</assetType>
        <assetUsageType>Created</assetUsageType>
        <workspace>YOUR_WORKSPACE_API_NAME</workspace>
    </workspaceAssetRelationships>
</AnalyticsVisualization>
```

### 5d. CRM Analytics widget → Tableau Next viz type mapping

| CRM Analytics Widget | Tableau Next viz layout | Key fields in visualSpec |
|---|---|---|
| KPI / BAN | `"layout":"Radial"`, `"ban":["F2"]`, type `Donut` with no slices | F2 = measure |
| Donut by dimension | `"layout":"Radial"`, `"slices":["F1"]`, `"ban":["F2"]` | F1 = dimension, F2 = measure |
| Vertical bar | `"layout":"Vizql"`, `"rows":["F2"]`, `"columns":["F1"]`, type `Bar` | F1 = dimension, F2 = measure |
| Horizontal bar | `"layout":"Vizql"`, `"rows":["F1"]`, `"columns":["F2"]`, type `Bar` | F1 = dimension (Y), F2 = measure (X) |
| Line chart | `"layout":"Vizql"`, `"rows":["F2"]`, `"columns":["F1"]`, type `Line` | F1 = date, F2 = measure |
| Revenue by Year | `"layout":"Vizql"`, F1 = `Close_DateX` with `"function":"DatePartYear"` | Use `DatePartYear` function on date field |

---

## Step 6 — Build the Dashboard

Once all vizes are deployed and verified, build the dashboard.

### 6a. ALWAYS use a new dashboard API name

Same rule as vizes — never reuse an existing API name.

### 6b. Dashboard XML rules — lessons learned the hard way

**Rule 1: No XML comments inside `<pages>` or `<widgets>`**

```xml
<!-- ❌ This will cause "We couldn't find the Dashboard" at runtime -->
<pages>
    <!-- Header widgets -->
    <pageWidgets>...</pageWidgets>
</pages>

<!-- ✅ No comments anywhere in the dashboard XML -->
<pages>
    <pageWidgets>...</pageWidgets>
</pages>
```

**Rule 2: Use `toggle` for filter widget viewType**

```xml
<!-- ✅ Safe -->
<parameters>{"viewType":"toggle", ...}</parameters>

<!-- ⚠️ May cause runtime issues when deployed via metadata -->
<parameters>{"viewType":"comboBox", ...}</parameters>
```

**Rule 3: Every widget's `<analyticsDashboard>` must match the dashboard API name**

```xml
<widgets>
    <analyticsDashboard>Olympus_Dashboard_Ajay_V4</analyticsDashboard>
    ...
</widgets>
```

**Rule 4: Filter widgets must reference the SM directly**

```xml
<filterWidgetDefs>
    <source>New_Semantic_Model_9fc2</source>  <!-- SM API name, not workspace -->
    <parameters>{"filterOption":{"objectName":"Opportunity_Home1","fieldName":"Stage2",...}}</parameters>
</filterWidgetDefs>
```

### 6c. Dashboard layout system

- 48 columns, `rowHeight` = 20px (default)
- Place widgets using `column`, `row`, `colspan`, `rowspan`
- `column` and `row` are 0-indexed
- Widget widths are in column units (max 48), heights in row units

```
Example: A widget starting at column 12, row 5, spanning 36 columns and 14 rows:
→ X position: 12 × (1200px/48) = 300px from left
→ Y position: 5 × 20px = 100px from top
→ Width: 36 × 25px = 900px
→ Height: 14 × 20px = 280px
```

### 6d. Dashboard XML template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AnalyticsDashboard xmlns="http://soap.sforce.com/2006/04/metadata"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <analyticsWorkspace>YOUR_WORKSPACE_API_NAME</analyticsWorkspace>
    <customConfig>null</customConfig>
    <layouts>
        <analyticsDashboard>DASHBOARD_API_NAME</analyticsDashboard>
        <columnCount>48</columnCount>
        <layoutName>default</layoutName>
        <maxWidth>1200</maxWidth>
        <pages>
            <index>0</index>
            <label>Page Label</label>
            <pageName>page_name</pageName>
            <pageWidgets>
                <analyticsDashboardWidget>widget_name</analyticsDashboardWidget>
                <colspan>12</colspan>
                <column>0</column>
                <row>5</row>
                <rowspan>10</rowspan>
            </pageWidgets>
        </pages>
        <rowHeight>20</rowHeight>
        <style>{"backgroundColor":"#03234D","gutterColor":"#02183A","cellSpacingX":4,"cellSpacingY":4}</style>
    </layouts>
    <masterLabel>Human Readable Dashboard Name</masterLabel>
    <version>67.14</version>
    <widgets>
        <analyticsDashboard>DASHBOARD_API_NAME</analyticsDashboard>
        <type>visualization</type>
        <vizWidgetDefs>
            <analyticsVisualization>VIZ_API_NAME</analyticsVisualization>
            <parameters>{"widgetStyle":{...},"receiveFilterSource":{"filterMode":"all","widgetIds":[],"publishers":[]}}</parameters>
        </vizWidgetDefs>
        <widgetName>widget_name</widgetName>
    </widgets>
    <workspaceAssetRelationships>
        <asset xsi:nil="true"/>
        <assetType>AnalyticsDashboard</assetType>
        <assetUsageType>Created</assetUsageType>
        <workspace>YOUR_WORKSPACE_API_NAME</workspace>
    </workspaceAssetRelationships>
</AnalyticsDashboard>
```

---

## Step 7 — Deploy in the Right Order

Order matters. The platform must know about vizes before the dashboard references them.

```bash
# 1. Deploy all vizes first
sf project deploy start \
  --metadata "AnalyticsVisualization:KPI_Revenue_Ajay_V2" \
  --metadata "AnalyticsVisualization:Opp_Type_Mix_Ajay_V2" \
  --metadata "AnalyticsVisualization:Top_Opps_Ajay_V2" \
  --target-org tableauNextOrg \
  --wait 10

# 2. Verify SM binding on all vizes
sf project retrieve start \
  --metadata "AnalyticsVisualization:KPI_Revenue_Ajay_V2" \
  --metadata "AnalyticsVisualization:Opp_Type_Mix_Ajay_V2" \
  --metadata "AnalyticsVisualization:Top_Opps_Ajay_V2" \
  --target-org tableauNextOrg

grep "dataSource" force-app/main/default/analyticsVisualizations/*.uaviz-meta.xml

# Expected: all show New_Semantic_Model_9fc2 (your correct SM)

# 3. Deploy the dashboard only after vizes are confirmed
sf project deploy start \
  --metadata "AnalyticsDashboard:Olympus_Dashboard_Ajay_V4" \
  --target-org tableauNextOrg \
  --wait 10
```

---

## Step 8 — Test & Validate

### 8a. Check the dashboard opens

Open the dashboard in the Tableau Next UI.

**If you see**: `"We couldn't find the Dashboard: [name]"`

Diagnose in this order:
1. Retrieve all vizes and check `<dataSource>` — did any revert to the wrong SM?
2. Check if any reverted SM belongs to a different workspace
3. Check the dashboard XML for XML comments inside `<pages>` or `<widgets>`
4. Check filter widget `viewType` — change `comboBox` to `toggle`
5. If a viz reverted — delete it from the org, create a new API name, redeploy

### 8b. Validate each widget against the CRM Analytics original

Go widget by widget:

| Widget | What to check |
|---|---|
| KPI / BAN | Number matches CRM Analytics total (same data, same aggregation) |
| Donut chart | Same segments/categories appear, proportions similar |
| Bar chart | Same dimension values on axis, bars sorted correctly |
| Table | Same columns present, data matches |
| Filters | Selecting a filter value updates all connected vizes |

### 8c. Common field name errors

If a viz shows `"F1 isn't a valid field in the semantic model"`:
- The field name suffix is wrong
- Use the deploy-and-fail approach — try the next number (e.g. `Stage2` → `Stage3`)
- Or create a test viz in the UI, drag the correct field, retrieve and read the `<fieldName>`

---

## Step 9 — Commit to Version Control

```bash
# Stage all new files
git add force-app/main/default/analyticsVisualizations/KPI_Revenue_Ajay_V2.uaviz-meta.xml
git add force-app/main/default/analyticsVisualizations/Opp_Type_Mix_Ajay_V2.uaviz-meta.xml
git add force-app/main/default/analyticsVisualizations/Top_Opps_Ajay_V2.uaviz-meta.xml
git add force-app/main/default/analyticsDashboards/Olympus_Dashboard_Ajay_V4.uadash-meta.xml
git add force-app/main/default/analyticsWorkspaces/Olympus_Workspace_Ajay.uawork-meta.xml

git commit -m "Migrate Olympus dashboard to Tableau Next (Workspace Ajay)"
git push origin feature/your-branch-name
```

---

## Full Migration Checklist

```
PREPARATION
  [ ] Screenshot every CRM Analytics dashboard widget
  [ ] Run SOQL to document all field names and data types
  [ ] List every widget: type, fields, aggregations, sort order

DATA CLOUD
  [ ] Confirm DLO exists and has data
  [ ] Note the DLO API name (e.g. Opportunity_Home__dll)

WORKSPACE
  [ ] Create workspace in Tableau Next UI
  [ ] Retrieve workspace XML and confirm SM is registered

SEMANTIC MODEL
  [ ] Create SM in Tableau Next UI (cannot be deployed via SFDX)
  [ ] Note SM API name (e.g. New_Semantic_Model_9fc2)
  [ ] Discover all SM field names (UI test viz or deploy-and-fail)
  [ ] Build a field name mapping table (CRM Analytics → SM suffix names)

VISUALIZATIONS (one per CRM Analytics widget)
  [ ] Choose a brand-new API name (never used before in the org)
  [ ] Create viz XML with correct SM, DLO object name, SM field names
  [ ] Deploy viz
  [ ] Retrieve viz and verify <dataSource> matches your SM
  [ ] Repeat for every viz

DASHBOARD
  [ ] Choose a brand-new API name
  [ ] Build dashboard XML — NO XML comments
  [ ] Use toggle viewType for filter widgets
  [ ] Every widget's <analyticsDashboard> matches the dashboard API name
  [ ] Filter widgets reference SM API name in <source>
  [ ] Deploy dashboard AFTER all vizes are deployed and verified

TESTING
  [ ] Dashboard opens without error
  [ ] Every widget renders data
  [ ] Numbers match CRM Analytics source
  [ ] Filters update vizes correctly
  [ ] SM binding verified on all vizes (no silent reversions)

VERSION CONTROL
  [ ] All XML files committed to Git
  [ ] Pushed to feature branch
  [ ] Pull request raised for review
```

---

## Key Lessons Summary

| # | Lesson | Impact |
|---|---|---|
| 1 | SM references in vizes are immutable after first creation | Any viz redeployed under the same API name reverts to its original SM silently |
| 2 | Always use brand-new API names | The only guaranteed way to get a viz on the right SM from day one |
| 3 | Verify SM binding after EVERY deploy | `sf project retrieve` + `grep dataSource` — don't trust the deploy success message |
| 4 | No XML comments in dashboard XML | Deploy succeeds but dashboard throws runtime error |
| 5 | SM field names use numbered suffixes | `Amount` → `Amount2`, `Stage` → `Stage2` — must be discovered, not guessed |
| 6 | Cross-workspace SM = runtime dashboard error | "We couldn't find the Dashboard" is caused by a viz using an SM from the wrong workspace |
| 7 | `AnalyticsSemanticModel` is not in SFDX registry | SM must be created and managed in the Tableau Next UI, not via CLI |
| 8 | Use `toggle` not `comboBox` for filter viewType | `comboBox` may cause runtime dashboard failures when deployed via metadata |
| 9 | Deploy vizes before the dashboard | Dashboard references vizes — the vizes must exist first |
| 10 | The Semantic Model is the hardest part | Everything else is straightforward once you have correct SM field names |

---

*This guide is based on the hands-on migration of the DTC Sales and Olympus Opportunities
dashboards from Salesforce CRM Analytics to Tableau Next during October 2026.*
