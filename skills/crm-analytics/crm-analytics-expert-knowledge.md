# 📊 CRM Analytics Expert Knowledge Base
**Last Updated:** 2026-09-28
**Source:** Salesforce Trailhead Trailmixes + Official Documentation
**Coverage:** 4 Trails | ~32+ hours of content | CRM Analytics & Einstein Discovery

> ⚠️ **Stored Separately** from Tableau Next & Data 360 knowledge base (see: `tableau-next-data360-expert-knowledge.md`)

---

## 🗺️ Table of Contents
1. [Trail Overview Summary](#trail-overview-summary)
2. [Product History & Positioning](#product-history--positioning)
3. [CRM Analytics Architecture & Core Concepts](#crm-analytics-architecture--core-concepts)
4. [Data Integration: Dataflows & Recipes](#data-integration-dataflows--recipes)
5. [Dashboard Building: Basic & Advanced](#dashboard-building-basic--advanced)
6. [CRM Analytics Administration & Security](#crm-analytics-administration--security)
7. [Advanced Editor: SAQL & Bindings](#advanced-editor-saql--bindings)
8. [Embedding CRM Analytics in Salesforce](#embedding-crm-analytics-in-salesforce)
9. [Einstein Discovery: Predictive Analytics](#einstein-discovery-predictive-analytics)
10. [CRM Analytics Subscriptions, Watchlist & Slack](#crm-analytics-subscriptions-watchlist--slack)
11. [Exam Cheat Sheet](#exam-cheat-sheet)
12. [Glossary of Key Terms](#glossary-of-key-terms)

---

## Trail Overview Summary

| Trail | Level | Duration | Points | Key Focus |
|-------|-------|----------|--------|-----------|
| Discover CRM Analytics | Foundational | ~20 mins | 400 | Overview, Subscriptions, Watchlist, Slack |
| Build and Administer CRM Analytics | Intermediate | ~9h 15m | 3,500 | Data Integration, Dashboards, Admin, Embedding |
| Gain Insight with Einstein Discovery | Foundational | ~2h 50m | 1,300 | Predictive Models, Ethical AI |
| Tableau Next Consultant Exam Prep | Foundational | ~7h 55m | 2,700 | CRM Analytics as related product context |

---

## Product History & Positioning

### 🏷️ Name Evolution
```
Wave Analytics (2014)
      ↓
Einstein Analytics (2017)
      ↓
Tableau CRM (2021)
      ↓
CRM Analytics (2022 → Present)
```

### 🎯 What is CRM Analytics?
CRM Analytics is Salesforce's **native, embedded analytics platform** built directly inside the Salesforce ecosystem. Unlike Tableau (which is a standalone BI tool), CRM Analytics is:
- **Natively embedded** in Salesforce Lightning Experience
- **Pre-integrated** with all Salesforce clouds (Sales, Service, Marketing, etc.)
- **AI-powered** via Einstein Discovery for predictive insights
- **Purpose-built** for CRM data and business users

### CRM Analytics vs. Tableau vs. Tableau Next
| Dimension | CRM Analytics | Tableau | Tableau Next |
|-----------|--------------|---------|--------------|
| **Home** | Inside Salesforce | Standalone/Cloud | AI-native platform |
| **Best For** | CRM/Salesforce data | Any data source | Agentic AI analytics |
| **User** | Salesforce users | Data analysts | Business users + AI |
| **AI Feature** | Einstein Discovery | Einstein in Tableau | Tableau Agent |
| **Embedding** | Lightning native | iFrame/SDK | Salesforce flows |
| **Data Prep** | Recipes/Dataflows | Tableau Prep | Data 360 |

### Licensing / Editions
| Edition | Target | Key Capabilities |
|---------|--------|-----------------|
| **CRM Analytics Growth** | SMBs | Core dashboards, limited datasets |
| **CRM Analytics Plus** | Enterprise | Full features, Einstein Discovery, advanced admin |
| **CRM Analytics Platform** | Developers/ISVs | API access, custom app development |

---

## CRM Analytics Architecture & Core Concepts

### 🏗️ Core Building Blocks

```
┌─────────────────────────────────────────┐
│           CRM Analytics App             │  ← Container (like a folder)
│  ┌──────────┐  ┌────────┐  ┌────────┐  │
│  │Dashboards│  │ Lenses │  │Datasets│  │
│  └──────────┘  └────────┘  └────────┘  │
└─────────────────────────────────────────┘
         ↑
   Powered by data from:
   Salesforce Objects + External Sources
```

### Key Components

| Component | Description | Analogy |
|-----------|-------------|---------|
| **App** | Container organizing related analytics assets | Folder/Workbook |
| **Dataset** | Optimized data store for analytics queries | Database table |
| **Lens** | Ad-hoc exploratory visualization of a dataset | Scratch pad |
| **Dashboard** | Curated, interactive multi-widget analytics page | Report/Page |
| **Story** | Einstein Discovery predictive analysis result | ML Report |
| **Dataflow** | Legacy ETL pipeline to build datasets | Old pipeline |
| **Recipe** | Modern visual ETL pipeline to build datasets | New pipeline |

### Analytics Studio
- Central hub for creating and managing all CRM Analytics assets
- Access via App Launcher → **Analytics Studio**
- Sections: Home, Datasets, Lenses, Dashboards, Apps

### Data Manager
- Admin tool for managing data connections, recipes, dataflows, and schedules
- Monitor data sync health and refresh status
- Configure external connectors

---

## Data Integration: Dataflows & Recipes

### 🔄 Two Approaches to Data Preparation

#### 1. Dataflows (Legacy)
- JSON-based ETL configuration
- Processes Salesforce data into datasets
- Steps run in sequence
- Still supported but Recipes are preferred

**Common Dataflow Nodes:**
| Node | Purpose |
|------|---------|
| **sfdcDigest** | Pull data from a Salesforce object |
| **edgemart** | Reference an existing dataset |
| **augment** | Join two datasets (like SQL JOIN) |
| **computeExpression** | Create calculated fields |
| **filter** | Filter rows based on conditions |
| **flatten** | Flatten hierarchical data (e.g., role hierarchy) |
| **delta** | Capture only changed records |
| **register** | Output a dataset |

#### 2. Recipes (Modern — Preferred ✅)
- Visual, drag-and-drop ETL pipeline builder
- More intuitive than JSON dataflows
- Supports both Salesforce and external data
- Has version history and rollback
- Runs on **Data Prep Engine**

**Recipe Node Types:**
| Node | Purpose |
|------|---------|
| **Input** | Connect to Salesforce object or external source |
| **Transform** | Reshape, filter, join, aggregate data |
| **Output** | Write result to a dataset |

**Recipe Transformations:**
- Add column (computed fields)
- Filter rows
- Join (inner, left, right, full outer)
- Union (stack datasets)
- Aggregate (group by + metrics)
- Flatten (hierarchies)
- Bucket (categorize values into groups)
- Type cast (change field data type)

### Data Connector Types
| Connector | Source | Method |
|-----------|--------|--------|
| **Local Salesforce** | Sales/Service/Marketing Cloud objects | Native sync |
| **CSV Upload** | Flat files | Manual or scheduled |
| **Amazon S3** | Cloud files | Scheduled pull |
| **Heroku PostgreSQL** | Heroku DB | Direct query |
| **Google BigQuery** | GCP data warehouse | Live or sync |
| **Snowflake** | Cloud data warehouse | Live or sync |
| **MuleSoft** | Any API-connected system | Event or batch |

### Live vs. Imported Datasets
| | Live Dataset | Imported Dataset |
|--|-------------|-----------------|
| **Data Location** | Stays in source | Copied into CRM Analytics |
| **Freshness** | Real-time | Refreshed on schedule |
| **Performance** | Depends on source | Fast (optimized storage) |
| **Transformation** | Limited | Full recipe/dataflow support |
| **Use Case** | Real-time dashboards | Historical analysis |

### Data Sync Schedule
- Dataflows/Recipes can be scheduled to run: hourly, daily, weekly
- **Incremental sync:** Only pull new/changed records (delta)
- **Full sync:** Re-pull entire dataset each run
- Monitor sync health in **Data Manager**

---

## Dashboard Building: Basic & Advanced

### 🎨 Dashboard Components

#### Widgets (Visual Elements)
| Widget Type | Use Case |
|-------------|---------|
| **Chart** | Bar, line, donut, scatter, waterfall, combo |
| **Table** | Tabular data display with sorting/filtering |
| **Map** | Geographic data visualization |
| **Number** | Single KPI metric display |
| **Gauge** | Progress toward a target |
| **Date Selector** | Date range filter control |
| **List Selector** | Dropdown filter control |
| **Toggle** | Binary filter (On/Off) |
| **Range Selector** | Slider filter control |
| **Container** | Group/layout widget |
| **Text/Image** | Static content widget |

#### Steps (Query Engine)
- Every widget is powered by a **Step** — a query against a dataset
- Steps can be shared between multiple widgets
- Step types: **SAQL**, **SOQL**, **Static** (hardcoded values)
- Steps support **faceting** (cross-widget filtering)

#### Faceting
- When a user clicks on a chart element, all **faceted** steps automatically filter
- Enables interactive, drill-down experiences
- Configured per-step in dashboard settings

### Dashboard Building Best Practices

**Basic Dashboards:**
1. Choose dataset(s) to power the dashboard
2. Add widgets from the widget panel
3. Bind each widget to a step/query
4. Add filter widgets (date selectors, list selectors)
5. Enable faceting for interactivity
6. Set layout for desktop and mobile

**Advanced Dashboard Techniques:**
| Technique | Description |
|-----------|-------------|
| **Bindings** | Dynamic values that change based on user selections |
| **Navigation Actions** | Click to navigate to another dashboard or URL |
| **Salesforce Actions** | Update records, create tasks, log calls directly from dashboard |
| **Conditional Formatting** | Color-code values based on rules |
| **Compact Form** | Mobile-optimized dashboard layout |
| **Init State** | Set default filter/selection state on dashboard load |
| **Container Layouts** | Responsive grid and flex layouts |

### Dashboard Actions
| Action Type | What It Does |
|-------------|-------------|
| **Navigate** | Go to another dashboard, lens, or URL |
| **Salesforce Navigate** | Open a Salesforce record page |
| **Update Record** | Write a value back to a Salesforce field |
| **Create Record** | Create a new Salesforce record |
| **Custom Action** | Trigger a custom flow or Apex |

---

## CRM Analytics Administration & Security

### 👤 Permission Sets

| Permission Set | Access Level |
|---------------|-------------|
| **CRM Analytics Admin** | Full admin: create apps, manage dataflows, configure security |
| **CRM Analytics Manager** | Create/edit apps and dashboards; manage sharing |
| **CRM Analytics Editor** | Create/edit dashboards and lenses |
| **CRM Analytics User** | View dashboards and explore datasets |
| **CRM Analytics Viewer** | View only (no exploration) |

### 🔐 Security Architecture

#### App-Level Sharing
- Control who can view, edit, or manage each App
- Share with users, roles, groups, or public groups
- Sharing settings cascade to assets within the app

#### Row-Level Security (Security Predicates)
- Filter rows in a dataset based on the logged-in user
- Defined using **SAQL predicates**
- Examples:
  - `'OwnerId' == "$User.Id"` → Each user sees only their own records
  - `'Region__c' == "$User.Region__c"` → Users see their region's data
- Applied at dataset level — invisible to end users

#### Dataset Security
- Assign security predicates per dataset
- Predicates reference Salesforce user fields
- Multiple predicates can be combined with AND/OR

#### Field-Level Security
- Control which fields are exposed in datasets
- Remove sensitive fields during recipe/dataflow processing

### Admin Setup Checklist
- [ ] Enable CRM Analytics in Salesforce Setup
- [ ] Assign CRM Analytics permission sets to users
- [ ] Configure Analytics app sharing settings
- [ ] Set up data connectors in Data Manager
- [ ] Schedule dataflows/recipes for regular refresh
- [ ] Apply security predicates to datasets
- [ ] Enable Einstein Discovery (if licensed)
- [ ] Configure subscriptions and alerts

---

## Advanced Editor: SAQL & Bindings

### 📝 SAQL (Salesforce Analytics Query Language)

SAQL is the query language that powers CRM Analytics steps — similar to SQL but designed for analytics datasets.

**Basic SAQL Structure:**
```saql
q = load "dataset_api_name";
q = filter q by date('Date_field_year', 'Date_field_month', 'Date_field_day') in ["current month".."current month"];
q = group q by ('Field1', 'Field2');
q = foreach q generate 'Field1' as 'Field1', 'Field2' as 'Field2', sum('Amount') as 'Revenue';
q = order q by 'Revenue' desc;
q = limit q 10;
```

**Common SAQL Functions:**
| Function | Purpose |
|----------|---------|
| `sum()` | Sum of values |
| `count()` | Count of records |
| `avg()` | Average value |
| `min()` / `max()` | Minimum / Maximum |
| `count_distinct()` | Count unique values |
| `toDate()` | Convert to date |
| `date()` | Reference date field parts |
| `coalesce()` | Return first non-null value |
| `case when` | Conditional logic |
| `matches()` | Regex-based text matching |

**Date Ranges in SAQL:**
```saql
-- Last 3 months
in ["-3M".."-1M"]
-- Current quarter
in ["current quarter".."current quarter"]
-- Year to date
in ["current year".."current month"]
```

### 🔗 Bindings

Bindings make dashboards **dynamic** — they link widget selections to step parameters in real time.

**Types of Bindings:**
| Binding Type | Description |
|-------------|-------------|
| **Result Binding** | Use query result of one step as input to another |
| **Selection Binding** | Use user's selection (click) from one widget to filter another |
| **Column Binding** | Reference a specific column from a step result |
| **Static Binding** | Hardcoded value passed to a step |

**Binding Syntax Example:**
```json
"filter": {
  "operator": "in",
  "subject": {
    "fieldName": "Stage"
  },
  "values": {
    "type": "jsonArrayOfStrings",
    "string": "{{cell(steps.StageFilter.selection, 0, \"Stage\").asString()}}"
  }
}
```

### Advanced Dashboard Editor (JSON)
- Access via the **"</>Edit Source"** button in Dashboard Designer
- Direct JSON editing of dashboard definition
- Necessary for: complex bindings, custom layouts, conditional formatting rules, advanced interactions
- Dashboard JSON structure:
  - `state` → current filter/selection state
  - `steps` → query definitions
  - `widgets` → visual element definitions
  - `layouts` → desktop and mobile layout grid
  - `pages` → multi-page dashboard tabs

---

## Embedding CRM Analytics in Salesforce

### ⚡ Lightning Experience Embedding

#### Method 1: Analytics Tab
- Add CRM Analytics tab to Lightning App
- Users access Analytics Studio directly within Salesforce

#### Method 2: Dashboard Component (Most Common)
- Embed specific dashboard into any Lightning Record Page, App Page, or Home Page
- Via Lightning App Builder → drag "CRM Analytics Dashboard" component
- Configure: choose app, choose dashboard, set filters
- Supports **context passing** (pass current record ID as a filter)

#### Method 3: Visualforce Embedding
- Use `<wave:dashboard>` component in Visualforce pages
- More flexible but requires development effort

#### Method 4: External Embedding (iFrame)
- Embed dashboards in external websites via iFrame
- Requires configuration of trusted origins
- Authentication handled via Salesforce session

### Context Passing (Record-Level Filtering)
- When embedded on a record page, pass the record ID to filter the dashboard
- E.g., embed an Opportunity dashboard on the Opportunity record page
- Config example: `"filter": "{'Opportunity_ID': '{{recordId}}'}"` 

### Compact Form Factor
- Optimized dashboard layout for small embedded spaces
- Fewer widgets, simplified design
- Configured as a separate layout in the dashboard

---

## Einstein Discovery: Predictive Analytics

### 🤖 What is Einstein Discovery?
Einstein Discovery is an **automated machine learning (AutoML)** platform embedded in CRM Analytics that:
- Analyzes historical data to find patterns
- Builds predictive models without requiring data science expertise
- Explains predictions in plain English
- Suggests improvements to change outcomes
- Writes predictions back to Salesforce records

### End-to-End Workflow

```
[Dataset] → [Story/Model] → [Evaluation] → [Deploy] → [Predictions on Records]
                ↑
        (AutoML builds model)
```

#### Step 1: Build Your Dataset
- Create a CRM Analytics dataset with the relevant fields
- Include: outcome variable (what you want to predict) + predictor variables
- Clean data: handle nulls, remove irrelevant fields, ensure sufficient records (min ~400 rows)

#### Step 2: Create a Model (Story)
- Select the dataset and outcome variable
- Choose model type:
  - **Binary Classification** — Yes/No prediction (e.g., Will this deal close?)
  - **Regression** — Numeric prediction (e.g., What will the deal size be?)
  - **Multi-class Classification** — Category prediction (e.g., Which tier will this customer be?)
- Einstein runs AutoML: tests multiple algorithms, selects best model

#### Step 3: Evaluate the Model
| Metric | Description |
|--------|-------------|
| **AUC (Area Under Curve)** | Overall model quality (0.5 = random, 1.0 = perfect) |
| **Accuracy** | % of predictions that are correct |
| **Precision** | Of predicted positives, % actually positive |
| **Recall** | Of actual positives, % correctly predicted |
| **GINI** | Alternative to AUC (0 = random, 1 = perfect) |
| **Top Predictors** | Which fields most influence the outcome |

**Understanding Top Predictors:**
- Shows which variables have the most influence on the outcome
- Can reveal unexpected business insights
- Used to validate model sensibility

#### Step 4: Explore Insights
- **Waterfall Chart:** Shows cumulative contribution of each predictor
- **Breakdown Chart:** Compares outcome across different segments
- **Improvement Suggestions:** "If you change X to Y, the likelihood increases by Z%"

#### Step 5: Deploy the Model
- Deploy model to run predictions on Salesforce records
- **Writeback Predictions:** Score is written as a field on the Salesforce object (e.g., `Churn_Score__c`)
- Schedule regular re-scoring as new data comes in
- Set up **Einstein Prediction Service** for real-time scoring via API

#### Step 6: Predictions in Action
- Predictions appear on record pages (via embedded component)
- Show: predicted outcome + top reasons + suggested improvements
- Users can take action directly from the prediction card

### Einstein Discovery for Reports
- Surface predictions within standard Salesforce Reports
- No dashboard required — adds AI insight column to any report
- Quick Look feature: add Einstein Discovery column to report builder

### Einstein Prediction Service
- REST API for real-time, on-demand predictions
- Call from Apex, Flow, or external systems
- Returns: prediction value + confidence score + contributing factors
- Use cases: real-time scoring during record creation/update

### Ethical AI in Einstein Discovery
| Principle | Description |
|-----------|-------------|
| **Bias Detection** | Review if model disadvantages any group |
| **Explainability** | Every prediction must have human-readable reasons |
| **Accountability** | Document model purpose, training data, and limitations |
| **Transparency** | Users should know when AI is influencing decisions |
| **Model Cards** | Document model performance across demographic segments |

---

## CRM Analytics Subscriptions, Watchlist & Slack

### 📧 CRM Analytics Subscriptions
- **What:** Schedule automated email delivery of dashboard snapshots
- **When:** Set frequency (daily, weekly, monthly) and time
- **Who:** Subscribe yourself or others (admins can subscribe groups)
- **Conditional Subscriptions:** Only send email IF a metric crosses a threshold
  - Example: Send weekly sales report only if pipeline drops below $1M
- **Format:** Email contains: dashboard image + link back to live dashboard

### 👁️ CRM Analytics Watchlist
- **What:** Personal dashboard of key metrics you want to monitor
- **How:** Add any number widget from any dashboard to your Watchlist
- **Access:** From CRM Analytics home → Watchlist tab
- **Use Case:** Quick daily check on your most important KPIs without opening full dashboards
- **Mobile:** Available on Salesforce Mobile app

### 💬 CRM Analytics for Slack
- **Share dashboards** and snapshots directly in Slack channels
- **Ask data questions** using natural language within Slack
- **Subscribe a Slack channel** to receive scheduled dashboard updates
- **Slack Notifications:** Alert a channel when a metric crosses a threshold
- **Setup:** Requires CRM Analytics for Slack app installation + admin configuration

---

## Exam Cheat Sheet

### ✅ Must-Know Concepts

#### Architecture
- **App** = container → holds Datasets, Lenses, Dashboards
- **Dataset** = optimized data store (not a raw database table)
- **Lens** = ad-hoc exploration view of a single dataset
- **Dashboard** = curated multi-widget analytics page
- **Story** = Einstein Discovery predictive model result

#### Data Integration
- **Recipes** = modern visual ETL (preferred over Dataflows)
- **Dataflows** = legacy JSON-based ETL (still supported)
- **sfdcDigest** = dataflow node to pull Salesforce object data
- **augment** = dataflow node to JOIN two datasets
- **Live Dataset** = real-time, stays in source
- **Imported Dataset** = copied into CRM Analytics, refreshed on schedule

#### SAQL Key Facts
- SAQL = Salesforce Analytics Query Language
- Powers all steps behind dashboard widgets
- `load` → `filter` → `group` → `foreach` → `order` → `limit`
- Date ranges: `"current month"`, `"-3M"`, `"current year"`

#### Security
- **Security Predicate** = row-level filter on a dataset
- Predicates reference Salesforce User fields (e.g., `$User.Id`)
- Permission sets: Admin → Manager → Editor → User → Viewer (decreasing access)
- App sharing = who can see the app; dataset security = what rows they see

#### Einstein Discovery
- **Binary Classification** → Yes/No outcomes
- **Regression** → Numeric predictions
- **AUC** = primary model quality metric (closer to 1.0 = better)
- **Top Predictors** = fields with highest influence on outcome
- **Writeback** = scores written back to Salesforce record fields
- **Ethical AI** = explainability, bias detection, accountability

#### Embedding
- **Dashboard Component** = most common embedding method in Lightning
- **Context Passing** = filter dashboard by current record ID
- **Compact Form** = mobile/small-space optimized layout

### ❌ Common Exam Traps
- **Lens ≠ Dashboard**: A Lens is exploratory (single dataset), a Dashboard is curated (multi-widget)
- **Recipes ≠ Dataflows**: Recipes are visual/modern; Dataflows are JSON/legacy — don't mix them up
- **Security Predicates are invisible to users** — they silently filter data at the dataset level
- **AUC of 0.5 = random guessing** (NOT a good model!) — higher is better
- **Live datasets don't support all recipe transformations** — some transforms require imported datasets
- **Subscriptions ≠ Watchlist**: Subscriptions = scheduled email delivery; Watchlist = personal on-screen KPI monitor
- **CRM Analytics ≠ Reports & Dashboards**: Standard Salesforce Reports ≠ CRM Analytics dashboards — they are separate systems
- **Einstein Discovery needs minimum ~400 clean rows** to build a reliable model

---

## Glossary of Key Terms

| Term | Definition |
|------|------------|
| **Analytics Studio** | Central hub for creating and managing CRM Analytics assets |
| **App** | Container organizing dashboards, lenses, and datasets |
| **AUC** | Area Under Curve — primary metric for Einstein Discovery model quality |
| **augment** | Dataflow node that joins two datasets |
| **Binding** | Dynamic link between a user selection and a step query |
| **Compact Form** | Mobile-optimized dashboard layout |
| **Conditional Subscription** | Email subscription that only sends when a metric condition is met |
| **CRM Analytics** | Salesforce's native embedded analytics platform (formerly Wave/Einstein Analytics/Tableau CRM) |
| **Dashboard** | Curated, multi-widget interactive analytics page |
| **Data Manager** | Admin tool for managing connectors, recipes, dataflows, and schedules |
| **Dataflow** | Legacy JSON-based ETL pipeline for building datasets |
| **Dataset** | Optimized data store that powers CRM Analytics queries |
| **Delta** | Incremental data sync — only new or changed records |
| **Einstein Discovery** | AutoML platform within CRM Analytics for predictive models |
| **Einstein Prediction Service** | REST API for real-time on-demand predictions |
| **Faceting** | Cross-widget filtering — clicking one chart filters others |
| **Flatten** | Dataflow/recipe node that expands hierarchical data (e.g., role hierarchy) |
| **Lens** | Ad-hoc exploratory view of a single dataset |
| **Live Dataset** | Dataset that queries source in real-time without copying data |
| **Recipe** | Modern visual ETL pipeline builder (preferred over dataflows) |
| **SAQL** | Salesforce Analytics Query Language — powers dashboard step queries |
| **Security Predicate** | Row-level filter applied to a dataset based on the current user |
| **sfdcDigest** | Dataflow node that pulls data from a Salesforce object |
| **Step** | Query definition that powers a dashboard widget |
| **Story** | Einstein Discovery predictive model and its insights |
| **Watchlist** | Personal monitor of key metric widgets from any dashboard |
| **Writeback** | Einstein Discovery feature that writes prediction scores to Salesforce record fields |

---

*Document generated from Salesforce Trailhead trails: Discover CRM Analytics, Build and Administer CRM Analytics, Gain Insight with Einstein Discovery, and Tableau Next Consultant Exam Prep. For exam preparation, always cross-reference with the official Salesforce Exam Guide.*
