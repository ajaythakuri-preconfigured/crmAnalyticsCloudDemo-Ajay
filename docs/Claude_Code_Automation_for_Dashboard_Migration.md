# Leveraging Claude Code (Halo) for Dashboard Migration
## CRM Analytics → Tableau Next / Tableau CRM

> Based on the full hands-on DTC Sales and Olympus Opportunities migrations
> Author: Ajay Thakuri | Date: October 2026

---

## Overview

Claude Code (Halo) is an AI coding agent with access to a terminal, file system,
Salesforce CLI, Git, and GitHub. This document maps every step of the
CRM Analytics → Tableau Next migration and tells you exactly:

- ✅ **Fully automated** — Claude Code handles it end-to-end
- ⚠️ **Partially automated** — Claude Code does the heavy lifting, minimal UI needed
- 🖥️ **UI required** — Must be done manually in the Salesforce / Tableau Next browser UI
- ❌ **Not possible** — API/platform does not support it programmatically

---

## Migration Steps — Automation Map

### Step 1 — Audit the CRM Analytics Dashboard

| Task | Who handles it | How |
|---|---|---|
| List all CRM Analytics assets in an app | ✅ Claude Code | `sf project retrieve start --metadata WaveApplication:AppName` |
| Read dashboard JSON to extract widgets | ✅ Claude Code | Reads `.wdash-meta.json` files, parses widget types, fields, aggregations |
| Run SOQL to confirm field names | ✅ Claude Code | `sf data query --query "SELECT ..." --target-org crmOrg` |
| Screenshot each dashboard state | 🖥️ UI required | Must be done manually in the browser |
| Understand business logic / intent | ⚠️ Partially | Claude reads the metadata but context about *why* requires human input |

---

### Step 2 — Set Up Data Cloud

| Task | Who handles it | How |
|---|---|---|
| Check if DLO already exists | ✅ Claude Code | `sf data query --query "SELECT ... FROM DataLakeObject__dlm" --target-org dataCloudOrg` |
| Ingest CSV data into Data Cloud | 🖥️ UI required | Data Cloud ingestion setup is UI-only |
| Set up CRM Connector (SF → Data Cloud) | 🖥️ UI required | Connector configuration is UI-only |
| Verify DLO has data | ✅ Claude Code | Query the DLO via SOQL or REST |
| Note DLO API name | ✅ Claude Code | Retrieved from metadata or SOQL |

---

### Step 3 — Create the Workspace

| Task | Who handles it | How |
|---|---|---|
| Create workspace metadata file | ✅ Claude Code | Writes `.uawork-meta.xml` with correct structure |
| Deploy workspace to Tableau Next | ✅ Claude Code | `sf project deploy start --metadata AnalyticsWorkspace:Name` |
| Retrieve and verify workspace | ✅ Claude Code | `sf project retrieve start` + file inspection |
| Register DLO and SM as workspace assets | ✅ Claude Code | Edits `workspaceAssetRelationships` in workspace XML |

---

### Step 4 — Create the Semantic Model

> **This step is the biggest constraint for automation.**
> `AnalyticsSemanticModel` is NOT in the SFDX metadata registry.

| Task | Who handles it | How |
|---|---|---|
| Create the Semantic Model | 🖥️ UI required | Must be created in Tableau Next UI — Workspace → New → Semantic Model |
| Add fields to the SM | 🖥️ UI required | Field selection and mapping is UI-only |
| Retrieve SM metadata | ❌ Not possible | `AnalyticsSemanticModel` is not in the SFDX metadata registry — CLI returns an error |
| Discover SM field names via test viz | ⚠️ Claude Code + UI | Claude creates and deploys test viz XML; reads error messages or retrieved XML to identify field suffixes |
| Discover SM field names via deploy-and-fail | ✅ Claude Code | Claude iterates: deploy viz with guessed field → read error → try next suffix → repeat |
| Build complete field name mapping | ✅ Claude Code | After discovery, Claude documents and maps all field names |

**Field Discovery Loop (fully automated by Claude Code):**
```
Claude guesses: Amount2 → deploys → succeeds → confirmed
Claude guesses: Stage2 → deploys → "F1 isn't valid" → tries Stage3 → succeeds
Claude guesses: Close_Date3 → deploys → succeeds → confirmed
... and so on for every field needed
```

---

### Step 5 — Create Visualizations

