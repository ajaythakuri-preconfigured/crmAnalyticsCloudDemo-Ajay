# 🔬 Dataset, Recipe, Filter & Data Layer Insights Report
## Dashboard: "DTC Sales" | Dataset: "DTC Opportunity"
**Report Date:** 2026-09-28 | **Org:** preconfigured4-dev-ed.develop.my.salesforce.com

---

## 📋 Executive Summary

| Category | Finding | Severity |
|----------|---------|---------|
| Data Freshness | Dataset data is **6+ years stale** (last refreshed Oct 2020) | 🔴 Critical |
| Data Source | Loaded via **CSV upload** — no live connection to Salesforce | 🔴 Critical |
| Security | **No security predicate** — all users see all 671 rows | 🔴 Critical |
| Filters | Only 1 user filter (year pillbox) — no global dashboard filters | 🟡 Warning |
| Field Utilization | Only **5 of 32+ fields** used in dashboard | 🟡 Warning |
| Recipes | 0 recipes feed this dataset | 🟡 Warning |
| Dataflows | 0 dataflows feed this dataset | 🟡 Warning |
| Scheduling | No recipe or dataflow is scheduled in the org | 🟡 Warning |

---

## 🗄️ Dataset Deep Dive: DTC Opportunity

### Core Metadata
| Property | Value | Notes |
|----------|-------|-------|
| **Dataset ID** | 0Fbhg0000001n71CAA | |
| **API Name** | DTC_Opportunity_SAMPLE | `_SAMPLE` suffix = sample/demo data |
| **Label** | DTC Opportunity | |
| **App** | My DTC Sales | |
| **Total Rows** | **671** | Very small — sample dataset |
| **Dataset Type** | Default | Standard imported dataset |
| **Data Connector** | **CSV** | ⚠️ Static file, not live Salesforce data |
| **Data Refresh Date** | **2020-10-15** | 🔴 Over 6 years stale! |
| **Last Queried** | 2026-09-28 | Actively being used today |
| **Last Modified** | 2026-09-15 | Metadata changed (not data) |
| **Created By** | Integration User | Auto-loaded by Salesforce |
| **Versions** | 3 | All 3 versions have same 671 rows |

---

## 🔴 CRITICAL FINDING #1: Data Is 6 Years Stale

```
Data Refresh Date:  2020-10-15
Today's Date:       2026-09-28
Data Age:           ⚠️  ~6 years, 0 months OLD
```

The `DTC Opportunity` dataset was last loaded with actual data on **October 15, 2020**. Every KPI number, every chart, every insight on the DTC Sales dashboard reflects **6-year-old sample data**.

**Why this happened:** The dataset was loaded via a CSV file upload (a one-time action). Unlike live Salesforce connector datasets, CSV uploads have no automatic refresh mechanism. Once loaded, the data stays frozen until manually re-uploaded.

**Business Impact:** Any business decisions made from this dashboard are based on entirely outdated data.

**Fix:**
- Replace with a **live Salesforce connector** dataset using a Recipe that pulls directly from the `Opportunity` object
- Or implement a **scheduled CSV re-upload process** (not recommended)
- Recommended Recipe design:
  ```
  SFDC_LOCAL: Opportunity → Filter (active records) → Transform → Register as "DTC Opportunity"
  Schedule: Daily at 6:00 AM
  ```

---

## 🔴 CRITICAL FINDING #2: No Security Predicate (Zero Row-Level Security)

```
Security Coverage Sources: []  ← EMPTY — no predicate defined
```

The dataset has **no security predicate** whatsoever. This means:
- **Every user** who can access this dashboard sees **all 671 rows**
- Sales rep A can see Sales rep B's pipeline
- Manager sees same data as an executive
- No territory or ownership-based filtering

**Fix — Add a Security Predicate for Ownership:**
```
'Opportunity_Owner' == "$User.Name"
```
Or for managers (allow hierarchy view):
```
'Owner_Role' == "$User.UserRole.Name" || 'Opportunity_Owner' == "$User.Name"
```

---

## 🔴 CRITICAL FINDING #3: CSV Source — No Live CRM Connection

**Connector Type:** `CSV` (identified from `xmdMain.dataset.connector`)
**Fully Qualified Name:** `DTC_Opportunity_SAMPLE_csv`

The `_SAMPLE` suffix in both the dataset name and the CSV source name confirms this is a **Salesforce-provided demo/sample dataset** — not connected to any real Salesforce CRM data.

