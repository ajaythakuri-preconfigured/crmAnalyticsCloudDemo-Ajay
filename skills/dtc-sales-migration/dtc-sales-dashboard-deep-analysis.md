# 🔍 Dashboard Deep Analysis Report
## Dashboard: "DTC Sales"
**Dashboard ID:** 0FKhg0000001YoCGAU
**API Name:** DTC_Sales_SAMPLE
**App:** My DTC Sales
**URL:** https://preconfigured4-dev-ed.develop.lightning.force.com/analytics/dashboard/0FKhg0000001YoCGAU
**Created By:** Anchit | **Created:** 2026-09-11 | **Last Modified:** 2026-09-11
**Reviewed By:** Claude AI (CRM Analytics Expert)
**Report Date:** 2026-09-28

---

## 📋 Dashboard Overview

| Property | Value |
|----------|-------|
| **Label** | DTC Sales |
| **API Name** | DTC_Sales_SAMPLE |
| **App** | My DTC Sales |
| **Total Widgets** | 17 |
| **Total Steps (Queries)** | 6 |
| **Datasets Used** | 1 — DTC Opportunity (DTC_Opportunity_SAMPLE) |
| **Pages** | 1 |
| **Layouts** | 2 — Desktop (Default) + Phone (Mobile) ✅ |
| **Mobile Enabled** | ✅ Yes |
| **Global Filters** | ❌ None |
| **layoutAutoSync** | ❌ Not enabled |
| **Preview Thumbnail** | ✅ Exists |
| **Sharing** | Public (visibility: All) |

---

## 🗄️ Dataset Used

### DTC Opportunity (`DTC_Opportunity_SAMPLE`)
**Dataset ID:** 0Fbhg0000001n71CAA

**Key Fields Used Across All Queries:**
| Field Name | Type | Purpose |
|-----------|------|---------|
| `Amount` | Measure | Revenue value — aggregated as SUM throughout |
| `Won` | Dimension/Boolean | Filter for won opportunities (`true`/`false`) |
| `Closed` | Dimension/Boolean | Filter for closed opportunities (`true`/`false`) |
| `Close_Date_Year` | Dimension/Date Part | Yearly grouping for filter + trending |
| `Close_Date_Month` | Dimension/Date Part | Monthly grouping for time chart |

**Business Logic Decoded from Field Names:**
- `Won = true` → Closed-Won opportunities
- `Won = false AND Closed = true` → Closed-Lost opportunities
- `Closed = false` → Open (active pipeline) opportunities
- `Closed = true` (no Won filter) → All closed (won + lost combined)

---

## ⚙️ Step-by-Step Query Analysis (6 Steps)

### Step 1: `Amount_4` — Total Pipeline
**Widget:** `number_5` → "TOTAL" KPI card
**Type:** aggregateflex | **Visualization:** Number
```json
{
  "measures": [["sum", "Amount"]],
  "filters": []
}
```
**What it does:** Sums the `Amount` field across ALL opportunities with no filter — giving the absolute total pipeline value regardless of stage.
**Interactivity:** `broadcastFacet: true`, `useGlobal: true` — responds to the year pillbox filter.
⚠️ **Issue:** No filter means this includes ALL statuses (open, won, lost). If the pillbox year filter is active, it filters by year — but without a year filter, this is a grand total of everything. The label "TOTAL" could be ambiguous to users.

---

### Step 2: `Amount_1` — Open Opportunities
**Widget:** `number_1` → "Open Opportunities" KPI card
**Type:** aggregateflex | **Visualization:** Number
```json
{
  "measures": [["sum", "Amount"]],
  "filters": [["Closed", ["false"], "in"]]
}
```
**What it does:** Sums Amount where `Closed = false` — active pipeline that hasn't closed yet.
**Interactivity:** Fully faceted and responsive to global year filter.
✅ Clean, correct filter logic.

---

