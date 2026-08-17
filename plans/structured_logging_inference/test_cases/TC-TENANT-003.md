---
test_case_id: TC-TENANT-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TENANT-003: Platform team has cross-namespace log access

**Objective**: Verify that a platform team user with cluster-wide
permissions can query logs across all inference namespaces for
SLA breach investigation in a model-as-a-service deployment.

**Preconditions**:
- MaaS deployment with at least two tenant namespaces:
  `tenant-alpha-ns` and `tenant-beta-ns`
- A platform team user with cluster-scoped RBAC for log access
- Inference workloads running in both namespaces
- Logs from both namespaces are collected and queryable in Loki

**Test Steps**:
1. Send inference requests in both `tenant-alpha-ns` and
   `tenant-beta-ns` to generate log entries
2. Authenticate to the OpenShift Console as the platform team
   user
3. Navigate to Observe > Logs
4. Query for logs across both namespaces:
   ```
   {k8s_namespace_name=~"tenant-alpha-ns|tenant-beta-ns"}
   ```
5. Verify that log entries from both namespaces are returned
6. Query for a specific trace ID that spans components in
   `tenant-alpha-ns`
7. Verify that all correlated log entries are visible

**Expected Results**:
- Platform team user can query and view log entries from both
  `tenant-alpha-ns` and `tenant-beta-ns` in a single query
- Cross-namespace log access includes all inference component
  logs (vLLM, EPP, MaaS) from both namespaces
- The platform team user can correlate logs by `trace_id` across
  namespaces for SLA breach investigation

**Notes**: To be filled later in the process.