```
What exists now:          What should exist:
CSV file (2020)           Live Salesforce Opportunity Object
     ↓                           ↓
DTC_Opportunity_SAMPLE    Recipe (SFDC_LOCAL connector)
     ↓                           ↓
Dashboard queries         Same Dashboard queries
```

---

## 📊 Full Field Schema (32 Fields)

### Dimensions (26 fields)

| Field Name | Label | Used in Dashboard? | Analytical Value |
|-----------|-------|-------------------|-----------------|
| `Opportunity_Name` | Opportunity Name | ❌ No | High — for record-level detail |
| `Account_Name` | Account Name | ❌ No | 🌟 High — account-level analysis |
| `Account_Owner` | Account Owner | ❌ No | High — account ownership tracking |
| `Account_Type` | Account Type | ❌ No | 🌟 High — segment by customer type |
| `Stage` | Stage | ❌ No | 🌟🌟 Critical — pipeline stage analysis |
| `Won` | Won | ✅ Yes (filter) | Used as filter |
| `Closed` | Closed | ✅ Yes (filter) | Used as filter |
| `Forecast_Category` | Forecast Category | ❌ No | 🌟🌟 High — forecasting analysis |
| `Product_Name` | Product Name | ❌ No | 🌟 High — product-level revenue |
| `Product_Family` | Product Family | ❌ No | 🌟 High — product family grouping |
| `Industry` | Industry | ❌ No | 🌟 High — vertical analysis |
| `Segment` | Segment | ❌ No | 🌟 High — customer segment analysis |
| `Opportunity_Source` | Opportunity Source | ❌ No | High — lead source attribution |
| `Opportunity_Owner` | Opportunity Owner | ❌ No | 🌟 High — rep-level performance |
| `Opportunity_Type` | Opportunity Type | ❌ No | Medium — new vs. renewal |
| `Owner_Role` | Owner Role | ❌ No | High — team/region hierarchy |
| `Billing_Country` | Billing Country | ❌ No | 🌟 High — geographic analysis |
| `Billing_State_Province` | Billing State/Province | ❌ No | High — regional detail |
| `Close_Date` | Close Date (full) | ❌ No | Used via date parts |
| `Close_Date_Year` | Close Date Year | ✅ Yes (pillbox+chart) | Used as filter |
| `Close_Date_Month` | Close Date Month | ✅ Yes (chart) | Used in time chart |
| `Close_Date_Quarter` | Close Date Quarter | ❌ No | 🌟 High — quarterly analysis |
| `Close_Date_Week` | Close Date Week | ❌ No | Medium — weekly view |
| `Close_Date_Day` | Close Date Day | ❌ No | Low — too granular for summary |
| `Created_Date` + parts | Created Date | ❌ No | High — pipeline age analysis |

### Measures (6 fields)

| Field Name | Label | Used in Dashboard? | Notes |
|-----------|-------|-------------------|-------|
| `Amount` | Amount | ✅ Yes (all KPIs) | Primary measure |
| `Column9` | # (Row count) | ❌ No | Count of opportunities — unused! |
| `Close_Date_day_epoch` | — | ❌ No | System epoch field |
| `Close_Date_sec_epoch` | — | ❌ No | System epoch field |
| `Created_Date_day_epoch` | — | ❌ No | System epoch field |
| `Created_Date_sec_epoch` | — | ❌ No | System epoch field |

### Field Utilization Summary
```
Total Fields:    32
Fields Used:      5  (Amount, Won, Closed, Close_Date_Year, Close_Date_Month)
Fields Unused:   27  (84% of available fields are untapped!)
```

### 🌟 High-Value Unused Fields (Recommended Additions)

| Field | What You Could Build |
|-------|---------------------|
| `Stage` | Pipeline funnel chart — deals by stage |
| `Forecast_Category` | Forecast vs. commit vs. pipeline breakdown |
| `Product_Family` | Revenue by product line |
| `Industry` | Win rate by industry vertical |
| `Opportunity_Owner` | Leaderboard / rep performance |
| `Billing_Country` | Geographic revenue map |
| `Column9` (`#`) | Opportunity count KPIs (alongside amount) |
| `Close_Date_Quarter` | Quarterly performance trends |
| `Created_Date_Year` | Pipeline creation vs. close date analysis |

---

## 📅 Dataset Version History

| Version ID | Created | Last Modified | Row Count | Notes |
|-----------|---------|--------------|-----------|-------|
| `0Fchg000000CX3lCAG` | 2026-09-11 | 2026-09-15 | 671 | ✅ Current active version |
| `0Fchg000000CX4QCAW` | 2026-09-11 | 2026-09-11 | 671 | Previous version |
| `0Fchg000000CX4RCAW` | 2026-09-11 | 2026-09-11 | 671 | Oldest version |

