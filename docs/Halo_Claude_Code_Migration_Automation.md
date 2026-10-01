# Leveraging Halo (Claude Code) for CRM Analytics → Tableau Next Migration
## What Can Be Automated vs What Needs the UI

> Based on the DTC Sales and Olympus Opportunities dashboard migrations
> Author: Ajay Thakuri | Date: October 2026

---

## What Is Halo / Claude Code?

Halo (Claude Code) is an AI coding agent that operates directly inside your
development environment. It has access to:

- A full terminal / bash shell
- The file system (read, write, edit files)
- Salesforce CLI (`sf` commands)
- Git and GitHub
- Google Cloud Storage
- The ability to run SOQL queries against any authenticated Salesforce org

In the context of a Salesforce dashboard migration, this means Halo can
act as a developer — reading metadata, writing XML files, deploying to orgs,
diagnosing errors, and committing code — all without leaving the terminal.

---

## Overall Automation Score

| Phase | Halo Can Handle | Needs UI |
|---|---|---|
| Audit CRM Analytics | ~80% | Screenshots, business context |
| Data Cloud / DLO setup | ~10% | Ingestion is entirely UI-based |
| Workspace creation | ~100% | — |
| Semantic Model creation | ~30% | SM must be created in Tableau Next UI |
| SM field name discovery | ~100% | — (deploy-and-fail loop) |
| Visualization creation | ~90% | Visual QA only |
| Dashboard creation | ~85% | Open and validate in browser |
| Error diagnosis & fixing | ~90% | — |
| Testing & validation | ~50% | Number matching, layout review |
| Git / CI / PR management | ~100% | PR approval only |
| Documentation | ~100% | — |
| **Overall** | **~70%** | **~30% UI** |

---

## Step-by-Step: Halo vs UI

### Step 1 — Audit CRM Analytics

| Task | Halo | UI |
|---|---|---|
| Retrieve all CRM Analytics app metadata | ✅ | |
| Parse dashboard JSON — extract widget types, fields, aggregations | ✅ | |
| Run SOQL to confirm field names and data | ✅ | |
| Screenshot dashboard states for reference | | 🖥️ Manual |
| Understand business intent behind dashboards | | 🖥️ Manual |

---

### Step 2 — Data Cloud Setup

| Task | Halo | UI |
|---|---|---|
| Check if Data Lake Object (DLO) exists | ✅ | |
| Ingest CSV / CRM data into Data Cloud | | 🖥️ Manual |
| Set up Salesforce CRM Connector | | 🖥️ Manual |
| Note DLO API name for SM setup | ✅ | |

---

### Step 3 — Workspace Creation

| Task | Halo | UI |
|---|---|---|
| Write workspace XML file | ✅ | |
| Deploy workspace to Tableau Next | ✅ | |
| Retrieve and verify workspace metadata | ✅ | |
| Register DLO and SM as workspace assets | ✅ | |

---

### Step 4 — Semantic Model (Biggest Gap)

> `AnalyticsSemanticModel` is **not in the SFDX metadata registry**.
> Creation must be done in the Tableau Next UI.
> However, once created, Halo takes over field discovery completely.

| Task | Halo | UI |
|---|---|---|
| Create Semantic Model | | 🖥️ Manual — Workspace → New → Semantic Model |
| Add fields and configure data types | | 🖥️ Manual |
| Discover exact SM field names | ✅ | |
| Build complete CRM Analytics → SM field mapping table | ✅ | |

**How Halo discovers SM field names automatically (deploy-and-fail loop):**

```
1. Halo writes a test viz XML with guessed field name (e.g. Amount2)
2. Deploys to org
3. If deploy fails → "F1 isn't a valid field" → Halo increments suffix → tries Amount3
4. If deploy succeeds → field name confirmed
5. Repeat for every field needed
6. Halo documents all confirmed field names in a mapping table
```

**Confirmed field names from Olympus migration:**

| SOQL Field | Tableau Next SM Field |
|---|---|
| `Amount` | `Amount2` |
| `Name` | `Name2` |
| `CloseDate` | `Close_Date3` |
| `StageName` | `Stage2` |
| `Type` | `Type1` |
| `Owner.Name` | `Owner_Name2` |

---

### Step 5 — Create Visualizations

| Task | Halo | UI |
|---|---|---|
| Generate viz XML for each dashboard widget | ✅ | |
| Generate base64 visualSpecification (chart config JSON) | ✅ | |
| Assign brand-new API names (prevent SM reversion) | ✅ | |
| Deploy all vizes to org | ✅ | |
| Retrieve and verify SM binding after deploy | ✅ | |
| Auto-detect and fix SM reversion (new API name) | ✅ | |
| Visual QA — does chart render correctly | | 🖥️ Manual |

**Critical rule Halo enforces:** Always generate brand-new API names.
If a viz API name was ever previously deployed against a different SM,
the platform silently reverts the SM after deploy — even if deploy says Succeeded.
Halo detects this via retrieve + grep and automatically re-creates with a new name.

