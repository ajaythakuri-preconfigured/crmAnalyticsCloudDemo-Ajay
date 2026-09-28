# 📊 Tableau Next & Data Cloud 360 Expert Knowledge Base
**Last Updated:** 2026-09-25  
**Source:** Salesforce Trailhead Trailmixes  
**Coverage:** 3 Trails | ~32 hours of content | Tableau Next Consultant + Data 360 Consultant Exam Prep

---

## 🗺️ Table of Contents
1. [Trail Overview Summary](#trail-overview-summary)
2. [Tableau Foundations: See, Understand, and Act on Data](#trail-1-tableau-foundations)
3. [Tableau Next Consultant Exam Prep](#trail-2-tableau-next-consultant-exam-prep)
4. [Data Cloud 360 Consultant Exam Prep](#trail-3-data-cloud-360-consultant-exam-prep)
5. [Unified Concept Map: How It All Connects](#unified-concept-map)
6. [Exam Cheat Sheet: Tableau Next Consultant](#exam-cheat-sheet-tableau-next)
7. [Exam Cheat Sheet: Data 360 Consultant](#exam-cheat-sheet-data-360)
8. [Glossary of Key Terms](#glossary)

---

## Trail Overview Summary

| Trail | Level | Duration | Points | Certification |
|-------|-------|----------|--------|---------------|
| Tableau: See, Understand, and Act on Data | Foundational | ~10h 43m | 6,900 | Foundation |
| Prepare for Tableau Next Consultant Exam | Foundational/Admin | ~7h 55m | 2,700 | Tableau Next Consultant |
| Prepare for Data 360 Consultant Exam | Intermediate/Admin | ~13h 40m | 10,000 | Data 360 Consultant |

---

## Trail 1: Tableau Foundations

### 🎯 Purpose
Build foundational understanding of the entire Tableau product family and how AI-powered analytics works.

---

### Module 1: Find the Right Tableau Offering
**Key Concepts:**
- Tableau is an AI-integrated analytics platform connected to Salesforce
- Multiple product offerings exist for different use cases

**Tableau Product Portfolio:**
| Product | Purpose |
|---------|---------|
| **Tableau Desktop** | Local, powerful authoring and data analysis |
| **Tableau Cloud** | Cloud-based hosting, sharing, and collaboration |
| **Tableau Server** | On-premises enterprise hosting |
| **Tableau Prep Builder** | Data cleaning, shaping, and combination |
| **Tableau Next** | AI-first analytics with agentic capabilities |

---

### Module 2: Tableau Prep — Prepare Data for Analysis
**Core Capabilities:**
- **Combine** data from multiple sources
- **Shape** data structure (pivot, join, union)
- **Clean** data (remove nulls, fix types, rename fields)
- **Output** data to Tableau or other formats

**Key Steps in Tableau Prep Workflow:**
1. Connect to a data source
2. Build a flow (visual pipeline)
3. Apply cleaning steps
4. Output to Tableau Data Source (.tds/.tdsx) or database

---

### Module 3: Visualize Data with Tableau Cloud
**Core Concepts:**
- **Site:** A Tableau Cloud environment/tenant for an organization
- **Web Authoring:** Build and edit views directly in the browser (no Desktop required)
- **Workbook:** Container for one or more sheets/dashboards
- **View:** A single visualization or dashboard

**Key Capabilities:**
- Connect to live or extracted data sources
- Create and save custom views
- Share visualizations with comments
- Collaborate with team members in real-time

---

### Module 4: Tableau Server
**Core Concepts:**
- On-premises alternative to Tableau Cloud
- Supports both admin and end-user roles
- Same core publishing and sharing functionality as Cloud

**Admin vs End User Roles:**
| Role | Capabilities |
|------|-------------|
| **Admin** | Manage users, sites, permissions, data sources |
| **End User** | View, interact with, and comment on published content |

---

### Module 5: Tableau Next — AI-First Analytics ⭐ EXAM CRITICAL
**What is Tableau Next?**
Tableau Next is an AI-native analytics platform built on Agentforce. It enables agentic AI to analyze, act, and collaborate with data insights across workflows.

**Key Features:**
| Feature | Description |
|---------|-------------|
| **Tableau Agent** | Conversational AI for natural language data analysis |
| **Workspaces** | Collaborative environments for multi-source analysis |
| **Metrics** | KPI monitoring and proactive alerting |
| **Tableau Next in Slack** | Share insights and trigger workflows directly in Slack |
| **Personal Org (Beta)** | Self-service sandbox analysis environment |

**Agentic AI Capabilities:**
- Analyze data using natural language prompts
- Deliver proactive data-driven insights
- Trigger Salesforce Flows via single-click actions
- Surface alerts when metrics cross thresholds

---

### Module 6: Tableau Desktop
**Workspace Components:**
- **Shelves:** Rows, Columns, Pages, Filters, Marks
- **Cards:** Marks Card, Filters Card, Pages Card
- **Data Pane:** Dimensions vs. Measures (Blue = Discrete, Green = Continuous)
- **Show Me Panel:** Recommends chart types

**Data Connection Types:**
- **Live Connection:** Real-time query to source system
- **Extract (.hyper):** Snapshot of data stored locally for performance

**Key Chart Types:**
- Bar Charts (comparing categories)
- Line Charts (trends over time)
- Scatter Plots (correlation between measures)
- Maps (geographic data)
- Pie/Donut Charts (part-to-whole)
- Bullet Graphs (measure vs. target)
- Stacked Bar Charts (composition)

**Key Formulas/Concepts:**
- **Calculated Fields:** Custom computations using Tableau formula syntax
- **LOD Expressions:** Level of Detail — FIXED, INCLUDE, EXCLUDE
- **Table Calculations:** WINDOW_SUM, RUNNING_TOTAL, RANK, etc.

---

### Module 7: Data Modeling and Measure Comparison
**The Tableau Data Model:**
- Tableau uses a **logical layer** (relationships) and **physical layer** (joins/unions)
- **Relationships** are the preferred way to connect tables (context-aware, no pre-join)
- **Joins** physically combine tables before analysis

**Relationship vs. Join:**
| | Relationship | Join |
|--|-------------|------|
| When combined | At query time | Before analysis |
| Null handling | Preserves nulls | May drop nulls |
| Granularity | Preserves per-table | May duplicate rows |
| Recommended | ✅ Yes (default) | ⚠️ Specific use cases |

**Metadata Management:**
- Field descriptions, aliases, default formatting
- Data source certification (mark trusted sources)
- Column-level descriptions for self-service analytics

**Charts for Comparing Measures:**
- **Bar Charts:** Simple categorical comparison
- **Stacked Bar Charts:** Compare parts of a whole across categories
- **Bullet Graphs:** Compare a measure against a reference/target value

---

## Trail 2: Tableau Next Consultant Exam Prep

### 🎯 Exam Focus Areas
1. Data Strategy & Governance
2. Technical Setup (Data 360 + Tableau Next)
3. Semantic Modeling
4. Security & Access Control
5. Analytical Deployment (Workspaces, Assets)
6. User Enablement (Dashboards, Agents)
7. Cross-Platform Integration (Slack, APIs)

---

### Section 1: Develop a Data Strategy ⭐
**Core Concept:** Data 360 and Tableau Next work as a **connected ecosystem** — Data 360 provides the unified data foundation, Tableau Next provides AI-powered analytics on top of it.

**Data Strategy Steps:**
1. Define organizational roles (data owners, stewards, consumers)
2. Review and inventory existing data assets
3. Plan data ingestion priorities
4. Establish governance frameworks

**Key Roles:**
| Role | Responsibility |
|------|---------------|
| **Data Owner** | Accountable for data quality and access decisions |
| **Data Steward** | Manages day-to-day data governance tasks |
| **Data Consumer** | Uses analytics for business decisions |

**Governance Framework Components:**
- Data classification (sensitivity tiers)
- Access policies
- Retention and archiving rules
- Audit and compliance tracking

---

### Section 2: Build the Data 360 Foundation ⭐
**Setup Steps:**
1. Enable Data 360 in Salesforce org
2. Configure data source connectors
3. Set up data ingestion pipelines
4. Configure identity resolution
5. Enable real-time data access

**Data 360 Connectors:**
- Salesforce CRM (native)
- Marketing Cloud
- External databases (JDBC/ODBC)
- Cloud storage (S3, Azure Blob, GCS)
- Zero Copy partners (Snowflake, Databricks)

**Identity Resolution:**
- Maps individuals across multiple data sources using match rules
- Creates **Unified Individual** records
- Uses probabilistic and deterministic matching
- **Data Lake Objects (DLOs):** Where raw ingested data lives
- **Data Model Objects (DMOs):** Harmonized, structured data objects

---

### Section 3: Create Semantic Models ⭐⭐ HIGHLY EXAM RELEVANT
**What is a Semantic Model?**
A semantic model is a **business-friendly layer** on top of raw data that:
- Unifies data into a single source of truth
- Adds business context (friendly names, descriptions, relationships)
- Enables Tableau Agent to answer business questions accurately
- Provides metadata precision for AI agents

**Key Semantic Model Concepts:**
| Concept | Description |
|---------|-------------|
| **Metrics** | Pre-defined business calculations (e.g., Revenue, Churn Rate) |
| **Dimensions** | Categorical attributes (e.g., Region, Product Category) |
| **Relationships** | How semantic objects connect to each other |
| **Descriptions** | AI-generated or manual metadata for agent comprehension |

**Semantic Model Optimization (Beta):**
- AI can suggest improvements to semantic model structure
- Marketplace templates accelerate model development
- AI-generated field descriptions improve agent accuracy

**Tableau Semantics Badge Key Topics:**
- Building semantic models
- Adding metrics and dimensions
- Configuring relationships between objects
- Writing clear descriptions for AI agents

---

### Section 4: Configure Tableau Next ⭐
**Setup Checklist:**
- [ ] Configure workspace preferences
- [ ] Set up user and group management
- [ ] Assign permission sets and licenses
- [ ] Define security control model
- [ ] Plan Tableau Agent implementation
- [ ] Configure Personal Org (Beta) if needed

**Security Model:**
| Level | Description |
|-------|-------------|
| **Org Level** | System administrator controls |
| **Workspace Level** | Workspace-specific access |
| **Asset Level** | Individual dashboard/metric permissions |
| **Row Level** | Data-level security (via Data 360 policies) |

**License Types:**
- **Viewer:** Consume published content only
- **Explorer:** Interact with and create some content
- **Creator:** Full authoring and publishing capabilities
- **Admin:** Full platform administration

---

### Section 5: Organize and Deploy Analytical Assets
**Workspace Strategy:**
- Workspaces are collaborative containers for analytical assets
- Group assets by: business function, team, project, or data domain
- Best Practice: Align workspaces with organizational structure

**Asset Types in Tableau Next:**
- Dashboards
- Metrics
- Tableau Agent conversations
- Data sources (via Semantic Models)
- Marketplace assets

**Sandbox Deployment:**
- Test configurations before production deployment
- Validate data connections and permissions
- Preview dashboard behavior with sample data

---

### Section 6: Analyze and Share Data
**Visualization Builder:**
- Drag-and-drop chart creation (no SQL required)
- Natural language chart generation via Tableau Agent
- Dashboard templating for rapid deployment
- Navigation flows and action integration

**Tableau Agent Capabilities:**
- Ask questions in natural language
- Get AI-generated visualizations
- Drill into metrics on demand
- Receive proactive alerts when data changes

**Single-Click Action Workflows:**
- Trigger Salesforce Flows directly from dashboards
- Connect insights to business actions
- Eliminate context switching between apps

**Proactive Alerts (Beta):**
- Set metric thresholds
- Receive notifications when KPIs cross limits
- Delivered via Slack or in-app

---

### Section 7: Integrate Analytics Across Platforms
**Slack Integration:**
- Share metrics and dashboards directly in Slack channels
- Subscribe teams to data updates
- Answer data questions within Slack using Tableau Agent

**Model Context Protocol (MCP):**
- Standardized protocol for connecting AI agents to data tools
- Allows external AI agents to query Tableau semantics
- Set up MCP server for Slack workflow integration

**API & SDK Capabilities:**
- REST API for programmatic content management
- Embedding SDK for custom app integration
- App Template Framework for ISV development
- AgentExchange for solution discovery

---

## Trail 3: Data Cloud 360 Consultant Exam Prep

### 🎯 Exam Coverage Areas (Weighted Topics)
1. Solution Positioning & Value (~12%)
2. Setup & Administration (~17%)
3. Data Source Connection & Ingestion (~13%)
4. Harmonization & Unification (~9%)
5. Data Enhancements, Sharing & Analysis (~12%)
6. Data Activations & Utilization (~14%)
7. Hands-On Implementation (~23%)

---

### Section 1: Data 360 Solution Positioning ⭐
**What is Data 360?**
Data 360 (formerly Data Cloud) is Salesforce's **Customer Data Platform (CDP)** that:
- Ingests data from ANY source
- Unifies customer profiles across touchpoints
- Activates data across the Salesforce ecosystem
- Enables real-time AI and analytics

**Core Value Proposition:**
- **Single Customer View:** Eliminate data silos
- **Real-Time Activation:** Act on data instantly
- **AI-Ready Data:** Power Agentforce with trusted data
- **Native Salesforce Integration:** No complex ETL

**Credit Consumption Model:**
- Data 360 uses a credit-based consumption model
- Credits consumed by: data ingestion, storage, queries, activations
- Proper sizing and planning is an exam topic

**Real-Time Use Cases:**
- Abandoned cart triggers
- Real-time personalization
- Service case enrichment
- Sales opportunity scoring

**Ethical Data Use:**
- Consent management integration
- Purpose-based data usage
- Compliance with privacy regulations (GDPR, CCPA)

---

### Section 2: Data 360 Setup and Administration ⭐⭐
**Initial Setup Steps:**
1. Enable Data Cloud in Salesforce org
2. Configure org-wide settings
3. Set up data spaces
4. Configure user permissions and roles
5. Connect initial data sources
6. Establish governance policies

**Data Spaces:**
- Logical partitions within Data 360 for data isolation
- Use cases: geographic regions, business units, brands
- Each data space has its own schemas and access controls

**Data Governance Strategies:**
| Component | Description |
|-----------|-------------|
| **Tags** | Metadata labels for data classification |
| **Classifications** | Sensitivity tiers (Public, Internal, Confidential, Restricted) |
| **Policies** | Rules governing data access and usage |
| **Audit Logs** | Track who accessed or modified data |

**Data Cloud One:**
- Multi-org expansion capability
- Share data across multiple Salesforce orgs
- Centralized governance with distributed access

**Communication Capping:**
- Limits on how frequently customers can be contacted
- Prevents over-messaging and fatigue
- Configured at the org or segment level

**Packaging and Data Kits:**
- Package data assets for reuse across orgs
- Data Kits: pre-built data bundles for quick deployment
- Share via AgentExchange marketplace

---

### Section 3: Data Source Connection and Ingestion ⭐⭐
**Connector Types:**
| Type | Examples | Use Case |
|------|---------|---------|
| **Native Salesforce** | Sales Cloud, Service Cloud, Marketing Cloud | Default CRM data |
| **Cloud Storage** | Amazon S3, Azure Blob, GCS | Batch file ingestion |
| **Database** | MySQL, PostgreSQL, Snowflake | Direct DB query |
| **Streaming** | MuleSoft, Kafka, Event Bus | Real-time events |
| **Zero Copy** | Snowflake, Databricks | Query in place, no copy |

**Zero Copy Integration:**
- Data stays in the external platform (no duplication)
- Query performance depends on external system
- Reduces storage costs and latency
- Supported partners: Snowflake, Databricks, Google BigQuery

**Batch Data Transforms:**
- SQL-based transformations on ingested data
- Scheduled execution
- Use for: aggregations, enrichments, complex joins
- Output to Data Model Objects (DMOs)

**Streaming Data Transforms:**
- Real-time transformations as data arrives
- Low-latency use cases
- Event-driven processing

**Fully Qualified Keys:**
- Unique identifiers that resolve conflicts when data from multiple sources uses the same key values
- Format: `{SourceSystem}__{ObjectName}__{KeyField}`
- Critical for data integrity in multi-source environments

---

### Section 4: Harmonization and Unification ⭐⭐
**Customer 360 Data Model (C360DM):**
The standard object model for unified customer data in Data 360.

**Key Standard Objects:**
| Object | Description |
|--------|-------------|
| **Individual** | Person-level records |
| **Unified Individual** | Deduplicated, merged person profile |
| **Contact Point** | Email, phone, address records |
| **Party Identification** | External IDs (loyalty #, CRM ID, etc.) |
| **Product** | Product catalog data |
| **Sales Order** | Transaction records |

**Data Mapping:**
- Connect source fields to C360DM standard fields
- Ensures consistent definitions across sources
- Required for identity resolution to work correctly

**Identity Resolution:**
- Processes that match and merge records from different sources
- **Deterministic Matching:** Exact match on key (email, phone)
- **Probabilistic Matching:** Fuzzy match on multiple partial signals
- Creates **Unified Individual** from multiple source records
- **Match Rules:** Configurable rules defining when records should merge
- **Reconciliation Rules:** Define which field value "wins" when merging

---

### Section 5: Data Enhancements, Sharing, and Analysis ⭐
**Data Insights:**
- **Calculated Insights:** SQL-based computed metrics stored as attributes on profiles
- **Real-Time Insights:** Computed at query time for up-to-the-second accuracy
- Examples: Lifetime Value, Engagement Score, Days Since Purchase

**Data Enrichments:**
- Append external data to customer profiles
- CRM enrichment: pull in Salesforce CRM fields
- Third-party data: append purchased data attributes

**Reporting in Data 360:**
- Standard Reports: pre-built dashboards (ingestion health, profile counts)
- Custom Reports: user-defined queries and visualizations
- Integration with Tableau Next for advanced analytics

**Machine Learning Predictions:**
- Train models on Data 360 data
- Apply predictions as profile attributes
- Use cases: churn prediction, propensity scoring, product recommendations
- AI Models: configure, train, and deploy within Data 360

---

### Section 6: Data Activations and Utilization ⭐⭐
**Segmentation:**
- **Segments:** Filtered subsets of unified profiles
- **Segment Criteria:** Attribute filters, engagement history, predictive scores
- **Nested Segments:** Segments within segments
- **Real-Time Segments:** Update membership dynamically as data changes

**Activation Types:**
| Type | Description |
|------|-------------|
| **CRM Activation** | Sync segments to Sales/Service Cloud objects |
| **Marketing Activation** | Push to Marketing Cloud journeys |
| **Advertising Activation** | Sync to Google, Meta, LinkedIn ad platforms |
| **CDP Activation** | Push to external CDPs or data warehouses |

**Advertising with Data 360:**
- First-party data for targeted advertising
- Audience sync to ad platforms
- Lookalike audience creation
- Suppression lists for opted-out contacts

**Marketing Cloud Engagement Integration:**
- Journey Builder: trigger journeys based on Data 360 segments
- Send Time Optimization: use profile data for best send time
- Personalization: dynamic content based on unified profile

**Agentforce Integration:**
- **Sales Enhancements:** Account enrichment, opportunity scoring, next best action
- **Service Enhancements:** Case context enrichment, resolution suggestions
- **Agentforce Agents powered by Data 360:** Agents query live unified data to respond accurately

---

### Section 7: Hands-On Implementation
**Core Functionality Projects:**
1. **Connect** — Set up a data source connector
2. **Unify** — Run identity resolution
3. **Query** — Use Data Explorer to query profiles
4. **Segment** — Create a segment
5. **Activate** — Push segment to a destination

**Unstructured Data in Data 360:**
- Ingest text, documents, and media files
- Use for: AI search grounding, sentiment analysis
- Stored in Data Lake Objects (DLOs)
- Queryable via vector search

**Data Graphs:**
- Visual representation of how data objects relate
- Configure relationships between DMOs
- Used for complex queries and activation targeting
- Support traversal across multiple object hops

---

## Unified Concept Map: How It All Connects

```
[External Sources]
     │
     ▼
[Data 360 Ingestion Layer]
  ├── Batch (S3, DB, files)
  ├── Streaming (events, APIs)
  └── Zero Copy (Snowflake, Databricks)
     │
     ▼
[Data Lake Objects (DLOs)] ← Raw ingested data
     │
     ▼ (Data Transforms + Mapping)
[Data Model Objects (DMOs)] ← Harmonized C360DM objects
     │
     ▼ (Identity Resolution)
[Unified Individual Profiles] ← Single customer view
     │
     ├──────────────────────────────────────┐
     ▼                                      ▼
[Semantic Models]                    [Segments]
  (Tableau Next)                      (Activation)
     │                                      │
     ▼                                      ▼
[Tableau Agent]                    [Marketing Cloud]
[Dashboards]                       [Agentforce]
[Metrics & Alerts]                 [Advertising]
[Slack Integration]                [CRM Objects]
```

---

## Exam Cheat Sheet: Tableau Next Consultant

### ✅ Must-Know Concepts

**Semantic Models:**
- Business-friendly data layer for Tableau Agent
- Contains: metrics, dimensions, relationships, descriptions
- AI uses descriptions to understand business context
- Marketplace templates = faster development

**Workspaces:**
- Organizational containers for analytical assets
- Align with business functions or teams
- Control access at workspace + asset level

**Tableau Agent:**
- Conversational AI for data analysis
- Requires well-defined semantic models to function accurately
- Can trigger Salesforce Flows (action layer)

**MCP (Model Context Protocol):**
- Protocol for external AI agents to access Tableau data
- Used in Slack integration for cross-platform AI

**Security Model (4 levels):**
1. Org level (System Admin)
2. Workspace level
3. Asset level (dashboards, metrics)
4. Row level (via Data 360 policies)

**License Types:** Viewer → Explorer → Creator → Admin

### ❌ Common Exam Traps
- Semantic models ≠ Data Model Objects (DMOs). Semantic models are the *analytics layer*; DMOs are the *data layer*.
- Personal Org is Beta — self-service analysis sandbox
- Proactive Alerts are Beta — threshold-based metric notifications
- MCP is for external AI agent connections, not for standard user setup

---

## Exam Cheat Sheet: Data 360 Consultant

### ✅ Must-Know Concepts

**Data Flow Sequence:**
`Source → Connector → DLO → Transform → DMO → Identity Resolution → Unified Individual → Segment → Activation`

**DLO vs. DMO:**
| | Data Lake Object (DLO) | Data Model Object (DMO) |
|--|----------------------|------------------------|
| Contains | Raw ingested data | Harmonized, mapped data |
| Schema | Source schema | C360DM standard schema |
| Use | Storage, unstructured | Analysis, identity resolution |

**Identity Resolution:**
- Deterministic = exact match (email = email)
- Probabilistic = fuzzy match (similar name + address)
- Output = Unified Individual

**Segment Types:**
- Standard Segment: batch, scheduled refresh
- Real-Time Segment: live membership updates

**Activation Destinations:**
- Salesforce CRM (Sales/Service Cloud)
- Marketing Cloud (Journey Builder)
- Advertising (Google, Meta, LinkedIn)
- External CDPs

**Zero Copy:**
- Data stays in source system (Snowflake, Databricks)
- No duplication = lower storage cost
- Query performance tied to external system

**Data Spaces:**
- Logical partition within Data 360
- Use for: regions, brands, business units

**Communication Capping:**
- Prevents over-messaging customers
- Configured at org or segment level

**Calculated Insights:**
- SQL-computed metrics stored on profiles
- Run on schedule; not real-time

**Real-Time Insights:**
- Computed at query time
- Always current but more compute-intensive

### ❌ Common Exam Traps
- Data Kits ≠ Segments. Data Kits are packaged data assets; Segments are profile subsets.
- Fully Qualified Keys prevent ID conflicts across sources — required when same key exists in multiple orgs.
- Zero Copy = **no data movement** (query in place, not a copy)
- Unified Individual is created AFTER identity resolution runs — not during ingestion
- DLOs store raw data; you must map/transform to DMOs before segmentation

---

## Glossary of Key Terms

| Term | Definition |
|------|------------|
| **Agentforce** | Salesforce's AI agent platform; Data 360 powers agent grounding |
| **Batch Transform** | Scheduled SQL-based data transformation |
| **C360DM** | Customer 360 Data Model — standard schema for unified data |
| **Calculated Insight** | SQL-computed metric stored as a profile attribute |
| **Communication Capping** | Limits on customer contact frequency |
| **Data Graph** | Visual map of how data objects relate; supports complex traversal |
| **Data Kit** | Packaged, reusable data assets for cross-org deployment |
| **Data Lake Object (DLO)** | Storage container for raw ingested data |
| **Data Model Object (DMO)** | Harmonized data object aligned to C360DM schema |
| **Data Space** | Logical partition in Data 360 for data isolation |
| **Deterministic Match** | Exact-key identity matching |
| **Fully Qualified Key** | Namespace-prefixed key to prevent cross-source ID conflicts |
| **Identity Resolution** | Process of merging records into a Unified Individual |
| **LOD Expression** | Level of Detail expression in Tableau (FIXED/INCLUDE/EXCLUDE) |
| **MCP** | Model Context Protocol — standard for AI-to-tool communication |
| **Personal Org (Beta)** | Self-service sandbox in Tableau Next |
| **Probabilistic Match** | Fuzzy/multi-signal identity matching |
| **Real-Time Insight** | Metric computed at query time (always fresh) |
| **Reconciliation Rule** | Defines which field value wins when merging records |
| **Semantic Model** | Business-friendly analytics layer on top of raw data |
| **Streaming Transform** | Real-time, event-driven data transformation |
| **Tableau Agent** | Conversational AI in Tableau Next for natural language analytics |
| **Unified Individual** | Deduplicated customer profile created after identity resolution |
| **Workspace** | Collaborative container for analytical assets in Tableau Next |
| **Zero Copy** | Query external data in place without ingesting/duplicating it |

---

*Document generated from Salesforce Trailhead trails. For exam preparation, always cross-reference with the official Salesforce Exam Guide.*
