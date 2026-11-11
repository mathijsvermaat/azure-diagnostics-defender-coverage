# Azure Log Coverage

## Overview

[AzureLogCoverage-DedicatedTables.json](AzureLogCoverage-DedicatedTables.json) helps security and operations teams identify potential Azure logging gaps by comparing resource inventory, observed Log Analytics records, and Microsoft Defender for Cloud subscription plan configuration.

The main **Defender Workloads & Logs** view deliberately focuses on **16 supported Azure/Arc resource types mapped to nine workload plans**, including resources whose plans are disabled or unknown. Separate tabs assess VM heartbeats, Azure Activity Logs, Network Flow Logs, Microsoft Entra ID logs, and workspace query auditing. This is not a universal Azure logging inventory.

> [!NOTE]
> Use this workbook alongside the [Sentinel Maturity Model](https://github.com/mathijsvermaat/Sentinel-Maturity) to identify Azure workload logging gaps and compare observed evidence with Defender subscription configuration. Findings support connector-onboarding and investigation-readiness assessments; they do not assign a maturity tier, validate retention, or certify effective protection.

## Key features and use cases

- **Coverage comparison:** distinguish resources with both observed logs and an enabled Defender subscription plan, one signal, neither signal, or unknown plan configuration.
- **Shared and dedicated logs:** check `AzureDiagnostics` together with supported resource-specific tables, rather than assuming only one format is used.
- **Investigation evidence:** review observed tables, categories, audit/access signals, timestamps, record counts, and source workspaces.
- **Coverage statistics:** compare three asset-based summaries with resource-level combined and logs-only views.
- **Interactive filtering:** select subscriptions, workspaces, lookback, and friendly resource-type labels; filter detail rows by coverage status.
- **Guidance and follow-up:** use **Show help** for scope and interpretation, or hide the text for a compact view. Diagnostic-settings links open configuration for review without enabling logging.

## Prerequisites

- Read access to the Azure resources and subscriptions being assessed.
- Read access to Defender pricing configuration, including `Microsoft.Security/pricings/read`.
- Permission to query the selected Log Analytics workspaces and relevant tables.
- Logs already collected in those workspaces. Importing the workbook does not enable collection.
- For the deployment path below, access to Microsoft Sentinel Workbooks in the [Microsoft Defender portal](https://security.microsoft.com/). The coverage queries themselves do not require Sentinel enablement; Sentinel-specific connector navigation requires a Sentinel-enabled workspace.

The assessment queries are **read-only**. They do not change diagnostic settings, Data Collection Rules (DCRs), Defender plans, or policies. Navigation and onboarding actions are separate from the evidence checks.

## Deployment and first use

1. In the **Microsoft Defender portal**, open **Microsoft Sentinel > Workbooks > Add workbook**, then select **Edit**.
2. Open **Advanced editor** and replace its contents with [AzureLogCoverage-DedicatedTables.json](AzureLogCoverage-DedicatedTables.json).
3. Select **Apply**, then **Done Editing**.
4. Select the management groups and subscriptions to inventory. Explicitly select the Log Analytics workspaces containing the expected evidence; do not assume the default **Any one** selection is the intended workspace.
5. Choose the lookback and resource types. Start with one known subscription and workspace to validate the results.
6. Confirm the statistics and detail views load. Review errors, partial results, and truncation before interpreting missing evidence.
7. Save the workbook when satisfied with the assessment scope. The preview remains unsaved until you explicitly save it.

### Initial filters

| Control | Purpose |
| --- | --- |
| Management groups / Subscriptions | Define the Azure inventory scope. |
| Workspace scope | Choose Sentinel-enabled workspaces or all eligible Log Analytics workspaces for the picker. |
| Log Analytics workspaces | Select where to look for evidence; this is separate from the resource inventory scope. |
| Lookback period | Defaults to **48 hours**. Begin with a short interval for larger environments. |
| Resource type | Select one or more friendly labels, such as **Key Vault** or **Storage accounts**. **All** includes supported types present in scope. |
| Show help | Defaults to **Show**; use **Hide** for compact mode. |

### Help and compact mode

**Show help** appears before the introduction. Its icon-led sections, tables, warning callouts, and examples explain the scope and evidence. Select **Hide** to remove all explanatory Markdown across all six tabs, including headings, coverage notes, action descriptions, and references.

Scope controls, tabs, chart titles, legends, grids, action links, and runtime errors remain available. Help visibility does not change queries or scope selections. Select **Show** to restore the guidance.

## Understanding the results

### Workbook views

| Tab | Assessment |
| --- | --- |
| Defender Workloads & Logs | Resource logs, investigation signals, and Defender subscription configuration |
| Activity Logs | Subscription-level `AzureActivity` ingestion |
| Network Flow Logs | NSG/VNet flow-log configuration and Traffic Analytics evidence |
| Virtual Machines | Azure/Arc inventory, extensions, and agent heartbeat |
| Microsoft Entra ID | Identity log sources |
| Workspace Audit | Log Analytics query audit evidence |

The five companion tabs retain their own inventories and evidence checks. Heartbeat is **not** guest security-log or SQL-audit coverage, and Traffic Analytics is **not** proof of raw DNS-query capture.

### Coverage statistics and detail views

The workload tab presents these views in order:

1. **Coverage statistics:** three donut charts showing **both logs and an enabled Defender plan**, **logs observed**, and **Defender subscription plan enabled**, each against the same assessed-asset total.
2. **Combined resource coverage:** resource-level log evidence alongside subscription configuration, with traffic-light status. Rows are grouped by log evidence, Defender plan, and subscription.
3. **Resource log coverage:** a logs-only grid with sources, investigation signals, timestamps, record counts, workspaces, and diagnostic-settings navigation.
4. **Defender subscription plans:** subscription configuration, including plan families not represented in the resource grid.

The denominator counts **returned supported resource IDs**, not log records or every tenant asset. Database and slot child resources count separately; multiple tables, records, or workspaces do not multiply a resource's contribution. Unknown plan states remain explicit. An empty inventory is not applicable, not 100% coverage.

Subscription and resource-type filters define the resource inventory; workspace and lookback selections define the log evidence. Filtering a detail grid does not change the summary totals.

### Coverage categories

| Color | Observed logs | Subscription plan | Interpretation |
| --- | --- | --- | --- |
| Green - Both | Yes | Enabled | Both signals present |
| Orange - Partial | Yes | Disabled | Logs only at this comparison scope |
| Orange - Partial | Not observed | Enabled | Enabled subscription plan only |
| Red - Neither | Not observed | Disabled | Neither signal present in this comparison |
| Gray - Unknown | Either | Unknown | Plan configuration is unresolved |

Use the combined grid's built-in filter with **Green**, **Orange**, **Red**, or **Gray**. Color names accompany the icons, so interpretation does not depend on color alone. Clear the filter to restore all rows; the separate logs and Defender views are unaffected.

### Configuration versus protection and evidence

> [!IMPORTANT]
> These colors are configuration/evidence comparisons, not protection-health or maturity ratings. Red does not prove that no logging, resource-level protection, or other security visibility exists. Unknown is never treated as Disabled.

Defender **Standard** pricing maps to Enabled, **Free** to Disabled, and missing/unrecognized pricing to Unknown. Only subscription-level pricing records are used; resource-level records cannot overwrite that subscription state.

Resource protection remains **Not verified**. Overrides, exclusions, plan components, eligible SKUs/regions, API onboarding, and sensor health require separate checks. Subscription Free does not exclude resource-level enablement or protection through another product.

Observed records do not establish complete category coverage or sufficient retention. Missing records can reflect a quiet resource, another workspace/destination, unsupported sources, or access issues. Raw logs alone do not supply Defender's detections, and Defender/XDR evidence may exist outside Log Analytics.

For example, a **Key Vault** with an enabled plan but no observed audit records has a different investigation-evidence position from a vault with both signals. Neither the plan setting nor one record proves complete protection or logging coverage.

### Evidence columns

| Column | Meaning |
| --- | --- |
| Tables / Categories | Observed shared and resource-specific sources; category falls back to a known mapping or table name when unavailable. |
| Investigation signals | Recognized audit/access evidence, distinct from generic log presence. An empty set does not mean all other records are useless. |
| Last seen | Latest observed `TimeGenerated`, not ingestion time. |
| Records | Records in the selected window, without cross-table/workspace event deduplication. |
| Workspaces | Source `TenantId` workspace identifiers where available, not display names. |
| Defender plan / Subscription plan / Subplan | Mapped workload plan and subscription configuration. |
| Verification needed | Workload-specific checks before concluding protection or investigation readiness. |
| Diagnostic settings | **Open settings** navigates to the resource's diagnostic-settings blade; it is not an enabled-state indicator and does not enable logging. |

## Troubleshooting and limitations

| Symptom | Checks and next steps |
| --- | --- |
| Empty resource inventory | Check subscriptions, permissions, and supported resource types. An empty result is not full coverage. |
| Logs not observed | Confirm the intended workspaces and lookback, recent resource activity, collection configuration, and supported tables before concluding logging is disabled. |
| Unknown Defender configuration | Check pricing read access and configuration availability. Do not reinterpret Unknown as Free. |
| No resources selected after import | Explicitly select the scope, allow parameter lookups to resolve, and retry or refresh the failed control. Review the successful result before interpreting coverage. |
| Query errors, missing permissions, or partial results | Resolve the errors first. An empty typed source and fuzzy union tolerate absent tables, but not every execution or access failure. |
| Truncated or unexpectedly small results | Narrow the scope. Grids have a 10,000-row limit; service limits and permission-scoped inventory can restrict results earlier. |
| Basic/Auxiliary table restrictions | Check supported query scope and charges. Basic-only queries are limited to the past 30 days. |

Diagnostic-setting and DCR configuration is **not inspected**. Other log destinations, Defender XDR hunting data, category completeness, and retention sufficiency are outside the evidence check. The workbook does not validate Defender for Cloud alert delivery to Sentinel. Resource Graph eventual consistency also applies.

### Validation status

Read-only Defender portal testing on 10-11 November 2026 exercised all six tabs, the three statistics charts, resource filtering, and Show/Hide guidance. One subscription and workspace over 48 hours returned 28 assets: 9 with observed logs, 9 with both signals, and 28 with an enabled plan. The local development suite passes 363 contract/model checks, including disabled/unknown and empty-inventory cases; it is not distributed with the workbook and does not execute live KQL/ARG.

> [!WARNING]
> Validate against representative data before operational use. Multi-subscription/workspace behavior, restricted permissions, every supported resource type, and every navigation action still require representative testing. A successful lab preview is not broad production validation.

## How it works / technical details

### Data and correlation

| Source | Role |
| --- | --- |
| Azure Resource Graph | Resource/subscription inventory, VM extensions, workspace metadata, and Network Watcher flow-log configuration. |
| Subscription-level `Microsoft.Security/pricings` | Defender plan configuration returned through Resource Graph. |
| `AzureDiagnostics` and supported dedicated tables | Resource-attributed log evidence for the workload comparison. |
| `Heartbeat`, `AzureActivity`, Entra reference tables, `LAQueryLogs` | Evidence for the separate VM, Activity, Entra, and Workspace Audit tabs. |
| `AzureNetworkAnalytics_CL` and `NTANetAnalytics` | Legacy/current Traffic Analytics evidence for Network Flow Logs. |

Resource identity is matched using the **full normalized ARM ID**. Only Storage Blob/File/Queue/Table service IDs roll up to their storage account. Databases, deployment slots, and clusters retain their own identities; same-named resources in different groups or subscriptions cannot match solely by name. A left outer merge preserves inventory rows without matching logs.

The three charts reuse the inventory/log merge. Each resource contributes `Assets = 1`, grouped by `Both outcome`, `Log evidence`, or `Subscription plan`. The donut center shows the assessed total, not an extra slice. Detail-grid filtering does not change that denominator.

### Supported workloads

The resource grid contains **16 Azure/Arc resource types**, associated with nine workload plan families. It queries **48 named log sources**: `AzureDiagnostics` plus 47 other tables, including `ContainerLog` and `ContainerLogV2`. Both shared and dedicated formats are checked together.

| Workload plan | Resource inventory | Evidence and boundaries |
| --- | --- | --- |
| Storage | Storage accounts | Blob, File, Queue, and Table logs. Queue/Table records do not establish Blob/Files visibility. |
| Key Vault | Vaults | Shared/dedicated logs; AuditEvent is distinguished from policy evaluation. |
| App Service | Sites and slots, including Function Apps and Logic Apps Standard | HTTP, audit, application, console, platform, Functions, and workflow logs. Hosting-model eligibility is not certified. |
| Containers | AKS, Arc-connected Kubernetes, and Container Registry | AKS audit/control-plane, Container Insights workload logs, and registry logs. Arc control-plane audit collection and runtime sensor health are not assessed. Registry scanning is separate and can also be available through CSPM. |
| Azure SQL Databases | SQL servers/databases and Managed Instances/databases | AzureDiagnostics, with SQLSecurityAuditEvents distinguished from other diagnostics. Parent and database IDs are not conflated. |
| Open-Source Relational Databases | PostgreSQL/MySQL Flexible Servers | Dedicated/shared logs; slow-query and server logs do not establish complete auditing or pgAudit configuration. |
| Azure Cosmos DB | Database accounts | Request and diagnostic sources; statistics alone do not prove auditing. API-specific protection eligibility remains unverified. |
| APIs | API Management services | Gateway, developer portal, WebSocket, LLM, and MCP logs. Service-level evidence does not establish individual API onboarding or logging coverage. |
| AI Services | Cognitive Services accounts of kind OpenAI or AIServices | AzureDiagnostics audit/request categories. Metadata is not proof of prompt/response capture or model protection. |

The resource-type picker uses friendly labels while retaining the original Azure type IDs for filtering. **App Service apps** includes Function Apps and Logic Apps Standard, which share the same resource type.

### Subscription plan overview and scope boundaries

The **subscription plan overview** lists all **12 current workload plan families**, plus **legacy standalone DNS**, for the selected subscriptions. It is independent of the resource-type selector and does not hide plans just because no corresponding resource is inventoried. Besides the nine families above, it includes:

- **Servers P1/P2:** Azure/Arc heartbeat remains in Virtual Machines; guest security/event/Syslog coverage is not assessed here.
- **SQL Servers on Machines:** configuration is shown; instance-level protection and guest SQL auditing are not assessed.
- **Resource Manager:** control-plane ingestion remains in Activity Logs.
- **DNS:** the row describes the legacy standalone plan only. DNS protection can also be bundled with Servers P2; Free/Unknown in the standalone row does not negate bundled protection. No raw DNS-query assessment is performed.

AWS/GCP connector inventory is not queried. Arc resources are evaluated in their Azure subscription context, including directly connected Kubernetes clusters. CSPM and DevOps posture are excluded from the workload-plan overview. AI model-registry scanning is not assessed.

Other general-purpose Azure services, such as Firewall, Application Gateway, Data Factory, messaging, Purview, and Recovery Services, are outside the main workload inventory. `AzureDiagnostics` can still contain records from excluded families, but those resources do not appear in the merged resource view. Their absence is a scope decision, not a statement that their logs are unimportant.

### Portability

Only the workbook JSON and this README are intended for version control. Local source snapshots, tests, historical notes, and editor settings are not distributed as part of the repository.

Use the repository JSON as the portable template. Before sharing a portal-exported copy, review it for saved subscription/workspace selections and tenant-specific bindings. Internal workbook control and merge IDs are not tenant bindings.

## Related resources

- [Sentinel Maturity Model](https://github.com/mathijsvermaat/Sentinel-Maturity): guidance on connector priorities, retention, and detection rationale to use alongside this evidence assessment.
- [Layered Detection Approach](https://github.com/mathijsvermaat/Sentinel-Maturity/blob/main/guidance/layered-detection.md): the rationale for retaining independent log evidence alongside security-product detections.
- [Forensic Readiness](https://github.com/mathijsvermaat/Sentinel-Maturity/blob/main/guidance/forensic-readiness.md): investigation requirements that extend beyond seeing a recent log record.
- [Sentinel Maturity Assessment Checklist](https://mathijsvermaat.github.io/sentinel-maturity-assessment.html): review findings against relevant assessment items and record follow-up manually. This workbook does not populate the checklist or calculate its maturity results.
- [Defender AMA Coverage](https://github.com/mathijsvermaat/Defender-AMA-coverage): a complementary endpoint-focused workbook for Defender for Endpoint, AMA telemetry, and DCR correlation; its endpoint categories are not interchangeable with this workbook's workload-plan comparison.

### Microsoft Learn references

- [Parameter-driven conditional visibility](https://learn.microsoft.com/azure/azure-monitor/visualize/workbooks-interactive-reports#set-conditional-visibility)
- [Workbook pie charts and chart metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-chart-visualizations#pie-charts)
- [Workbook data sources and merge behavior](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-data-sources#merge)
- [Reusing query results in different visualizations](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-interactive-reports#reuse-query-data-in-different-visualizations)
- [Defender for Cloud plans](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-cloud-introduction)
- [Defender plan configuration in Resource Graph](https://learn.microsoft.com/azure/defender-for-cloud/resource-graph-samples)
- [Pricing scope and inheritance](https://learn.microsoft.com/rest/api/defenderforcloud/pricings/get?view=rest-defenderforcloud-2024-01-01)
- [Key Vault investigation guidance](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-key-vault-introduction)
- [Storage protection and account exclusions](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-storage-introduction)
- [Container deployment and Azure Arc](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-containers-deployment-planning)
- [Defender for APIs onboarding and evidence](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-apis-introduction)
- [AI protection capabilities](https://learn.microsoft.com/azure/defender-for-cloud/ai-threat-protection)
- [AI account log categories](https://learn.microsoft.com/azure/azure-monitor/reference/supported-logs/microsoft-cognitiveservices-accounts-logs)
- [Legacy and bundled DNS protection](https://learn.microsoft.com/azure/defender-for-cloud/defender-for-dns-introduction)