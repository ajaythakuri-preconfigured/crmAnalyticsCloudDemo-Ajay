# CRM Analytics vs Tableau Next (Data 360)
## General Product Comparison

---

## What Are They?

### CRM Analytics
CRM Analytics (formerly Einstein Analytics / Wave Analytics) is Salesforce's **native, embedded analytics platform** built specifically for Salesforce CRM users. It lives inside the Salesforce platform and is designed to give sales, service, and marketing teams actionable insights directly within their CRM workflow.

### Tableau Next (Data 360)
Tableau Next is the **next generation of Tableau**, deeply integrated with Salesforce Data Cloud. "Data 360" refers to the full Salesforce vision of combining Tableau's world-class visualization engine with Data Cloud's unified data platform — giving organisations a 360-degree view of their customer data across all sources, not just CRM.

---

## 1. Purpose & Philosophy

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Core purpose** | Analytics **within** the Salesforce CRM | Analytics **across all data**, powered by Data Cloud |
| **Philosophy** | Bring insights to CRM users in their daily workflow | Unify all enterprise data and make it explorable |
| **Data scope** | Primarily Salesforce CRM data (Opportunities, Cases, Leads, etc.) | Any data — CRM, ERP, web, IoT, third-party — unified in Data Cloud |
| **Primary outcome** | Faster CRM decisions (close more deals, resolve cases faster) | Enterprise-wide data intelligence and customer 360 visibility |

---

## 2. Target Users

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Primary users** | Sales reps, sales managers, service agents, CRM admins | Data analysts, BI teams, data engineers, business leaders |
| **Technical skill required** | Low to medium — drag and drop, pre-built templates | Medium to high — data modelling, Semantic Model design |
| **Works best for** | Teams already living in Salesforce | Organisations with data spread across multiple systems |
| **Admin persona** | Salesforce Admin | Salesforce Admin + Data Cloud Architect + BI Developer |

---

## 3. Data Architecture

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Where data lives** | Salesforce Analytics datasets (proprietary store inside CRM Analytics) | **Salesforce Data Cloud** — a unified, scalable data lake |
| **Data sources** | Salesforce objects, CSV uploads, connected apps via dataflows | Any source — Salesforce, Snowflake, AWS S3, Google BigQuery, SAP, web events, IoT, etc. |
| **Data model** | Flat datasets with SAQL-based transformations | **Data Model Objects (DMOs)** and **Data Lake Objects (DLOs)** with relationships |
| **Data unification** | Limited — primarily Salesforce-native | Full **identity resolution** and **data harmonisation** across all sources |
| **Real-time data** | Near real-time via Salesforce object sync | Real-time streaming ingestion into Data Cloud |
| **Data volume** | Designed for CRM-scale data | Built for enterprise scale — billions of rows |

---

## 4. Analytics & Visualisation Capabilities

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Visualisation engine** | Proprietary SAQL-powered renderer | **VizQL** — Tableau's industry-leading visual query language |
| **Chart types** | Bar, donut, funnel, scatter, map, KPI, timeline, gauge, table | All Tableau chart types — maps, LOD calculations, dual-axis, forecasting, etc. |
| **Calculation language** | SAQL (proprietary, Salesforce-specific) | VizQL + Tableau Calculated Fields (industry-standard) |
| **Dashboard interactivity** | Actions, filters, cross-filtering, parameters | Full Tableau interactivity — cross-filtering, sets, parameters, actions, drill-down |
| **Mobile experience** | Salesforce mobile app — optimised for mobile | Tableau Mobile app |
| **Embedded analytics** | Deeply embedded inside Salesforce Lightning pages | Can be embedded in Salesforce pages via Tableau Embedded Analytics |

---

## 5. AI & Intelligence Features

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **AI branding** | Einstein AI | Einstein + Tableau AI (formerly Ask Data, Explain Data) |
| **Predictive analytics** | Einstein Prediction Builder — predictions on Salesforce records (e.g. likelihood to close) | AI-powered forecasting, anomaly detection, cluster analysis |
| **Natural language** | Einstein Discovery insights in plain language | **Tableau Pulse** — AI-generated natural language insights and digest emails |
| **Automated insights** | Einstein Discovery recommends drivers of metrics | Explain Data and Einstein Copilot for Tableau |
| **GenAI** | Einstein Copilot integration for CRM data | **Tableau Pulse with Einstein Copilot** — ask questions of your data in plain English |
| **Segmentation** | Basic | **Data Cloud AI Segmentation** — ML-powered audience segmentation across all data |

---

## 6. Data Cloud Integration (Key Differentiator)

This is where Tableau Next (Data 360) has a fundamental advantage:

| Capability | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Identity Resolution** | ❌ Not available | ✅ Unifies customer records across all systems into a single profile |
| **Customer 360 Profile** | ❌ Only Salesforce CRM profile | ✅ Unified profile from CRM + Marketing + Commerce + Service + third-party |
| **Activation** | ❌ | ✅ Segments created in Data Cloud can be activated to Marketing Cloud, Ad platforms, etc. |
| **Data Harmonisation** | ❌ | ✅ Maps disparate data to a standard data model |
| **Consent & Compliance** | Basic | ✅ Data Cloud manages consent, data residency, GDPR compliance |

---

## 7. Semantic Model vs Dataset

