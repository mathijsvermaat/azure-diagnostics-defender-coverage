# Azure Log Coverage

[AzureLogCoverage-DedicatedTables.json](AzureLogCoverage-DedicatedTables.json) is an Azure Monitor Workbook for comparing resource inventory with observed Log Analytics data. The **Defender Workloads & Logs** tab adds Defender for Cloud subscription configuration alongside that evidence, with logs as the primary grouping.

## Use the workbook

1. In the **Microsoft Defender portal**, open **Microsoft Sentinel > Workbooks > Add workbook**, then select **Edit**.
2. Open **Advanced editor** and replace its contents with the workbook JSON.
3. Select **Apply**, then **Done Editing**. Select the management groups, subscriptions, and Log Analytics workspaces to assess. The preview remains unsaved until you explicitly save it.
4. Choose a lookback period. The default is 48 hours; begin with a short interval for larger environments.
5. Review query errors, warnings, and truncation before interpreting missing evidence. Use the Documentation control for detailed guidance.

Read access to the selected resources and Defender pricing configuration (`Microsoft.Security/pricings/read`), plus permission to query the selected workspaces and tables, is required. Microsoft Sentinel is optional for the coverage queries. Existing Sentinel-specific navigation requires a Sentinel workspace.

The queries are read-only. Diagnostic settings, collection rules, Defender plans, and policies are not changed automatically. Existing navigation/onboarding actions remain separate from the evidence checks.

## Views

| Tab | Assessment |
| --- | --- |
| Defender Workloads & Logs | Resource logs, investigation signals, and Defender subscription configuration |
| Activity Logs | Subscription-level `AzureActivity` ingestion |
| Network Flow Logs | NSG/VNet flow-log configuration and Traffic Analytics evidence |
| Virtual Machines | Azure/Arc inventory, extensions, and agent heartbeat |
| Microsoft Entra ID | Identity log sources |
| Workspace Audit | Log Analytics query audit evidence |

The existing five separate views are unchanged by the Defender enhancement. In particular, heartbeat is **not** guest security-log or SQL-audit coverage, and Traffic Analytics is **not** proof of raw DNS-query capture.

## Defender and logs

The resource grid is grouped first by **Log evidence**, then Defender plan and subscription. It preserves resources with no matching logs and includes disabled or unknown plans.

The **Resource type** picker shows alphabetically sorted friendly labels such as **Key Vault**, **Storage accounts**, and **Kubernetes (AKS)**. Selection still filters by the original Azure resource-type IDs. Multi-select and **All** are retained. **App Service apps** includes Function Apps and Logic Apps Standard, which share the same Azure resource type.

The first visible columns in the combined grid are **Resource**, **Diagnostic settings**, and **Defender + Logs**. **Open settings** opens the resource's diagnostic-settings blade, including for resources without observed logs; it does not enable logging. **Defender + Logs** displays the traffic-light `Coverage status`, derived from the merged resource inventory and Log Analytics evidence. The longer calculated `Comparison` text remains in the underlying dataset as a hidden helper.

### Summary and views

The workload tab now presents these views in order:

1. **Coverage statistics:** three statistics-only pie charts, ordered **both logs and Defender plan enabled / total assets**, **logs observed / total assets**, and **Defender subscription plan enabled / total assets**. Each complete pie represents the same assessed-asset total. Legends and metrics distinguish covered, not covered, and (where applicable) unknown configuration. There is no expandable summary table.
2. **Combined resource coverage:** resource-level log evidence alongside subscription configuration, with traffic-light icons.
3. **Resource log coverage:** a separate logs-only grid with observed/missing evidence, tables, categories, investigation signals, last seen, record counts, source workspaces, and diagnostic-settings navigation. Defender columns are omitted from this view.
4. **Defender subscription plans:** the existing subscription configuration overview, including plans not represented by resource rows.

The denominator is the number of returned supported resource IDs under the selected subscriptions and resource types, not every asset in the tenant. Database and slot child resources count separately. Multiple records, tables, or source workspaces do not multiply a resource's contribution. Log evidence follows the selected workspaces and lookback. An empty inventory shows a not-applicable message, not 100% coverage.

