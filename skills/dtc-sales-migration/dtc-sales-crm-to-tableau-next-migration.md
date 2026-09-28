# 🚀 Migration Plan: DTC Sales Dashboard
## CRM Analytics → Tableau Next
**Source:** CRM Analytics | DTC Sales Dashboard (0FKhg0000001YoCGAU)
**Target:** Tableau Next + Data 360
**Report Date:** 2026-09-28
**Prepared By:** Claude AI (Dual-Certified CRM Analytics + Tableau Next Expert)

---

## 📋 Executive Summary

| Dimension | Assessment |
|-----------|-----------|
| **Overall Migration Complexity** | 🟡 Medium |
| **Estimated Effort** | ~12–18 hours (1.5–2.5 days) |
| **Data Layer Effort** | 🟡 Medium (3–5 hrs) |
| **Semantic Model Effort** | 🟡 Medium (3–4 hrs) |
| **Dashboard Rebuild Effort** | 🟢 Low-Medium (3–4 hrs) |
| **Security Migration Effort** | 🟢 Low (1 hr) |
| **Testing & Validation** | 🟢 Low-Medium (2–3 hrs) |
| **Critical Complexity** | Cumulative window function replication |
| **Big Win from Migration** | Tableau Agent, proactive alerts, Slack integration |

---

## 🔄 Concept Translation Map: CRM Analytics → Tableau Next

| CRM Analytics Concept | Tableau Next Equivalent | Notes |
|----------------------|------------------------|-------|
| **App** | **Workspace** | Direct equivalent — container for assets |
| **Dataset** | **Data 360 Dataset + Semantic Model** | Splits into data layer + business layer |
| **SAQL Step (query)** | **Metric (in Semantic Model)** | Business logic moves to semantic layer |
| **Dashboard** | **Dashboard (Visualization Builder)** | Rebuilt, not migrated directly |
| **Lens** | **Tableau Agent conversation** | Ad-hoc exploration via AI conversation |
| **Widget (Number)** | **Metric Tile / KPI Card** | Direct equivalent |
| **Widget (Chart)** | **Visualization** | Rebuilt in Visualization Builder |
| **Widget (Pillbox/Filter)** | **Filter Control / Dimension Filter** | Rebuilt as a date dimension filter |
| **Widget (Text)** | **Text Widget** | Direct equivalent |
| **Widget (Container)** | **Layout Container** | Direct equivalent |
| **Widget (Link)** | **Navigation Action** | Rebuilt as dashboard action |
| **Security Predicate** | **Data 360 Row-Level Security Policy** | More powerful in Data 360 |
| **Global Filters** | **Dashboard Filters** | Direct equivalent |
| **Faceting (broadcastFacet)** | **Cross-filter interactions** | Rebuilt via filter bindings |
| **Recipe/Dataflow** | **Data 360 Data Transform / Recipe** | Similar concept, different platform |
| **Einstein Discovery** | **Tableau Agent + AI Models** | Broader AI capability in Tableau Next |
| **Subscription (email)** | **Proactive Alert + Slack notification** | Enhanced in Tableau Next |
| **Watchlist** | **Metrics page** | Similar monitoring capability |

---

## 📐 Migration Architecture Overview

```
CURRENT STATE (CRM Analytics)              TARGET STATE (Tableau Next + Data 360)
══════════════════════════════             ══════════════════════════════════════

CSV File (2020 stale data)                 Salesforce CRM (Live)
        ↓                                          ↓
DTC_Opportunity_SAMPLE                     Data 360 Ingestion
(671 rows, no security)                    (Opportunity + Account + User objects)
        ↓                                          ↓
6 SAQL Steps (hardcoded)                   Data Model Objects (DMOs)
        ↓                                          ↓
17 Widgets (static)                        Semantic Model
        ↓                                   (Metrics + Dimensions + Descriptions)
CRM Analytics Dashboard                            ↓
(No AI, no alerts)                         Tableau Next Dashboard
                                           + Tableau Agent (NL queries)
                                           + Proactive Alerts
                                           + Slack Integration
```

---

## 🗄️ Phase 1: Data Layer Migration

### Step 1.1 — Ingest Data into Data 360

**What to do:** Connect Salesforce CRM objects to Data 360 as the source of truth.