This is the core technical difference in how data is served to analytics:

### CRM Analytics — Dataset
- A **Dataset** is a flat, denormalised snapshot of data stored inside CRM Analytics
- Data is prepared via **Dataflows** or **Recipes** — ETL processes that join, transform, and load data
- Once in a dataset, field names are fixed and directly used in SAQL queries and charts
- Changes to data require re-running the dataflow/recipe

### Tableau Next — Semantic Model
- A **Semantic Model (SM)** is a **live, relationship-aware layer** on top of Data Cloud DLOs
- No pre-aggregation or flattening required — the SM defines joins and metrics at query time
- Supports multiple objects with relationships (like a star schema)
- Field names are assigned at the SM level and may differ from the underlying DLO field names
- Multiple workspaces can have their own SMs pointing to the same DLOs

---

## 8. Development & Administration

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Build tool** | CRM Analytics Studio (web-based, in Salesforce) | Tableau Desktop + Tableau Next web editor |
| **Deployment** | SFDX metadata deploy — fully supported | SFDX metadata deploy — partially supported (Semantic Model has restrictions) |
| **Version control** | Via SFDX + Git | Via SFDX + Git (with caveats) |
| **Pre-built templates** | Extensive library of industry-specific CRM Analytics apps | Tableau Exchange accelerators + Data Cloud starter templates |
| **API access** | Wave REST API — full CRUD | Wave REST API — limited by user profile permissions |
| **Licensing** | Included in some Salesforce editions; standalone Einstein Analytics license | Requires **Tableau license** + **Data Cloud credits** — separate entitlement |

---

## 9. Performance & Scale

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Query performance** | Optimised for Salesforce CRM data volumes | Optimised for large-scale Data Cloud queries (billions of rows) |
| **Refresh frequency** | Scheduled dataflow runs (hourly/daily) or live Salesforce connections | Near real-time with Data Cloud streaming ingestion |
| **Caching** | Dataset-level caching | Data Cloud query cache + Tableau extract options |
| **Multi-cloud data** | Requires data ingestion into CRM Analytics first | **Direct federation** — query external data sources without moving data |

---

## 10. Licensing & Cost

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Included with** | Sales Cloud, Service Cloud (limited), or standalone Einstein Analytics SKU | Requires Tableau Creator/Explorer license + Data Cloud entitlement |
| **Data Cloud credits** | Not required | Required — Data Cloud charges per data volume and query |
| **Add-ons** | Einstein Discovery (predictive), Einstein Prediction Builder | Tableau Pulse, Einstein Copilot for Tableau, Data Cloud AI |
| **Cost model** | Per-user license | Per-user (Tableau) + consumption-based (Data Cloud) |

---

## 11. When to Choose Each

### Choose CRM Analytics if:
- Your team lives and works in Salesforce every day
- Your data is primarily Salesforce CRM data (Opportunities, Cases, Contacts)
- You need embedded analytics directly in Salesforce record pages and Lightning
- Your users are non-technical (sales reps, managers) who need pre-built apps
- You need Salesforce-specific features like Opportunity scoring, Case trending, Activity tracking
- Budget is a constraint — CRM Analytics may already be included in your Salesforce edition

### Choose Tableau Next (Data 360) if:
- You need to combine **data from multiple systems** (Salesforce + ERP + web + marketing + commerce)
- You want a **unified customer profile** across all touchpoints
- Your analytics team is experienced with Tableau
- You need advanced analytics, LOD calculations, and complex data relationships
- You want to activate insights back into marketing campaigns, personalisation, and engagement
- You are building a **modern data platform** with Data Cloud at the centre
- You need real-time streaming data analysis
- You want AI-powered natural language insights via Tableau Pulse and Einstein Copilot

---

## 12. The Convergence (Where Salesforce is Heading)

Salesforce's strategic direction is to **converge CRM Analytics and Tableau Next** into a unified analytics layer on top of Data Cloud:

- CRM Analytics is gradually being enhanced to support Data Cloud as a data source
- Tableau Next brings Tableau's best-in-class visualisation into the Salesforce platform
- **Data Cloud** is the unifying data layer that both products will eventually share
- **Einstein Copilot** and **Tableau Pulse** are converging on a single AI-powered analytics experience
- The long-term vision is: one data platform (Data Cloud) → one semantic layer → one analytics experience (Tableau Next)

---

## Summary

| | CRM Analytics | Tableau Next (Data 360) |
|---|---|---|
| **Best for** | CRM insights for Salesforce users | Enterprise-wide analytics on unified customer data |
| **Data scope** | Salesforce CRM | All enterprise data via Data Cloud |
| **Analytics power** | Good for CRM use cases | World-class (Tableau engine) |
| **AI** | Einstein predictions within CRM | Tableau Pulse + Einstein Copilot across all data |
| **Complexity** | Lower — targeted at CRM admins | Higher — requires data platform expertise |
| **Cost** | Often bundled with Salesforce | Additional Tableau + Data Cloud licensing |
| **Future direction** | Converging toward Data Cloud | The strategic platform of the future |

---

*Salesforce CRM Analytics and Tableau Next both serve analytics needs but at different levels of data maturity and organisational complexity. CRM Analytics solves the "last mile" analytics problem for CRM users, while Tableau Next (Data 360) solves the enterprise-wide customer intelligence problem.*
