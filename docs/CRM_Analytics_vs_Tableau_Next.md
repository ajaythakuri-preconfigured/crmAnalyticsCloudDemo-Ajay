# CRM Analytics vs Tableau Next — Detailed Comparison

> Based on hands-on migration experience building the Olympus Opportunities dashboard
> from Salesforce CRM Analytics into Tableau Next (Tableau CRM).
>
> Author: Ajay Thakuri | Date: October 2026

---

## Table of Contents

1. [Overview](#1-overview)
2. [Data Layer](#2-data-layer)
3. [Semantic Model — Tableau Next Only Concept](#3-semantic-model--tableau-next-only-concept)
4. [Workspaces](#4-workspaces)
5. [Visualizations & Charts](#5-visualizations--charts)
6. [Dashboard Building](#6-dashboard-building)
7. [Metadata & Deployment](#7-metadata--deployment)
8. [Developer Experience](#8-developer-experience)
9. [SOQL — Opportunity Object Fields](#9-soql--opportunity-object-fields)
10. [Olympus Dashboard Component Breakdown](#10-olympus-dashboard-component-breakdown)
11. [Summary — When to Use Which](#11-summary--when-to-use-which)

---

## 1. Overview

Salesforce offers two analytics platforms that serve different purposes and architectures:

| | CRM Analytics | Tableau Next |
|---|---|---|
| **Also known as** | Salesforce Analytics, Wave Analytics, Einstein Analytics | Tableau CRM (next-gen), Tableau integrated into Data Cloud |
| **Primary use case** | Embedded analytics on Salesforce CRM data | Visual analytics powered by Data Cloud and VizQL |
| **Query engine** | SAQL (Salesforce Analytics Query Language) | VizQL (Tableau's visual query engine) |
| **Data source** | Salesforce datasets synced into the Analytics platform | Data Cloud Data Lake Objects (DLOs) exposed through Semantic Models |
| **Target user** | Salesforce admins, CRM power users | Data analysts familiar with Tableau |
| **Deployment maturity** | Fully supported via SFDX metadata | Partially supported — key metadata types have restrictions |

---

## 2. Data Layer

### CRM Analytics
- Data is ingested into **proprietary datasets** stored inside the Analytics platform
- Supports CSV uploads, Salesforce object syncs, and dataflow transformations
- Field names in datasets match exactly what you use in charts and SOQL (e.g. `Amount`, `StageName`)
- The dataset API name is directly referenced by lenses and dashboards (e.g. `OlympusOpportunities_DataCloud_csv`)

### Tableau Next
- Data lives in **Salesforce Data Cloud** as **Data Lake Objects (DLOs)**
- A DLO is the physical data store (e.g. `Opportunity_Home__dll`)
- A **Semantic Model** (SM) sits on top of the DLO and defines which fields are exposed, their labels, and relationships
- Field names in the SM have **global org-wide numbered suffixes** — every time a new SM exposes the same underlying field, the counter increments:
  - `Amount` → `Amount1` (first SM) → `Amount2` (second SM) → `Amount3` (third SM)
  - `CloseDate` → `Close_Date1` → `Close_Date2` → `Close_Date3`
  - `StageName` → `Stage1` → `Stage2`
- These suffixed names are what you use in viz metadata and filters — **not** the SOQL field names

### Field Name Mapping (Olympus Project)

| SOQL Field | CRM Analytics Dataset Field | Tableau Next SM Field (`New_Semantic_Model_9fc2`) |
|---|---|---|
| `Name` | `Opportunity_Name1` | `Name2` |
| `Amount` | `Amount` | `Amount2` |
| `CloseDate` | `Close_Date1` | `Close_Date3` |
| `StageName` | `Stage2` | `Stage2` |
| `Type` | `Opportunity_Type1` | `Type1` |
| `Owner.Name` | `Opportunity_Owner` | `Owner_Name2` |

---

## 3. Semantic Model — Tableau Next Only Concept

This is the **biggest architectural difference** between the two platforms.

### What is a Semantic Model?
A Semantic Model (SM) is a metadata layer between the raw DLO and visualizations. It defines:
- Which DLO fields are exposed and under what names
- Field data types (measure vs dimension, continuous vs discrete)
- Relationships between objects
- Calculated fields and aggregations

### Key Restrictions Discovered

**1. SM references in a viz are immutable after first deploy**

Once a visualization is first created/deployed against a specific SM, the platform **permanently locks** the viz to that SM. If you deploy the same viz API name pointing to a different SM, the platform silently reverts it back to the original SM on the next retrieve. The only workaround is to create a brand-new viz with a completely different API name.

> **Real example from migration**: We deployed `Olympus_Rev_by_Year` pointing to `New_Semantic_Model_9fc2`. After deploy, a retrieve showed it had reverted back to `New_Semantic_Model_9fc` (Anchit's original SM). This happened because the viz had been first created against that SM.

**2. SM metadata is not retrievable via SFDX**

The `AnalyticsSemanticModel` metadata type is **not registered in the SFDX metadata registry**. You cannot run `sf project retrieve start --metadata AnalyticsSemanticModel`. There is no supported way to retrieve SM definitions via the CLI.

**3. Wave REST API is restricted**

The CRM Analytics Wave REST API (`/services/data/v67.0/wave/semanticModels`) is not enabled for all user profiles in Tableau Next orgs. It returns `FUNCTIONALITY_NOT_ENABLED` for standard users, blocking programmatic field discovery.

**4. Field discovery requires trial and error**

Because the SM cannot be retrieved, the only way to discover exact field names (e.g. is it `Stage2` or `StageName2`?) is the **deploy-and-fail approach** — deploy a viz with a guessed field name, read the error message, correct it, redeploy.

---

## 4. Workspaces

| | CRM Analytics | Tableau Next |
|---|---|---|
| **Equivalent concept** | Apps | Workspaces |
| **What it contains** | Datasets, lenses, dashboards | SMs, DLOs, vizes, dashboards |
| **Cross-workspace access** | Datasets can be referenced across apps | **A viz cannot use an SM from a different workspace** |
| **Runtime error** | N/A | "We couldn't find the Dashboard: [name]" — thrown when a viz references an SM outside its workspace |
| **Asset registration** | Assets belong to an app automatically | Must be declared via `workspaceAssetRelationships` in both the asset XML and the workspace XML |

### Cross-Workspace SM Restriction — Root Cause of Dashboard Failures

The error `We couldn't find the Dashboard: [DashboardName]` is thrown at runtime when:
- A dashboard in Workspace A contains a viz that uses an SM from Workspace B
- This happens invisibly when the SM reversion issue (point 3 above) causes a viz in Workspace A to silently revert to an SM that lives in Workspace B

---

## 5. Visualizations & Charts

### CRM Analytics
- Charts defined as JSON widgets **inside** the dashboard JSON file
- Chart types: bar, column, donut, pie, map, scatter, timeline, funnel, gauge, KPI/BAN, table, waterfall
- **Parameter controls** are native — Top N selectors (e.g. 10 / 20 / 50 toggle), sliders, dependent picklists
- SAQL handles grouping, filtering, and calculations inline

### Tableau Next
- Each chart is a **separate `.uaviz-meta.xml` file** deployed independently
- Chart types follow Tableau's VizQL engine (bar, line, scatter, donut, BAN/KPI)
- Viz definition includes a **base64-encoded `visualSpecification`** blob containing JSON layout/style/mark config
- **`viewSpecification`** is a separate JSON blob controlling sort orders and filters
- Field keys (`F1`, `F2`) are mapped in `<fields>` elements — the visualSpec uses `F1`/`F2`, not actual field names

### Visualization File Structure (Tableau Next)

```xml
<AnalyticsVisualization>
    <analyticsWorkspace>Workspace_API_Name</analyticsWorkspace>
    <dataSource>SM_API_Name</dataSource>         <!-- Semantic Model -->
    <fields>
        <fieldKey>F1</fieldKey>
        <fieldName>Stage2</fieldName>             <!-- SM field name with suffix -->
        <objectName>Opportunity_Home1</objectName> <!-- DLO object name -->
        <role>Dimension</role>
    </fields>
    <fields>
        <fieldKey>F2</fieldKey>
        <fieldName>Amount2</fieldName>
        <function>Sum</function>
        <role>Measure</role>
    </fields>
    <visualSpecification>[base64 encoded JSON]</visualSpecification>
</AnalyticsVisualization>
```

---

## 6. Dashboard Building

### Layout System
Both platforms use a **grid-based layout**:
- 48 columns, configurable row height (default 20px in Tableau Next)
- Widgets placed by `column`, `row`, `colspan`, `rowspan`

### Filter Widgets

| | CRM Analytics | Tableau Next |
|---|---|---|
| **Dropdown** | `selectMode: single/multi` | `viewType: comboBox` (may cause runtime errors when deployed via metadata — use `toggle` instead) |
| **Toggle** | Supported | `viewType: toggle` — safest option via metadata deploy |
| **Date range** | Native date range picker | Limited — date filter shows raw ISO timestamp (`2025-11-09T00:00:00+00:00`) in some configurations |
| **Dependent filters** | Supported natively | Not confirmed in Tableau Next metadata |

### Theme & Styling

| | CRM Analytics | Tableau Next |
|---|---|---|
| **Background color** | Set via dashboard JSON | Set via `style` JSON string in dashboard XML using hex or CSS token |
| **Dark theme** | Native dark themes available | Achieved via `"backgroundColor":"#03234D"` in layout and widget styles |
| **Widget borders** | Configurable | `borderEdges`, `borderColor`, `borderWidth`, `borderRadius` in widget parameters JSON |

### Critical XML Rule for Tableau Next
**Do not put XML comments (`<!-- -->`) inside `<pages>` or `<widgets>` elements in dashboard metadata.** The SFDX deploy will succeed, but Tableau Next's runtime parser will throw "We couldn't find the Dashboard" when opening the dashboard. CRM Analytics has no such restriction.

---

## 7. Metadata & Deployment

### Supported Metadata Types

| Type | CRM Analytics | Tableau Next |
|---|---|---|
| Workspace / App | `WaveApplication` | `AnalyticsWorkspace` |
| Dashboard | `WaveDashboard` | `AnalyticsDashboard` |
| Lens / Visualization | `WaveLens` | `AnalyticsVisualization` |
| Dataset / DLO | `WaveDataset` | Not directly deployable via SFDX |
| Semantic Model | N/A | `AnalyticsSemanticModel` — **NOT in SFDX registry** |

### Deployment Behavior Comparison

| Scenario | CRM Analytics | Tableau Next |
|---|---|---|
| Change a chart's data source | Works — update JSON and deploy | **Reverts silently** if the viz already existed with a different SM |
| Deploy new dashboard | Works reliably | Works IF all referenced vizes use the correct workspace's SM |
| Retrieve what you deployed | Returns exact metadata | May return **different metadata** (SM reverted, fields normalized) |
| XML comments in metadata files | Fine | **Causes runtime failures** even though deploy succeeds |
| Add new workspace assets | Declare in app JSON | Must declare in both the asset XML and the workspace XML via `workspaceAssetRelationships` |

---

## 8. Developer Experience

### CRM Analytics
- **SAQL** is powerful but proprietary — steep learning curve
- Full Wave REST API access for datasets, lenses, dashboards, dataflows
- What you deploy is what you get — no silent reversions
- Field names are consistent throughout (dataset → chart → SOQL)
- Dashboard JSON is human-readable and directly editable

### Tableau Next
- **VizQL** is familiar to Tableau users — drag and drop in UI, but metadata is complex
- `visualSpecification` is base64-encoded JSON — not human-readable without decoding
- SM field names differ from SOQL names — requires a mental mapping layer
- No CLI/API to introspect SM fields — field discovery is manual/iterative
- The deploy-and-fail loop is the only reliable field name discovery method
- Brand-new API names are required when migrating existing vizes to a new SM

### Error Messages

| Error | CRM Analytics | Tableau Next |
|---|---|---|
| Wrong field name | Specific error naming the field and dataset | "F1 isn't a valid field in the semantic model" |
| Wrong data source | Specific error naming the dataset | "We couldn't find the Dashboard: [name]" (generic — covers multiple root causes) |
| Cross-workspace issue | N/A | "We couldn't find the Dashboard: [name]" (same generic error) |
| Invalid XML in metadata | Deploy fails with XML parse error | Deploy **succeeds**, runtime **fails** |

---

## 9. SOQL — Opportunity Object Fields

The following SOQL query retrieves all fields used in the Olympus Opportunities dashboard from the CRM Analytics org:

```sql
SELECT
    Id,
    Name,
    Amount,
    CloseDate,
    StageName,
    Type,
    Owner.Name,
    Account.Name,
    Account.Type,
    Account.BillingCountry
FROM Opportunity
```

### Sample Data (from `crmsAnalyticsAnchit` org)

| Name | Amount | CloseDate | StageName | Type | Owner | Account | Acct Type | Country |
|---|---|---|---|---|---|---|---|---|
| Opportunity for Edwards146 | 4,212,140 | 2033-09-25 | Perception Analysis | New Business | Johnny Green | Harris13 Inc | Customer | Normway |
| Opportunity for Williams149 | 2,305,550 | 2033-09-06 | Needs Analysis | New Business | Evelyn Williamson | Dean902 Inc | Customer | Australia |
| Opportunity for McDonald13 | 240,747 | 2032-08-04 | Closed Won | New Business / Add-on | Dennis Howard | Parks99 Inc | Partner | USA |

---

## 10. Olympus Dashboard Component Breakdown

### CRM Analytics Original Dashboard

```
┌─────────────────────────────────────────────────────────────────┐
│  Revenue YTD: 124.2M [89M bar]  │ Owner Filter │ Close Date    │
├─────────────────────────────────┼──────────────────────────────┤
│  Opportunity_Type Mix (donut)   │  [10]  [20]  [50]  Top N     │
│  • New Business                 ├──────────────────────────────┤
│  • Existing Business            │  Top Opportunities (h-bar)   │
│  • New Business / Add-on        │  Butler33        ████████ 8.6M│
│                                 │  Figuueroa377    █████  5.9M  │
├─────────────────────────────────┤  Lee231          █████  5.4M  │
│  Account_Type Mix (donut)       │  Cunningham175   ████   4.4M  │
│  • Customer                     │  Edwards146      ████   4.2M  │
│  • Partner                      │  Bush283         ████   4.1M  │
│                                 │  Bass32          ████   4.0M  │
│                                 │  Smith309        ███    3.7M  │
│                                 │  McKenzie412     ███    3.6M  │
│                                 │  Bryant359       ███    3.0M  │
├─────────────────────────────────┴──────────────────────────────┤
│  Details Table (white bg)                              [Open]  │
│  Opp Type │ Acct Type │ Opp Name │ Stage │ Country │ Acct │ $  │
└─────────────────────────────────────────────────────────────────┘
```

### Widget Inventory

| # | Widget | Type | Fields |
|---|---|---|---|
| 1 | Revenue YTD | KPI / BAN | `Amount` (Sum) |
| 2 | Opportunity Owner | Filter – Dropdown | `Owner.Name` |
| 3 | Close Date | Filter – Dropdown | `CloseDate` |
| 4 | Opportunity Type Mix | Donut chart | `Type` (slices), `Amount` (size) |
| 5 | Account Type Mix | Donut chart | `Account.Type` (slices), `Amount` (size) |
| 6 | Top N Selector | Parameter toggle | 10 / 20 / 50 |
| 7 | Top Opportunities | Horizontal bar chart | `Name` (Y-axis), `Amount` (X-axis), sorted desc |
| 8 | Details | Table / Grid | `Type`, `Account.Type`, `Name`, `StageName`, `BillingCountry`, `Account.Name`, `Amount` |

---

## 11. Summary — When to Use Which

| Use Case | CRM Analytics | Tableau Next |
|---|---|---|
| Data lives in Salesforce CRM objects | ✅ Direct sync | ✅ Via Data Cloud DLO |
| Data lives in Data Cloud / external sources | ❌ Needs ETL | ✅ Native |
| Complex inline calculations (SAQL) | ✅ | ❌ Limited |
| Tableau-style drag-and-drop analysis | ❌ | ✅ |
| Reliable CI/CD metadata deployment | ✅ Fully supported | ⚠️ SM immutability makes it fragile |
| Sharing data across workspaces / apps | ✅ Easy | ⚠️ Cross-workspace SM restriction |
| Top-N parameter controls | ✅ Native | ⚠️ Workaround needed |
| SM / data model introspection via API | ✅ Wave API | ⚠️ Restricted / not in SFDX registry |
| Field name consistency (SOQL ↔ chart) | ✅ Same names | ❌ Suffixed names differ from SOQL |
| Dark theme dashboards | ✅ Native | ✅ Via hex color overrides |
| Human-readable metadata | ✅ JSON | ⚠️ Base64-encoded visualSpec blobs |

---

*Document generated from hands-on CRM Analytics → Tableau Next migration of the Olympus Opportunities dashboard.*