| Source Object | Fields to Ingest | Data 360 Object |
|--------------|-----------------|----------------|
| Opportunity | Id, Name, Amount, StageName, CloseDate, CreatedDate, AccountId, OwnerId, IsClosed, IsWon, ForecastCategory, Type | DMO: Opportunity |
| Account | Id, Name, Type, Industry, BillingCountry, BillingState, OwnerId, Sic | DMO: Account |
| User | Id, Name, Title, RoleName | DMO: User |

**Configuration:**
```
Connector: SFDC_LOCAL (Salesforce native connector — already exists in org!)
Connection mode: SYNCED (live sync)
Refresh schedule: Daily at 6:00 AM
```

> ✅ Good news: The `Opportunities_with_Accounts_and_Users` recipe in your org already pulls from these exact Salesforce objects — you already have the blueprint!

---

### Step 1.2 — Data Transforms (replaces CRM Analytics Recipe)

**What to replace:** The manual CSV upload + stale data approach.

**Data 360 Transform logic:**
```
Opportunity (DMO)
    ↓ JOIN on AccountId = Account.Id
Account (DMO)
    ↓ JOIN on OwnerId = User.Id
User (DMO)
    ↓ FILTER: active records only
    ↓ COMPUTE: Close_Date_Year, Close_Date_Month (date parts — auto-handled in Data 360)
    ↓ OUTPUT → Unified Opportunity Profile DMO
```

**Key difference from CRM Analytics:**
- CRM Analytics: Manual recipe JSON with sfdcDigest + augment nodes
- Data 360: Visual transform builder + automatic date part generation
- No need to manually create `Close_Date_Year`, `Close_Date_Month` — Data 360 handles date decomposition automatically

---

### Step 1.3 — Row-Level Security (replaces Security Predicate)

**CRM Analytics Predicate (old):**
```
'Opportunity_Owner' == "$User.Name"
```

**Data 360 Policy (new):**
```
Policy Type: Row-Level Security
Object: Opportunity DMO
Rule: OwnerId = {CurrentUser.Id}
Hierarchy: Enable role-based hierarchy (managers see their team's data)
```