**Key Observations:**
- All 3 versions have **exactly 671 rows** — the data content has never changed
- The current version was last modified on 2026-09-15 — likely a metadata edit (field labels, formatting), not a data reload
- All versions were created by the **Integration User** (system account)
- No incremental or delta loads have ever occurred

---

## 🔄 Recipes Analysis (2 Recipes in Org)

### Recipe 1: Opportunities with SIC Descriptions
| Property | Value |
|----------|-------|
| **ID** | 05vhg0000002I5hAAE |
| **Status** | Success |
| **Schedule** | ❌ None |
| **Last Run** | Not available |
| **Feeds DTC Dashboard?** | ❌ No |

**Recipe Logic (Decoded):**
```
[Opportunities_with_Accounts_and_Users dataset]  ← Input (CRM Analytics dataset)
              ↓
    LOOKUP JOIN on Account.Sic = SIC_Code
              ↓
[SIC Descriptions dataset]  ← Right side of join
              ↓
    Output → "Opportunities with SIC Descriptions" dataset
```
This recipe enriches opportunities with SIC (Standard Industrial Classification) codes descriptions — adds industry classification detail to account data. **No schedule configured — never auto-refreshes.**

---

### Recipe 2: Opportunities_with_Accounts_and_Users
| Property | Value |
|----------|-------|
| **ID** | 05vhg0000002HzFAAU |
| **Status** | Success |
| **Schedule** | ❌ None |
| **Last Run** | Not available |
| **Feeds DTC Dashboard?** | ❌ No |

**Recipe Logic (Decoded):**
```
[SFDC_LOCAL: Opportunity]     ←  Live Salesforce Opportunity object
Fields: AccountId, Amount, CloseDate, CreatedDate, Id, OwnerId, StageName, Name
              ↓
    LOOKUP JOIN on AccountId = Account.Id
              ↓
[SFDC_LOCAL: Account]         ← Live Salesforce Account object
Fields: Sic, Id, Name, Type, BillingCity, BillingCountry, Industry, OwnerId
              ↓
    + JOIN with User object (truncated)
              ↓
    Output → "Opportunities_with_Accounts_and_Users" dataset
```
This recipe joins live Salesforce CRM data (Opportunity + Account + User). It's the foundation for the SIC Descriptions recipe above. **This is EXACTLY what the DTC dashboard should be using** — but isn't!

**⚠️ Key Gap: This live recipe exists but is unused by the DTC Sales dashboard!** The DTC dashboard uses a stale 2020 CSV instead of this live connected recipe.

---

## ⚙️ Dataflows Analysis (3 Dataflows in Org)

| Dataflow | API Name | Schedule | Last Run | Feeds DTC? |
|---------|---------|---------|---------|-----------|
| salestest eltDataflow | salestest_eltDataflow | ❌ None | Never | ❌ No |
| The_Motivator | The_Motivator | ❌ None | Never | ❌ No |
| Default Salesforce Dataflow | SalesEdgeEltWorkflow | ❌ None | Never | ❌ No |

**Critical Finding: ALL 3 dataflows have NO schedule and have NEVER run.**
- No data is being refreshed automatically anywhere in this org
- The `Default Salesforce Dataflow` (SalesEdgeEltWorkflow) should typically run automatically for the Sales Analytics template — it's not configured here
- `salestest eltDataflow` powers the salestest app datasets (Accounts, Opportunities, Cases, etc.) — but without scheduling, those datasets are also potentially stale

---

## 🔒 Filter Architecture Analysis

### Dashboard-Level Filters
```
Global Filters: [] ← EMPTY — no persistent dashboard filters
```
No global filters are defined. This means there are no pre-applied filters that persist across all steps when the dashboard loads.

### User-Facing Filters (1 Filter)
| Widget | Type | Field | Broadcasts To | Default |
|--------|------|-------|--------------|---------|
| `pillbox_1` | Pillbox (Year Selector) | `Close_Date_Year` | All 6 steps | `[2016]` hardcoded |

**Only ONE user-controllable filter exists** — the year selector. Users cannot filter by:
- Sales rep / Owner
- Product or Product Family
- Region / Country
- Stage
- Account Type / Industry
- Segment

### Step-Level Filters (Hard-coded in SAQL)
| Step | Filter Logic | User-Controllable? |
|------|-------------|-------------------|
| Amount_4 (TOTAL) | None | ❌ No |
| Amount_1 (Open) | Closed = false | ❌ No |
| Amount_2 (Won) | Won = true | ❌ No |
| Amount_3 (Lost) | Won = false AND Closed = true | ❌ No |
| CloseDate_Year_1 (Pillbox) | Closed = true | ❌ No |
| CloseDate_Year_Close_5 (Chart) | Won = true | ❌ No |