---

### Step 6 — Build the Dashboard

| Task | Halo | UI |
|---|---|---|
| Generate dashboard XML with full layout | ✅ | |
| Calculate 48-column grid positions | ✅ | |
| Apply dark navy theme (`#03234D`) | ✅ | |
| Wire filter widgets to correct SM and fields | ✅ | |
| Deploy dashboard to org | ✅ | |
| Open and visually validate in browser | | 🖥️ Manual |

**Rules Halo knows and enforces:**
- No XML comments inside `<pages>` or `<widgets>` → causes runtime error even though deploy succeeds
- Use `toggle` not `comboBox` for filter `viewType` → `comboBox` may fail at runtime
- Deploy vizes BEFORE the dashboard that references them
- Every widget's `<analyticsDashboard>` must match the dashboard API name exactly

---

### Step 7 — Error Diagnosis

When a dashboard fails with `"We couldn't find the Dashboard: [name]"`,
Halo runs a systematic diagnosis:

```
1. Retrieve all vizes → grep dataSource → check for wrong SM
2. Check if wrong SM belongs to a different workspace
3. Check dashboard XML for XML comments
4. Check filter widget viewType
5. Check all workspaceAssetRelationships are correctly declared
6. If viz SM is wrong → generate new API name → redeploy → verify
```

All of this is fully automated. Halo does not need to be told what to check —
it knows the failure patterns from experience.

---

### Step 8 — Git, CI/CD, Documentation

| Task | Halo | UI |
|---|---|---|
| Commit all files with descriptive message | ✅ | |
| Push to feature branch | ✅ | |
| Create pull request with summary | ✅ | |
| Comment on PR with deployment details | ✅ | |
| Read CI check results | ✅ | |
| Approve and merge PR | | 🖥️ Human governance |
| Write migration guides and documentation | ✅ | |

---

## APIs Halo Uses

### Salesforce CLI (`sf`) — Core tool ✅
```bash
sf project deploy start    # Deploy metadata to org
sf project retrieve start  # Pull metadata from org
sf data query              # Run SOQL queries
sf org list                # List authenticated orgs
```

### Salesforce Metadata API (via SFDX) ✅
All metadata operations — deploy, retrieve, validate — run through the
Metadata API via the SF CLI.

### Wave REST API ⚠️ Partially available
```
GET /wave/datasets          ✅ Available — list CRM Analytics datasets
GET /wave/dashboards        ✅ Available — read dashboard JSON
GET /wave/semanticModels    ❌ FUNCTIONALITY_NOT_ENABLED
GET /wave/semanticModels/id ❌ FUNCTIONALITY_NOT_ENABLED
```
> If SM endpoints were enabled, Halo could discover all field names in a
> single API call — eliminating the deploy-and-fail loop entirely.

### SOQL ✅
```sql
SELECT Id, Name, Amount, CloseDate, StageName, Type,
       Owner.Name, Account.Name, Account.Type, Account.BillingCountry
FROM Opportunity
```
Used to audit source data and confirm field names from the CRM Analytics org.

### GitHub MCP ✅
```
create_pull_request         — Open PR after migration
comment_on_pull_request     — Add notes to PR
get_pull_request_checks     — Read CI results
get_pull_request_diff       — Review changes
view_pull_request           — Read PR status
```

### Google Cloud Storage MCP ✅
```
pull_from_source   — Download reference screenshots / CSV files
push_to_output     — Upload generated docs to shared bucket
list_source        — List available source files
```

---

## What Would Unlock Full Automation (90%+)

| Gap Today | What Would Fix It |
|---|---|
| SM creation requires UI | Salesforce adds `AnalyticsSemanticModel` to SFDX metadata registry |
| SM field names need deploy-and-fail discovery | Salesforce enables `GET /wave/semanticModels/{id}` for all user profiles |
| DLO ingestion requires UI | Salesforce exposes Data Cloud Bulk Ingestion via REST API |
| Visual validation needs human | Automated screenshot comparison tooling |

---

## Summary

```
TODAY — Halo handles ~70% automatically

  ✅ Reads all CRM Analytics metadata
  ✅ Runs SOQL field audits
  ✅ Discovers SM field names (deploy-and-fail)
  ✅ Generates all viz and dashboard XML files
  ✅ Deploys, verifies, and auto-fixes SM issues
  ✅ Manages Git, PRs, and documentation

  🖥️ Human does 30%:
     - Create the Semantic Model in Tableau Next UI
     - Set up Data Cloud DLO ingestion
     - Visual sign-off on the final dashboard
     - Approve and merge the pull request

WITH FULL API ACCESS — Halo could handle ~90%

  + SM field names discovered via API (no deploy loop)
  + DLO fields confirmed directly (no guessing)
  + Only visual QA and PR approval remain human tasks
```

---

*All automation percentages and API findings are based on real migration work
on the DTC Sales and Olympus Opportunities dashboards — October 2026.*