### Step 3: `Amount_2` — Won Opportunities
**Widget:** `number_2` → "Won Opportunities" KPI card
**Type:** aggregateflex | **Visualization:** Number
```json
{
  "measures": [["sum", "Amount"]],
  "filters": [["Won", ["true"], "in"]]
}
```
**What it does:** Sums Amount where `Won = true` — all closed-won revenue.
**Interactivity:** Fully faceted.
✅ Correct and clean.

---

### Step 4: `Amount_3` — Lost Opportunities
**Widget:** `number_3` → "Lost Opportunities" KPI card
**Type:** aggregateflex | **Visualization:** Number
```json
{
  "measures": [["sum", "Amount"]],
  "filters": [
    ["Won", ["false"], "in"],
    ["Closed", ["true"], "in"]
  ]
}
```
**What it does:** Sums Amount where `Won = false AND Closed = true` — all closed-lost opportunities.
**Interactivity:** Fully faceted.
✅ Correctly uses compound filter (AND logic) for closed-lost.

---

### Step 5: `CloseDate_Year_1` — Year Pillbox Selector
**Widget:** `pillbox_1` → Year filter selector
**Type:** aggregateflex | **Visualization:** Pillbox selector
```json
{
  "measures": [["sum", "Amount"]],
  "groups": ["Close_Date_Year"],
  "filters": [["Closed", ["true"], "in"]],
  "start": "[2016]"
}
```
**What it does:** Groups closed opportunities by `Close_Date_Year` — each year becomes a selectable pill. When a user clicks a year, it broadcasts a facet filter to ALL other steps.
**Default state:** Starts with year 2016 pre-selected (`start: "[2016]"`).
⚠️ **Issue:** The filter is `Closed = true` only — this means open opportunities won't appear as a year option even if they have a future close date. This may cause confusion when "TOTAL" includes open opps but years shown come from only closed opps.
⚠️ **Issue:** Default start year hardcoded to `[2016]` — if data has evolved past 2016, this should be updated to be dynamic or set to the most recent year.

---

### Step 6: `CloseDate_Year_Close_5` — Cumulative Won Revenue Time Chart
**Widget:** `chart_1` → Time chart (area)
**Type:** aggregateflex | **Visualization:** `time` (time series)

**Advanced SAQL — Two-Column Comparison Table:**
```json
{
  "measures": [["sum", "Amount", "B"]],
  "groups": [["Close_Date_Year", "Close_Date_Month"]],
  "columns": [
    {
      "query": {
        "measures": [["sum", "Amount"]],
        "groups": [["Close_Date_Year", "Close_Date_Month"]],
        "filters": [["Won", ["true"], "in"]]
      }
    },
    {
      "query": {
        "measures": [["sum", "Amount"]],
        "groups": [["Close_Date_Year", "Close_Date_Month"]],
        "formula": "sum(A) over ([..0] partition by all order by ('Close_Date_Year~~~Close_Date_Month'))",
        "filters": [["Won", ["true"], "in"]]
      }
    }
  ]
}
```
**What it does (step by step):**
- **Column A:** Monthly sum of won `Amount` grouped by Year + Month → raw monthly won revenue
- **Column B (displayed):** A running/cumulative SUM using the window function `sum(A) over ([..0] partition by all order by Close_Date_Year~~~Close_Date_Month)` → this computes the running total from the very first month up to the current month, creating a constantly growing line
- **Result:** The chart shows cumulative won revenue growing over time (an always-ascending area chart if business is growing)

**Window Function Breakdown:**
| Part | Meaning |
|------|---------|
| `sum(A)` | Sum of column A (monthly won revenue) |
| `over ([..0])` | From the very beginning of the dataset up to current row |
| `partition by all` | No segmentation — single cumulative total |
| `order by ('Close_Date_Year~~~Close_Date_Month')` | Sorted chronologically by year then month |