| Task | Who handles it | How |
|---|---|---|
| Generate viz XML for each widget | ✅ Claude Code | Writes `.uaviz-meta.xml` files with correct SM, fields, and structure |
| Generate base64 visualSpecification | ✅ Claude Code | Python script to build and encode the JSON chart config |
| Deploy vizes to org | ✅ Claude Code | `sf project deploy start --metadata AnalyticsVisualization:Name` |
| Verify SM binding after deploy | ✅ Claude Code | Retrieve + `grep dataSource` — confirms no silent SM reversion |
| Handle SM reversion (new API name) | ✅ Claude Code | Detects reversion, generates new API name, redeploys automatically |
| Visual QA of chart appearance | 🖥️ UI required | Must open in browser to confirm chart renders correctly |

---

### Step 6 — Build the Dashboard

| Task | Who handles it | How |
|---|---|---|
| Generate dashboard XML | ✅ Claude Code | Writes `.uadash-meta.xml` with all widgets, layout, filters, styles |
| Calculate grid layout (columns/rows) | ✅ Claude Code | Converts pixel positions to 48-column grid coordinates |
| Apply dark theme | ✅ Claude Code | Injects hex color values into style JSON strings |
| Deploy dashboard | ✅ Claude Code | `sf project deploy start --metadata AnalyticsDashboard:Name` |
| Verify dashboard opens | 🖥️ UI required | Must open in browser to confirm no runtime errors |
| Fix XML comment runtime errors | ✅ Claude Code | Detects pattern, removes comments, redeploys with new API name |
| Fix filter widget issues | ✅ Claude Code | Changes `comboBox` to `toggle`, redeploys |

---

### Step 7 — Testing & Validation

| Task | Who handles it | How |
|---|---|---|
| Verify deploy succeeded | ✅ Claude Code | Reads CLI output — `Status: Succeeded` |
| Verify SM binding on all vizes | ✅ Claude Code | Batch retrieve + grep across all viz files |
| Check dashboard opens without error | 🖥️ UI required | Must be confirmed visually in browser |
| Validate numbers match CRM Analytics | 🖥️ UI required | Side-by-side comparison requires human judgement |
| Validate filter interactions | 🖥️ UI required | Filter behaviour must be tested manually |
| Diagnose "We couldn't find the Dashboard" | ✅ Claude Code | Systematically checks SM binding, XML structure, workspace registration |

---

### Step 8 — Version Control & CI/CD

| Task | Who handles it | How |
|---|---|---|
| Commit all files to Git | ✅ Claude Code | `git add`, `git commit`, `git push` |
| Create pull request | ✅ Claude Code | `mcp__github__create_pull_request` tool |
| Comment on PR | ✅ Claude Code | `mcp__github__comment_on_pull_request` tool |
| View PR checks | ✅ Claude Code | `mcp__github__get_pull_request_checks` tool |
| Diff review | ✅ Claude Code | `mcp__github__get_pull_request_diff` tool |
| Approve and merge | 🖥️ UI required | Human approval required |

---

## Full Automation Score

| Phase | Automatable | Requires UI |
|---|---|---|
| Audit CRM Analytics | ~80% | Screenshots, business context |
| Data Cloud setup | ~10% | DLO ingestion is entirely UI |
| Workspace creation | ~100% | — |
| Semantic Model creation | ~30% | SM creation + field setup is UI; field name discovery is automated |
| Visualization creation | ~90% | Visual QA only |
| Dashboard creation | ~85% | Open/validate only |
| Testing | ~50% | Visual validation, number matching |
| Version control | ~100% | Human PR approval |
| **Overall** | **~70%** | **~30% requires UI** |

---

## API Inventory — What's Available

### 1. Salesforce CLI (sf / sfdx) — ✅ Fully usable by Claude Code

The primary tool for all metadata operations.

```bash
# Deploy metadata
sf project deploy start --metadata "AnalyticsVisualization:Name" --target-org alias

# Retrieve metadata
sf project retrieve start --metadata "AnalyticsDashboard:Name" --target-org alias

# List orgs
sf org list

# Run SOQL
sf data query --query "SELECT ..." --target-org alias

# Open org in browser
sf org open --target-org alias
```

**Supported Tableau Next metadata types via CLI:**

| Metadata Type | Deploy | Retrieve | Notes |
|---|---|---|---|
| `AnalyticsWorkspace` | ✅ | ✅ | Full support |
| `AnalyticsDashboard` | ✅ | ✅ | Full support |
| `AnalyticsVisualization` | ✅ | ✅ | Full support — but SM reversion behaviour applies |
| `AnalyticsSemanticModel` | ❌ | ❌ | NOT in SFDX metadata registry |

**Supported CRM Analytics metadata types via CLI:**