**What you GAIN vs. CRM Analytics:**
- ✅ Hierarchy-aware security (managers automatically see subordinates' data)
- ✅ Policy managed centrally in Data 360 — applies to ALL analytics on top (Tableau Next, Agentforce, etc.)
- ✅ No need to duplicate security rules per-dataset

---

## 🧠 Phase 2: Semantic Model (Replaces SAQL Steps)

This is the **core of the migration** — your 6 SAQL steps become Metric definitions in a Tableau Next Semantic Model.

### Semantic Model: "DTC Sales"

**Object:** DTC Opportunity
**Connected to:** Opportunity DMO (from Data 360)

---

### Metric 1: Total Pipeline Amount
**Replaces:** Step `Amount_4`
**Old SAQL:** `{"measures":[["sum","Amount"]]}`
```yaml
Metric Name:        Total Pipeline Amount
Display Label:      Total Pipeline
Description:        Sum of Amount across all opportunities regardless of status.
                    Used by Tableau Agent to answer 'what is our total pipeline?'
Formula:            SUM(Amount)
Format:             Currency, 0 decimal places
Dimensions:         Close_Date_Year, Close_Date_Month, Stage, Product_Family,
                    Industry, Opportunity_Owner, Billing_Country
```

---

### Metric 2: Open Pipeline Amount
**Replaces:** Step `Amount_1`
**Old SAQL:** `{"measures":[["sum","Amount"]],"filters":[["Closed",["false"],"in"]]}`
```yaml
Metric Name:        Open Pipeline Amount
Display Label:      Open Opportunities
Description:        Sum of Amount for opportunities that are still open (not yet closed).
                    Use to understand active pipeline value.
Formula:            SUM(Amount) WHERE IsClosed = false
Format:             Currency, 0 decimal places
```

---

### Metric 3: Won Revenue
**Replaces:** Step `Amount_2`
**Old SAQL:** `{"measures":[["sum","Amount"]],"filters":[["Won",["true"],"in"]]}`
```yaml
Metric Name:        Won Revenue
Display Label:      Won Opportunities
Description:        Total closed-won revenue. The primary success metric for sales performance.
Formula:            SUM(Amount) WHERE IsWon = true
Format:             Currency, 0 decimal places
```

---

### Metric 4: Lost Revenue
**Replaces:** Step `Amount_3`
**Old SAQL:** `{"measures":[["sum","Amount"]],"filters":[["Won",["false"],"in"],["Closed",["true"],"in"]]}`
```yaml
Metric Name:        Lost Revenue
Display Label:      Lost Opportunities
Description:        Total value of closed-lost opportunities. High lost revenue
                    indicates conversion rate issues or product-market fit gaps.
Formula:            SUM(Amount) WHERE IsWon = false AND IsClosed = true
Format:             Currency, 0 decimal places
```

---

### Metric 5: Cumulative Won Revenue ⚠️ HIGH COMPLEXITY
**Replaces:** Step `CloseDate_Year_Close_5`
**Old SAQL Window Function:**
```
sum(A) over ([..0] partition by all order by ('Close_Date_Year~~~Close_Date_Month'))
```

**Challenge:** Tableau Next Semantic Models use metric definitions, not window functions. Cumulative/running totals need to be handled differently.

**Options:**

**Option A — Table Calculation in Visualization (Recommended)**
- Define `Won Revenue` as a simple metric (SUM Amount WHERE IsWon = true)
- In the Tableau Next Visualization Builder, apply a **Running Total** table calculation on the time axis
- Produces identical visual output with less complexity

**Option B — Pre-computed in Data 360 Transform**
- Add a computed field in the Data 360 transform using SQL window function
- `SUM(Amount) OVER (ORDER BY YEAR(CloseDate), MONTH(CloseDate) ROWS UNBOUNDED PRECEDING)`
- Store as `Cumulative_Won_Amount` field in the DMO
- Reference directly as a measure in the Semantic Model

> ✅ **Recommendation: Option A** — keeps the semantic model clean and leverages Tableau Next's native table calculation capability.

---

### Metric 6: Year Selector (Pillbox replacement)
**Replaces:** Step `CloseDate_Year_1`

In Tableau Next, this becomes a **Date Filter Dimension** in the Semantic Model:
```yaml
Dimension:     Close Date (Year)
Field:         YEAR(CloseDate)
Type:          Date dimension
Filter Type:   Relative or Fixed date range
Tableau Agent: "Show me 2024 performance" → auto-applies this filter
```

---

### Full Semantic Model Definition

```
SEMANTIC MODEL: DTC Sales Analytics
│
├── METRICS
│   ├── Total Pipeline Amount     → SUM(Amount)
│   ├── Open Pipeline Amount      → SUM(Amount) WHERE IsClosed=false
│   ├── Won Revenue               → SUM(Amount) WHERE IsWon=true
│   ├── Lost Revenue              → SUM(Amount) WHERE IsWon=false AND IsClosed=true
│   └── Win Rate %               → (Won Revenue / Total Closed) * 100  [NEW - add this!]
│
├── DIMENSIONS
│   ├── Close Date (Year, Quarter, Month)
│   ├── Stage
│   ├── Product Family
│   ├── Industry
│   ├── Opportunity Owner
│   ├── Billing Country
│   ├── Account Type
│   └── Segment
│
└── DESCRIPTIONS (for Tableau Agent)
    ├── "Total Pipeline: includes open, won, and lost opportunities"
    ├── "Won Revenue: closed-won deals — primary revenue metric"
    ├── "Open Pipeline: active opportunities not yet closed"
    └── "Lost Revenue: closed-lost opportunities — used to calculate win rate"
```

> ✅ **Bonus:** Adding descriptions enables **Tableau Agent** to answer questions like "What's our won revenue this year?" or "Show me pipeline by product family" without any additional configuration.

---

## 🎨 Phase 3: Dashboard Rebuild in Tableau Next

### Widget-by-Widget Migration Map

| CRM Analytics Widget | Type | Tableau Next Equivalent | Effort |
|---------------------|------|------------------------|--------|
| `text_1` — "DTC Sales" title | Text | Text widget (same) | 🟢 5 min |
| `container_7` — Header background | Container | Layout section / styled container | 🟢 10 min |
| `pillbox_1` — Year selector | Pillbox | Date Range Filter control | 🟡 20 min |
| `container_6` — KPI background | Container | Styled KPI card container | 🟢 10 min |
| `text_5` — "TOTAL" label | Text | Text widget or KPI card built-in label | 🟢 5 min |
| `text_2` — "Open Opportunities" | Text | Text widget or KPI card label | 🟢 5 min |
| `text_3` — "Won Opportunities" | Text | Text widget or KPI card label | 🟢 5 min |
| `text_4` — "Lost Opportunities" | Text | Text widget or KPI card label | 🟢 5 min |
| `number_5` — Total KPI | Number | Metric Tile (Total Pipeline Amount) | 🟢 10 min |
| `number_1` — Open KPI | Number | Metric Tile (Open Pipeline Amount) | 🟢 10 min |
| `number_2` — Won KPI | Number | Metric Tile (Won Revenue) | 🟢 10 min |
| `number_3` — Lost KPI | Number | Metric Tile (Lost Revenue) | 🟢 10 min |
| `text_6` — Chart title (desktop) | Text | Text widget | 🟢 5 min |
| `text_7` — Chart title (mobile) | Text | Handled by responsive layout | 🟢 5 min |
| `chart_1` — Cumulative time chart | Time Chart | Line/Area chart + Running Total calc | 🔴 45 min |
| `link_4` — "Opportunity Details >" | Link | Dashboard Navigation Action | 🟡 15 min |
| `link_5` — "Regional Sales >" | Link | Dashboard Navigation Action | 🟡 15 min |

---

### New Dashboard Layout in Tableau Next

```
┌──────────────────────────────────────────────────────────┐
│  DTC Sales                    [Year Filter: 2024 ▼]      │
├──────────────┬──────────────┬──────────────┬─────────────┤
│  💰 TOTAL    │  📂 OPEN     │  ✅ WON      │  ❌ LOST    │
│  $X,XXX,XXX  │  $X,XXX,XXX  │  $X,XXX,XXX  │  $X,XXX,XXX │
│  [▲ +12% YoY]│  [▼ -3% YoY] │  [▲ +18% YoY]│  [▼ -5% YoY]│
├──────────────┴──────────────┴──────────────┴─────────────┤
│  Cumulative Won Revenue Over Time                        │
│  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~                         │
│  /                                                       │
│ /___________________________________________________     │
│  Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  │
├──────────────────────────────────────────────────────────┤
│  [Ask Tableau Agent] → "Compare won vs lost by quarter"  │
└──────────────────────────────────────────────────────────┘
```

> 💡 **New in Tableau Next:** KPI Metric Tiles can show **period-over-period comparison** (e.g., +12% YoY) automatically — not possible in CRM Analytics without manual SAQL.

---

## 🤖 Phase 4: What You GAIN from Tableau Next

These capabilities **don't exist in CRM Analytics** and come free with the migration:

### 1. Tableau Agent — Natural Language Analytics
Users can type questions directly:
```
"What is our won revenue in Q3 compared to Q2?"
"Show me pipeline by product family for accounts in the tech industry"
"Which sales rep has the highest conversion rate this year?"
"What's driving the drop in won revenue in August?"
```
→ Agent reads the Semantic Model and generates answers + visualizations instantly.

### 2. Proactive Metric Alerts (Beta)
```yaml
Alert: Won Revenue drops below $500,000 in any month
Notify: Sales Ops team via Slack
Frequency: Real-time
```
No manual subscription setup — threshold-based intelligence.

### 3. Slack Integration
- Pin "Won Revenue" metric in the #sales-ops Slack channel
- Get weekly automated snapshots delivered to the channel
- Ask data questions directly in Slack: "@Tableau What was Q3 won revenue?"

### 4. Period-over-Period Comparison (built-in)
- Metric tiles automatically show vs. prior period
- No SAQL window function needed

### 5. AI-Generated Insights
- Tableau Agent can surface: "Won revenue is down 8% vs last month, driven by a 23% drop in Enterprise deals"
- Proactive insight delivery without any dashboard building

### 6. Single-Click Salesforce Actions
- From a metric insight → "Create a follow-up task for all open deals closing this month"
- Bridge between analytics and CRM action in one click

---

## ⚠️ Phase 5: What's Hard to Replicate

These CRM Analytics capabilities need special attention during migration:

### Challenge 1: SAQL Window Function → Running Total
**CRM Analytics:** `sum(A) over ([..0] partition by all order by (...))`
**Tableau Next:** Table calculation in Visualization Builder

| | CRM Analytics | Tableau Next |
|--|-------------|-------------|
| **Where defined** | Step (query layer) | Visualization layer |
| **Reusability** | Only in that step | Per-visualization |
| **Complexity** | Advanced SAQL | UI-driven (easier) |
| **Recommendation** | ✅ Easier in Tableau Next |

---

### Challenge 2: Pillbox → Date Filter
**CRM Analytics Pillbox** — discrete year buttons users click.
**Tableau Next** — date range picker or dimension filter list.

| | CRM Analytics Pillbox | Tableau Next Date Filter |
|--|---------------------|------------------------|
| **UX** | Visual pill buttons | Dropdown or date range selector |
| **Default state** | Hardcoded start value | Dynamic (e.g., current year) |
| **Filter scope** | Broadcasts via faceting | Applied as global dashboard filter |
| **Better option?** | ✅ Tableau Next (dynamic defaults, no hardcoding) |

---

### Challenge 3: Cross-Widget Faceting → Filter Interactions
**CRM Analytics:** `broadcastFacet: true` on every step — clicking any chart element filters everything.
**Tableau Next:** Filter interactions are configured per-visualization via the filter panel.

- Tableau Next supports cross-filter interactions but requires explicit configuration
- The rich faceting model of CRM Analytics needs to be mapped to Tableau Next filter bindings
- Effort: ~1 hour to set up equivalent filter interactions

---

### Challenge 4: Container Background Widgets
**CRM Analytics:** `container_6`, `container_7` used as colored background cards.
**Tableau Next:** Layout sections and styled containers handle this natively — cleaner approach.

- No special effort needed — Tableau Next layout is more modern
- Backgrounds can be applied at the section/container level

---

### Challenge 5: No Direct Dashboard Migration Tool
⚠️ **There is no automatic CRM Analytics → Tableau Next migration tool.** Everything must be rebuilt manually.

---

## ⏱️ Effort Estimate by Phase

| Phase | Task | Effort | Complexity |
|-------|------|--------|-----------|
| **Phase 1: Data Layer** | Set up Data 360 Salesforce connector | 1 hr | 🟢 Low |
| | Configure Opportunity + Account + User ingestion | 1 hr | 🟢 Low |
| | Set up daily refresh schedule | 30 min | 🟢 Low |
| | Configure Data 360 row-level security policy | 1 hr | 🟡 Medium |
| **Phase 2: Semantic Model** | Create Semantic Model object | 30 min | 🟢 Low |
| | Define 4 core metrics (Total, Open, Won, Lost) | 1 hr | 🟢 Low |
| | Define dimensions (Stage, Product, Industry, etc.) | 1 hr | 🟡 Medium |
| | Write AI-readable descriptions for each metric | 30 min | 🟢 Low |
| | Add Win Rate % metric (new, bonus) | 30 min | 🟡 Medium |
| | Test Tableau Agent queries against model | 30 min | 🟢 Low |
| **Phase 3: Dashboard** | Create workspace and base dashboard | 30 min | 🟢 Low |
| | Build 4 KPI metric tiles | 1 hr | 🟢 Low |
| | Build date filter control | 30 min | 🟢 Low |
| | Build cumulative time chart + running total | 1.5 hr | 🔴 High |
| | Configure cross-filter interactions | 1 hr | 🟡 Medium |
| | Set up navigation actions (Opp Details, Regional) | 30 min | 🟢 Low |
| | Mobile responsive layout | 30 min | 🟢 Low |
| **Phase 4: Enhancements** | Set up proactive alert on Won Revenue | 30 min | 🟢 Low |
| | Configure Slack metric subscription | 30 min | 🟢 Low |
| **Phase 5: Testing** | Validate metrics vs. CRM Analytics values | 1 hr | 🟡 Medium |
| | User acceptance testing | 1 hr | 🟡 Medium |
| **TOTAL** | | **~14–18 hours** | |

---

## 📋 Pre-Migration Checklist

Before starting, confirm the following:

### Licensing
- [ ] Tableau Next license provisioned for the org
- [ ] Data 360 license provisioned
- [ ] Users have appropriate Tableau Next roles (Viewer/Explorer/Creator)

### Data
- [ ] Salesforce SFDC_LOCAL connector is active (already confirmed in your org ✅)
- [ ] Opportunity, Account, User objects are syncing to Data 360
- [ ] Data 360 is enabled in the org

### Access
- [ ] Admin access to Data 360 setup
- [ ] Creator access in Tableau Next
- [ ] Access to current CRM Analytics dashboard for reference

---

## 🗺️ Step-by-Step Migration Playbook

```
WEEK 1: Data Foundation
━━━━━━━━━━━━━━━━━━━━━
Day 1 (3 hrs):
  ✅ Enable Data 360 + configure Salesforce connector
  ✅ Ingest Opportunity + Account + User objects
  ✅ Set up daily refresh schedule
  ✅ Verify data in Data 360 Data Explorer

Day 2 (4 hrs):
  ✅ Build Semantic Model "DTC Sales Analytics"
  ✅ Define all 4 core metrics + Win Rate bonus metric
  ✅ Add all dimensions with labels and descriptions
  ✅ Apply row-level security policy
  ✅ Test Tableau Agent with 5 sample questions

WEEK 2: Dashboard + Enhancements
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Day 3 (4 hrs):
  ✅ Create "DTC Sales" Workspace in Tableau Next
  ✅ Build dashboard: 4 KPI tiles + date filter
  ✅ Build cumulative time chart with running total
  ✅ Configure cross-filter interactions

Day 4 (3 hrs):
  ✅ Add navigation actions (Opportunity Details, Regional Sales)
  ✅ Set up proactive Won Revenue alert
  ✅ Configure Slack integration + metric subscription
  ✅ Mobile layout review

Day 5 (2 hrs):
  ✅ Side-by-side validation: CRM Analytics vs Tableau Next numbers
  ✅ Fix any discrepancies
  ✅ User acceptance testing with stakeholders
  ✅ Go live — deprecate CRM Analytics dashboard
```

---

## 🆚 Side-by-Side: Before vs. After Migration

| Capability | CRM Analytics (Before) | Tableau Next (After) |
|-----------|----------------------|---------------------|
| **Data freshness** | ❌ 6-year stale CSV | ✅ Live daily refresh |
| **Row-level security** | ❌ None | ✅ Data 360 policy (hierarchy-aware) |
| **Natural language queries** | ❌ Not available | ✅ Tableau Agent (full NL) |
| **Proactive alerts** | ❌ Manual subscriptions only | ✅ Threshold-based smart alerts |
| **Slack integration** | ❌ No native integration | ✅ Native metric sharing in Slack |
| **Period comparison** | ❌ Complex SAQL required | ✅ Built-in YoY/QoQ comparison |
| **Mobile layout** | ✅ Phone layout configured | ✅ Responsive by default |
| **Cumulative chart** | ✅ SAQL window function | ✅ Running total table calculation |
| **Cross-filtering** | ✅ broadcastFacet on all steps | ✅ Filter interactions (explicit setup) |
| **AI-generated insights** | ❌ Not available | ✅ Tableau Agent proactive insights |
| **Salesforce actions** | ❌ Not available | ✅ Single-click Flow triggers |
| **Field utilization** | ❌ 5 of 32 fields used | ✅ All dimensions in Semantic Model |
| **Win Rate metric** | ❌ Not built | ✅ Easy to add in Semantic Model |
| **Dashboard** | ✅ Existing | 🔄 Rebuilt (same look, more power) |

---

## 💰 Migration Cost-Benefit Summary

### Costs
- ~14–18 hours of admin/developer time
- Tableau Next + Data 360 licensing (if not already provisioned)
- Learning curve: team needs Tableau Next training (~4–8 hrs)

### Benefits
- **Immediate:** Live data (vs. 6-year stale)
- **Immediate:** Row-level security (was completely missing)
- **New capability:** Tableau Agent (NL queries on demand)
- **New capability:** Proactive alerts + Slack delivery
- **New capability:** AI-generated business insights
- **Ongoing:** Zero SAQL maintenance (metrics managed in semantic layer)
- **Future-proof:** Foundation for Agentforce integration

---

*Migration plan built from live CRM Analytics org data (API v67.0) combined with Tableau Next & Data 360 expert knowledge. All component mappings based on Salesforce official documentation.*