✅ **This is an advanced SAQL technique** — excellent use of window functions for cumulative analytics.

---

## 🎨 Widget Inventory (17 Widgets)

### Widget Type Breakdown
| Type | Count | Widgets |
|------|-------|---------|
| **text** | 7 | text_1, text_2, text_3, text_4, text_5, text_6, text_7 |
| **number** | 4 | number_1, number_2, number_3, number_5 |
| **container** | 2 | container_6, container_7 |
| **link** | 2 | link_4, link_5 |
| **pillbox** | 1 | pillbox_1 |
| **chart** | 1 | chart_1 |

### Detailed Widget Map

| Widget | Type | Label/Content | Powered By | Color | Notes |
|--------|------|--------------|-----------|-------|-------|
| `text_1` | Text | "DTC Sales" (20px, #44A2F5) | — | Blue | Dashboard title — desktop only |
| `container_7` | Container | Background header bar | — | — | Header background |
| `pillbox_1` | Pillbox | Year selector | CloseDate_Year_1 | #677A97 selected | Global filter control |
| `container_6` | Container | KPI card background | — | — | Styled card background for KPIs |
| `text_5` | Text | "TOTAL" (18px, #FFFFFF) | — | White | KPI label |
| `text_2` | Text | "Open Opportunities" (18px, #FFFFFF) | — | White | KPI label |
| `text_3` | Text | "Won Opportunities" (18px, #FFFFFF) | — | White | KPI label |
| `text_4` | Text | "Lost Opportunties" (18px, #FFFFFF) | — | White | KPI label — **typo!** |
| `number_5` | Number | Total Amount | Amount_4 | #FFFFFF | TOTAL KPI — 48px |
| `number_1` | Number | Open Amount | Amount_1 | #FFFFFF | Open KPI — 48px |
| `number_2` | Number | Won Amount | Amount_2 | #FFFFFF | Won KPI — 48px |
| `number_3` | Number | Lost Amount | Amount_3 | #FFFFFF | Lost KPI — 48px |
| `text_6` | Text | "Cumulative Won Opportunities Over Time" | — | #7D98B3 | Chart title — desktop only |
| `text_7` | Text | "Cumulative Won Opportunities Over Time" | — | #7D98B3 | Chart title — mobile only |
| `chart_1` | Chart | Time area chart (cumulative won) | CloseDate_Year_Close_5 | Light theme | Main visualization |
| `link_4` | Link | "Opportunity Details >" | → Opportunity_Details dashboard | #5C7A99 | Navigation — mobile only |
| `link_5` | Link | "Regional Sales >" | → Regional_Sales_SAMPLE dashboard | #5C7A99 | Navigation — mobile only |

---

## 📐 Layout Analysis

### Desktop Layout (Default — 18 Columns, Max 1440px)

```
Col:  1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16
     ┌───────────────────────────────────────────────────────────────────────────┐
R0   │ [text_1: DTC Sales]          [container_7 bg]          [pillbox_1: Year] │
     ├────────────────────┬────────────────────┬───────────────────┬────────────┤
R1   │   text_5: TOTAL    │  text_2: Open Opp  │  text_3: Won Opp  │text_4: Lost│
     │  [container_6: KPI background card]                                      │
R2-3 │   number_5 (Total) │  number_1 (Open)   │  number_2 (Won)   │number_3(L) │
     ├───────────────────────────────────────────────────────────────────────────┤
R4   │   (empty row)                                                             │
     ├───────────────────────────────────────────────────────────────────────────┤
R5   │   text_6: "Cumulative Won Opportunities Over Time"                        │
     ├───────────────────────────────────────────────────────────────────────────┤
R6-11│                     chart_1: TIME CHART (Area)                            │
     └───────────────────────────────────────────────────────────────────────────┘
```

### Phone Layout (2 Columns)
```
Col: 0 (left)              1 (right)
     ┌──────────────────────────────────┐
R0   │    pillbox_1: Year Filter        │
     ├────────────────┬─────────────────┤
R1   │ text_5: TOTAL  │ text_2: Open    │
R2-3 │ number_5(Total)│ number_1(Open)  │
R4   │ text_3: Won    │ text_4: Lost    │
R5-6 │ number_2(Won)  │ number_3(Lost)  │
     │ [container_6 background]         │
R7   │ text_7: Chart Title              │
R8-13│ chart_1: TIME CHART              │
R14  │ link_4: Opp Det>│ link_5: RegSal>│
     └──────────────────────────────────┘
```

**Mobile Layout Observations:**
- ✅ Fully configured separate phone layout
- ✅ Navigation links appear in mobile (not cluttering desktop)
- ✅ KPI cards stack logically (Total/Open top, Won/Lost bottom)
- ⚠️ Dashboard title (`text_1`) is **missing from phone layout** — no dashboard title on mobile

---

## 🔁 Interactivity & Faceting Analysis

### Facet Map
```
[pillbox_1: Year Selector]
        │  broadcastFacet: true
        │  useGlobal: true
        ▼ (filters ALL steps when user clicks a year)
┌──────────────────────────────────────────────────┐
│  Amount_4 → number_5 (TOTAL)                     │
│  Amount_1 → number_1 (Open Opp)                  │
│  Amount_2 → number_2 (Won Opp)                   │
│  Amount_3 → number_3 (Lost Opp)                  │
│  CloseDate_Year_Close_5 → chart_1 (Time Chart)   │
└──────────────────────────────────────────────────┘
```

| Setting | Value | Impact |
|---------|-------|--------|
| `broadcastFacet` | `true` on ALL steps | Every step can broadcast clicks as filters |
| `receiveFacetSource.mode` | `all` on ALL steps | Every step receives facets from ALL other steps |
| `useGlobal` | `true` on ALL steps | Steps respond to global dashboard context |
| `useExternalFilters` | `true` on ALL steps | Supports embedding with external filter passing |
| **Global Filters** | ❌ None defined | No pre-set persistent filters |

**Result:** Clicking any element on the dashboard (including the year pillbox) will cross-filter all other widgets. This is full cross-dashboard faceting — a strong interactive experience.

---

## 🧭 Navigation & Actions

| Widget | Destination | Type | State Passed? |
|--------|------------|------|--------------|
| `link_4` | Opportunity Details (`Opportunity_Details`) | Dashboard → Dashboard | ❌ No (`includeState: false`) |
| `link_5` | Regional Sales (`Regional_Sales_SAMPLE`) | Dashboard → Dashboard | ❌ No (`includeState: false`) |

⚠️ Both navigation links use `includeState: false` — meaning when a user clicks "Opportunity Details >" after selecting a specific year in the pillbox, the year filter is **NOT carried over** to the destination dashboard. The user loses their filter context on navigation.

---

## 🔍 Findings Summary

### 🔴 Issues (Fix Recommended)

| # | Issue | Location | Recommendation |
|---|-------|----------|---------------|
| 1 | **Typo in widget label** | `text_4` | "Lost **Opportunties**" → fix to "Lost **Opportunities**" |
| 2 | **Navigation links don't pass filter state** | `link_4`, `link_5` | Set `includeState: true` so year filter context carries to destination dashboard |
| 3 | **Dashboard title missing on mobile** | Phone layout | Add `text_1` (or a new text widget) to the phone layout Row 0 |
| 4 | **Year pillbox hardcoded start at 2016** | `CloseDate_Year_1` | Update `start` dynamically or remove static default if data has newer years |
| 5 | **TOTAL KPI includes Open opportunities** | `Amount_4` | Clarify label: rename "TOTAL" to "Total Pipeline" or add a tooltip so users understand it includes open opps |
| 6 | **Non-descriptive step names** | All 6 steps | Rename `Amount_1`, `Amount_2`, etc. to `Open_Pipeline`, `Won_Revenue`, `Lost_Revenue`, `Total_Pipeline` — makes maintenance far easier |

### 🟡 Warnings

| # | Warning | Details |
|---|---------|---------|
| 7 | **layoutAutoSync not enabled** | Dashboard was built without `layoutAutoSync: true`. As the canvas evolves, desktop and phone layouts may drift out of sync. |
| 8 | **Empty number widget titles** | All 4 number widgets have `title: ""` — they rely on separate text widgets above them for labels. If widgets are moved, labels and numbers could get misaligned. Consider using the widget's built-in title instead. |
| 9 | **2 container widgets have empty `documentId`** | `container_6` and `container_7` are used purely as color background overlays — they work but can't hold images. This is a common pattern but fragile when layout is rearranged. |
| 10 | **Only 1 dataset** | The entire dashboard is powered by a single sample dataset (`DTC_Opportunity_SAMPLE`). No user, account, or product context is available. |

### 🟢 Strengths

| # | Strength | Why It Matters |
|---|----------|---------------|
| 1 | ✅ **Both desktop and phone layouts configured** | Great mobile coverage — uncommon in many orgs |
| 2 | ✅ **Full faceting on all 6 steps** | Clicking any element cross-filters everything — excellent UX |
| 3 | ✅ **Advanced SAQL window function** for cumulative chart | Running total using `sum(A) over ([..0] partition by all order by ...)` is expert-level SAQL |
| 4 | ✅ **Clean KPI card design** with container backgrounds | Professional look using container widgets as styled color backgrounds |
| 5 | ✅ **Logical metric grouping** (Total → Open → Won → Lost) | Covers the full opportunity lifecycle at a glance |
| 6 | ✅ **Mobile navigation links** appear only on phone layout | Smart UX — navigation links on mobile where they're needed, not cluttering desktop |
| 7 | ✅ **Fill area time chart** with cumulative data | Visually compelling — makes growth trend obvious |
| 8 | ✅ **`exploreLink: true`** on all number widgets | Users can drill into the data from each KPI — good self-service capability |
| 9 | ✅ **Preview thumbnail** exists | Shows up correctly in Analytics Studio gallery |

---

## 📊 Dashboard Maturity Score

| Dimension | Score | Rationale |
|-----------|-------|-----------|
| **Query Design** | 8/10 | Advanced SAQL window function; could improve step naming |
| **Widget Design** | 7/10 | Good layout; typo and empty titles reduce score |
| **Interactivity** | 9/10 | Full faceting on all steps — excellent |
| **Mobile Design** | 8/10 | Phone layout configured; missing title on mobile |
| **Navigation** | 5/10 | Links exist but don't pass filter state |
| **Data Coverage** | 5/10 | Single sample dataset limits depth |
| **Naming Conventions** | 4/10 | Step names (Amount_1, Amount_2) are not descriptive |
| **Best Practices** | 6/10 | Missing layoutAutoSync, hardcoded year default |
| **Overall** | **7/10** | Solid intermediate dashboard with expert SAQL — fixable issues |

---

## 🛠️ Quick Win Fixes (Priority Order)

```
1. Fix typo: "Lost Opportunties" → "Lost Opportunities" in text_4
2. Set includeState: true on link_4 and link_5 (navigation links)
3. Add text_1 (title) to Phone layout Row 0
4. Rename steps: Amount_1→Open_Pipeline, Amount_2→Won_Revenue, Amount_3→Lost_Revenue, Amount_4→Total_Pipeline
5. Update pillbox default year (2016) to current/latest year
6. Enable layoutAutoSync: true on the dashboard
7. Add number widget titles instead of relying on separate text widgets
```

---

*Report generated via live CRM Analytics REST API (v67.0). Full dashboard JSON parsed and analyzed using Claude AI CRM Analytics Expert knowledge base.*