| Metadata Type | Deploy | Retrieve | Notes |
|---|---|---|---|
| `WaveApplication` | ✅ | ✅ | Full support |
| `WaveDashboard` | ✅ | ✅ | Full support |
| `WaveLens` | ✅ | ✅ | Full support |
| `WaveDataset` | ✅ | ✅ | Full support |
| `WaveDataflow` | ✅ | ✅ | Full support |
| `WaveRecipe` | ✅ | ✅ | Full support |

---

### 2. Wave REST API — ⚠️ Partially available

Salesforce's Analytics REST API (`/services/data/vXX.0/wave/`).

**From our experience**: Several endpoints returned `FUNCTIONALITY_NOT_ENABLED`
for the `tableauNextOrg` user profile. Access varies by org configuration and
user permissions.

| Endpoint | Purpose | Status in our project |
|---|---|---|
| `GET /wave/datasets` | List all datasets | ✅ Available (CRM Analytics org) |
| `GET /wave/datasets/{id}` | Dataset details + field list | ✅ Available |
| `GET /wave/dashboards` | List dashboards | ✅ Available |
| `GET /wave/dashboards/{id}` | Full dashboard JSON | ✅ Available |
| `GET /wave/lenses` | List lenses | ✅ Available |
| `GET /wave/semanticModels` | List Semantic Models | ❌ `FUNCTIONALITY_NOT_ENABLED` |
| `GET /wave/semanticModels/{id}` | SM field details | ❌ `FUNCTIONALITY_NOT_ENABLED` |
| `POST /wave/query` | Run SAQL query | ✅ Available (CRM Analytics) |
| `GET /wave/workspaces` | List workspaces | ✅ Available (Tableau Next) |

**If Wave API SM endpoints were enabled**, Claude Code could fully automate
field name discovery by querying the SM definition directly — eliminating
the deploy-and-fail loop entirely.

---

### 3. Salesforce SOQL — ✅ Fully usable by Claude Code

```bash
sf data query \
  --query "SELECT Id, Name, Amount, CloseDate, StageName, Type,
           Owner.Name, Account.Name, Account.Type, Account.BillingCountry
           FROM Opportunity" \
  --target-org crmsAnalyticsAnchit
```

Used for:
- Auditing source data from the CRM Analytics org
- Confirming field names match what's in the SM
- Validating data volumes and values
- Cross-checking numbers between CRM Analytics and Tableau Next

---

### 4. GitHub API (via MCP tools) — ✅ Fully usable by Claude Code

Claude Code has access to these GitHub MCP tools:

| Tool | What it does |
|---|---|
| `mcp__github__create_pull_request` | Open a PR with title, body, base/head branch |
| `mcp__github__comment_on_pull_request` | Add review comments to a PR |
| `mcp__github__view_pull_request` | Read PR details, status, comments |
| `mcp__github__get_pull_request_diff` | Get the full diff of a PR |
| `mcp__github__get_pull_request_checks` | Read CI/CD check results on a PR |

This means Claude Code can manage the full Git/GitHub workflow:
write code → commit → push → open PR → read CI results → comment.

---

### 5. Google Cloud Storage (via MCP tools) — ✅ Available

| Tool | What it does |
|---|---|
| `mcp__gcs__pull_from_source` | Download reference files (screenshots, CSVs) from shared GCS bucket |
| `mcp__gcs__push_to_output` | Upload generated files (docs, exports) to session GCS bucket |
| `mcp__gcs__list_source` | List files in the source bucket |

Used in our project to:
- Download dashboard screenshots (`Olympus_Opportunities_CRM Analytics.png`)
- Compare target state vs current state

---

### 6. Salesforce Metadata API (via SFDX) — ✅ Available indirectly

All Metadata API calls go through the SFDX CLI.
Claude Code does not call the Metadata API directly but leverages it fully via `sf`.

Key operations Claude Code runs via SFDX:
```bash
# Bulk retrieve
sf project retrieve start --metadata "AnalyticsVisualization" --target-org alias

# Deploy with validation
sf project deploy start --dry-run --metadata "AnalyticsDashboard:Name"

# Check deploy status
sf project deploy report --job-id 0Afxx...
```

---

### 7. Data Cloud API — ⚠️ Limited access confirmed

Salesforce Data Cloud has its own REST API separate from the Wave API.

| Endpoint | Purpose | Access |
|---|---|---|
| `GET /services/data/vXX.0/ssot/objects` | List Data Lake Objects | Requires Data Cloud API permission |
| `GET /services/data/vXX.0/ssot/objects/{name}/fields` | List DLO fields | Requires Data Cloud API permission |
| `POST /services/data/vXX.0/ssot/query` | Run query against DLO | Requires Data Cloud API permission |