The pies reuse the existing inventory and log query results without another source query. Each asset contributes a single numeric `Assets = 1` value, grouped explicitly by `Both outcome`, `Log evidence`, or `Subscription plan`. The native chart sums these values into category slices and displays their total in the donut center. Unknown plan states remain a separate category. The total is never an extra slice alongside covered assets. Explicit grouping prevents Defender from choosing resource IDs as slices or showing only the first numeric coverage flag. Their totals are independent of filtering either resource grid. Query failures, partial results, and resource/query row limits can invalidate these totals; review warnings before interpreting the visuals.

### Traffic lights

| Color | Observed logs | Subscription plan | Interpretation |
| --- | --- | --- | --- |
| Green - Both | Yes | Enabled | Both signals present |
| Orange - Partial | Yes | Disabled | Logs only at this comparison scope |
| Orange - Partial | No | Enabled | Enabled subscription plan only |
| Red - Neither | No | Disabled | Neither signal present in this comparison |
| Gray - Unknown | Either | Unknown | Plan configuration is unresolved |

Use the combined grid's built-in filter with **Green**, **Orange**, **Red**, or **Gray** to filter by status. Color names are part of the underlying visible values, so the filter does not depend on recognizing the icon. Clear the filter to restore all rows. Filtering the combined grid does not change the summary or the separate logs and Defender views.

These colors are not verified protection-health ratings. In particular, red means no observed Log Analytics records and a disabled subscription plan, not proof that no logging, resource-level protection, or other security visibility exists. Unknown plan states are never colored red or green.

| Subscription plan | Logs observed | Meaning |
| --- | --- | --- |
| Enabled | Yes | Subscription plan enabled; resource-attributed logs observed |
| Enabled | No | Subscription plan enabled; logs not observed in the selected scope |
| Disabled | Yes | Subscription plan disabled; resource-attributed logs observed |
| Disabled | No | Subscription plan disabled; logs not observed in the selected scope |
| Unknown | Either | Missing or unrecognized plan configuration; log evidence remains independent |

**These are configuration comparisons, not effective protection certifications.** Standard pricing maps to Enabled, Free to Disabled, and missing/unrecognized pricing to Unknown. Only subscription-level pricing records are used; resource-level records cannot overwrite that subscription state.

Resource protection is explicitly **Not verified**. Resource overrides, exclusions, plan components, eligible SKUs/regions, API onboarding, and sensor health require separate checks. An enabled subscription is not sufficient to establish all of these conditions. Conversely, subscription Free does not exclude resource-level enablement or protection through another product.

Neither missing logs nor a disabled subscription plan establishes an absolute absence of security visibility. Other workspaces, destinations, Defender/XDR evidence, and quiet workloads can change the interpretation. Raw logs alone also do not supply Defender's detections.

### Evidence columns

- **Tables / Categories:** observed shared and resource-specific sources. Category falls back to a known mapping or table name when unavailable.
- **Investigation signals:** recognized audit/access evidence, distinct from generic log presence. An empty set does not establish that all other records are useless.
- **Last seen:** latest observed `TimeGenerated`, not ingestion time.
- **Records:** records in the selected window, without cross-table/workspace event deduplication.
- **Workspaces:** source `TenantId` workspace identifiers where available; not workspace display names.
- **Defender plan / Subscription plan / Subplan:** the mapped workload plan and subscription configuration.
- **Verification needed:** workload-specific limitations and checks before concluding protection or investigation readiness.
- **Diagnostic settings:** a navigation action, not an enabled-state indicator.

Resource identity is matched using the full normalized ARM ID. Only Storage Blob/File/Queue/Table service IDs roll up to their storage account. Databases, deployment slots, and clusters retain their own identities; same-named resources in different groups or subscriptions cannot match solely by name.

## Scope

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

The **subscription plan overview** lists all **12 current workload plan families**, plus **legacy standalone DNS**, for the selected subscriptions. It is independent of the resource-type selector and does not hide plans just because no corresponding resource is inventoried. Besides the nine families above, it includes:

- **Servers P1/P2:** Azure/Arc heartbeat remains in Virtual Machines; guest security/event/Syslog coverage is not assessed here.
- **SQL Servers on Machines:** configuration is shown; instance-level protection and guest SQL auditing are not assessed.
- **Resource Manager:** control-plane ingestion remains in Activity Logs.
- **DNS:** the row describes the legacy standalone plan only. DNS protection can also be bundled with Servers P2; Free/Unknown in the standalone row does not negate bundled protection. No raw DNS-query assessment is performed.

AWS/GCP connector inventory is not queried. Arc resources are evaluated in their Azure subscription context, including directly connected Kubernetes clusters. CSPM and DevOps posture are excluded from the workload-plan overview. AI model-registry scanning is not assessed.

Compared with the earlier general-purpose PaaS inventory, resource types decreased from **29 to 16**: AI accounts and Arc-connected Kubernetes were added, and these 15 types were removed from the resource grid:

- CDN profiles (including Front Door Standard/Premium), classic Storage accounts, Data Factory, IoT Hub, and Event Hubs namespaces.
- Application Gateway, Azure Firewall, Bastion, classic Front Door, network security groups, public IP addresses, and Traffic Manager.
- Purview accounts, Recovery Services vaults, and Service Bus namespaces.

Named log sources decreased from **101 to 48**: 55 dedicated sources for those removed families were removed, while `ContainerLog` and `ContainerLogV2` were added. `AzureDiagnostics` remains a shared source and can still contain records from excluded families, but those resources do not appear in the merged inventory. This does not imply their logs are unimportant; they are outside the selected Defender workload-plan comparison. The other five tabs were preserved, including the separate Network Flow Logs view.

## Limits and validation

- Observed records do not certify every category, endpoint, Storage service, investigation requirement, or retention period.
- Diagnostic-setting and DCR configuration is not inspected. Storage/Event Hubs destinations and Defender XDR hunting data are outside the selected Log Analytics evidence scope.
- An empty typed source and fuzzy union tolerate absent tables, but not every permission, workspace, or execution failure. Failed/partial queries invalidate negative conclusions.
- Azure Resource Graph permissions, result limits, and eventual consistency apply. Grids are limited to 10,000 rows, and service limits can apply earlier. Narrow the scope when results are truncated.
- Basic/Auxiliary scope and query charges apply; Basic-only queries are limited to the past 30 days.
- The local development copy has an intentionally untracked PowerShell regression suite. It validates JSON, embedded merges, mappings, evidence contracts, and identity/state models, not live KQL/ARG execution.
- Read-only browser testing in the Microsoft Defender portal on 10 November 2026 verified the three repaired donut charts against a live resource grid. One subscription and one workspace over 48 hours returned 28 assets: 9 with observed logs, 9 with both signals, and 28 with an enabled subscription plan. These are test-scope results, not a coverage claim for other environments.
- All five other tabs rendered their grids in that smoke test; Entra displayed all 11 reference tables. The Defender subscription overview returned 13 plan rows after retrying a transient "No resources selected" error following JSON import. Retry or refresh a failed control and review its result before interpreting coverage.
- Disabled/unknown plan partitions and empty/all/none-covered scopes are covered by local fixtures, not demonstrated by the live scope. Multi-subscription/workspace behavior, restricted permissions, every drilldown/action, and every supported resource type still require representative testing. No workbook was saved and no cloud settings were changed during validation.
- VS Code reports the existing untrusted remote workbook-schema warning; parsing JSON is not full schema validation.

Before operational use, validate a representative subscription with enabled and disabled plans, missing configuration, both log formats, resources without recent events, API onboarding differences, Arc workloads, and restricted permissions. Confirm grid links, multi-workspace/multi-subscription selection, query warnings, and last-seen values in the portal.

Only the workbook JSON and this README are intended for version control. Local source snapshots, tests, historical notes, and editor settings are not distributed as part of the repository.

## Microsoft Learn references

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