All step filters are hardcoded — no dynamic binding to user selections beyond the year pillbox.

### Missing Filters That Would Add Value

| Filter to Add | Widget Type | Field | Business Value |
|--------------|------------|-------|---------------|
| Sales Rep | List Selector | `Opportunity_Owner` | Manager view by rep |
| Product Family | Pillbox | `Product_Family` | Product performance |
| Region/Country | List Selector | `Billing_Country` | Geographic view |
| Stage | Pillbox | `Stage` | Pipeline stage view |
| Industry | List Selector | `Industry` | Vertical analysis |

---

## 🔍 Lenses Related to DTC Dataset

No lenses are directly saved against the DTC Opportunity dataset. The DTC Sales app has **0 lenses** — meaning no ad-hoc exploration has been saved. All analysis is confined to the 4 dashboards.

**Notable Lenses in Other Apps:**
| Lens | App | Insight |
|------|-----|---------|
| D01 - Laptops Salespeople - Wall of Fame | My Exploration | Product-specific analysis |
| D02 - Digital Media Opportunities Evolution / Time | My Exploration | Time series exploration |
| D03 - Digital Media Sales % Evolution / Time | My Exploration | % change analysis |
| M01 - Easy Closing Opportunities In Banking | My Exploration | Industry-filtered view |
| Historical Pipeline By Stage/Forecast | salestest | Pipeline trending lenses |

---

## 🏗️ Full Data Flow Architecture

### Current (Broken) State
```
CSV File (2020 data)
        ↓ One-time upload
DTC_Opportunity_SAMPLE  ← 671 stale rows, no refresh
        ↓
DTC Sales Dashboard  ← Shows 6-year-old data
```

### Recommended Target Architecture
```
Salesforce CRM (Live)
  ├── Opportunity Object
  ├── Account Object
  └── User Object
        ↓
Recipe: Opportunities_with_Accounts_and_Users  (already exists!)
        ↓ (add schedule: daily 6AM)
Live Dataset: DTC_Opportunity  (new, replaces sample)
        ↓
Security Predicate: 'Opportunity_Owner' == "$User.Name"
        ↓
DTC Sales Dashboard  ← Live, fresh, secure data
```

---

## 📊 Summary Scorecard

| Area | Score | Key Finding |
|------|-------|------------|
| **Data Freshness** | 1/10 | 6-year stale CSV data |
| **Data Source Quality** | 2/10 | CSV instead of live CRM connector |
| **Security** | 0/10 | No row-level security at all |
| **Filter Coverage** | 3/10 | Only year filter; 80%+ of fields ignored |
| **Recipe Design** | 7/10 | Good recipes exist but unconnected to this dashboard |
| **Dataflow Health** | 2/10 | No schedules, never run |
| **Field Utilization** | 2/10 | Only 5 of 32 fields used |
| **Overall Data Layer** | **3/10** | Fundamental data infrastructure issues |

---

## 🛠️ Priority Action Plan

### 🔴 Immediate (This Week)
1. **Connect live data:** Modify or clone `Opportunities_with_Accounts_and_Users` recipe to output a new `DTC_Opportunity` dataset using live Salesforce data
2. **Schedule the recipe:** Set daily schedule at 6:00 AM
3. **Apply security predicate:** Add `'Opportunity_Owner' == "$User.Name"` to the new dataset
4. **Update dashboard** to point to new live dataset

### 🟡 Short-Term (Next 2 Weeks)
5. **Add global dashboard filters:** Stage, Product Family, Region
6. **Add count KPIs:** Use `Column9` field to show opportunity counts alongside revenue
7. **Schedule all dataflows:** Enable schedules on `salestest eltDataflow` and `Default Salesforce Dataflow`

### 🟢 Medium-Term (Next Month)
8. **Leverage unused fields:** Add Stage funnel chart, Product Family breakdown, Regional map
9. **Add `Close_Date_Quarter` filter** for quarterly views
10. **Create lenses** in DTC Sales app for ad-hoc exploration
11. **Add `Opportunity_Owner` list selector** for manager-level rep performance drill-down
12. **Build Einstein Discovery model** for opportunity win probability using Stage, Industry, Amount, Forecast_Category

---

*Report generated via live CRM Analytics REST API (v67.0). Dataset schema extracted from XMD metadata. Recipe definitions decoded from R3 format. All findings based on live org data as of 2026-09-28.*