**If Data Cloud API access were enabled**, Claude Code could:
- Enumerate all DLO fields with their data types directly
- Skip the deploy-and-fail field discovery loop
- Build the complete field name mapping automatically

---

## The One Bottleneck: Semantic Model Field Names

The single biggest limitation for full automation is SM field name discovery.
Here is the comparison of what's possible today vs what would be possible
with full API access:

### Today (partial automation)
```
Claude deploys test viz with guessed field name
    ↓
Platform returns error: "F1 isn't a valid field in the semantic model"
    ↓
Claude increments suffix and redeploys
    ↓
Repeat until all fields confirmed
    ↓
Takes: 2-5 deploy iterations per field
```

### With Wave API SM endpoint enabled (full automation)
```
Claude calls GET /wave/semanticModels/{id}
    ↓
API returns full field list with exact names, types, and mappings
    ↓
Claude builds complete field map in seconds
    ↓
Deploys all vizes correctly on first attempt
    ↓
Takes: 1 API call, 0 failures
```

### With Data Cloud SSOT API enabled (full automation)
```
Claude calls GET /ssot/objects/Opportunity_Home1/fields
    ↓
API returns all DLO field names and types
    ↓
Claude cross-references with SM naming pattern
    ↓
Zero trial-and-error required
```

---

## What Claude Code Does Better Than Manual

| Task | Manual time | Claude Code time |
|---|---|---|
| Read and parse 10 CRM Analytics dashboard JSON files | 2-3 hours | 2 minutes |
| Generate 5 viz XML files with correct structure | 4-5 hours | 5 minutes |
| Discover 8 SM field names via deploy-and-fail | 1-2 hours | 15-20 minutes |
| Run SOQL and document all fields | 30 minutes | 2 minutes |
| Commit, push, open PR with description | 20 minutes | 2 minutes |
| Generate base64 visualSpecification for custom chart | 1-2 hours | 5 minutes |
| Detect and fix SM reversion on 5 vizes | 2-3 hours | 10 minutes |
| Write migration docs | 3-4 hours | 10 minutes |

---

## What Still Requires Human UI Interaction

| Task | Why UI is required | Workaround / Future |
|---|---|---|
| Create Semantic Model | `AnalyticsSemanticModel` not in SFDX registry | Salesforce could add it to the registry |
| Ingest data into Data Cloud | No CLI for DLO ingestion jobs | Data Cloud Bulk Ingestion API (preview) |
| Visual QA of dashboard | Numbers and layout need human eyes | Could use screenshot comparison tools |
| Validate filter interactions | Requires browser interaction | Could use Selenium/Playwright automation |
| Approve & merge PRs | Requires human governance | Intentionally human-gated |
| Authenticate new orgs | OAuth flow requires browser | `sf org login web` opens browser once |

---

## Recommended Workflow: Human + Claude Code

```
HUMAN DOES (30%)                    CLAUDE CODE DOES (70%)
─────────────────────────────────────────────────────────────
Screenshot CRM Analytics dashboard  Read and parse all CRM Analytics metadata
                                    Run SOQL to document all fields
                                    Generate field discovery test vizes
                                    Run deploy-and-fail field discovery loop
                                    Build complete field mapping table

Create Semantic Model in UI         Read workspace + SM metadata
Configure DLO connection            Verify SM is registered in workspace

                                    Generate all viz XML files
                                    Deploy all vizes
                                    Retrieve and verify SM binding
                                    Auto-fix any SM reversions (new API names)

                                    Generate dashboard XML
                                    Deploy dashboard
                                    Retrieve and validate dashboard metadata

Open dashboard in browser           Diagnose any errors automatically
Validate numbers & layout           Suggest fixes, redeploy

Approve PR                          Commit all files, push, open PR
                                    Generate migration documentation
                                    Write comparison guides
                                    Document field mappings
```

---

## Summary

Claude Code handles approximately **70% of the migration end-to-end** today.

The remaining 30% is blocked by:
1. **SM creation** — not available via SFDX (biggest gap)
2. **Data ingestion** — DLO setup requires UI
3. **Visual validation** — requires human eyes on the browser

If Salesforce adds `AnalyticsSemanticModel` to the SFDX registry and enables the
Wave API SM endpoints, Claude Code could handle **90%+ of the migration**
with humans only needed for final visual sign-off and PR approval.

---

*The DTC Sales and Olympus Opportunities migrations in this project demonstrated
that Claude Code is highly effective for the metadata engineering, field discovery,
and CI/CD aspects of Tableau Next migrations — the gaps are entirely in Salesforce
platform API coverage, not in Claude Code's capabilities.*
