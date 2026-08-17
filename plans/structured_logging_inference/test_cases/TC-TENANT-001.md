---
test_case_id: TC-TENANT-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TENANT-001: Tenant A cannot see Tenant B's logs

**Objective**: Verify that in a model-as-a-service deployment with
two tenants in separate namespaces, one tenant querying Loki
cannot access logs from the other tenant's namespace (AC 13).

**Preconditions**:
- MaaS deployment with two tenants configured in separate
  namespaces: `tenant-alpha-ns` and `tenant-beta-ns`
- Both tenants have active inference workloads emitting
  structured logs
- Loki multi-tenancy is configured with namespace-to-tenant ID
  mapping aligned with Kubernetes RBAC
- User accounts exist for each tenant with namespace-scoped
  permissions only

**Test Steps**:
1. Send inference requests as tenant-alpha to produce log entries
   in `tenant-alpha-ns`
2. Send inference requests as tenant-beta to produce log entries
   in `tenant-beta-ns`
3. Authenticate to the OpenShift Console as tenant-alpha's user
4. Navigate to Observe > Logs
5. Query Loki for all available log entries (no namespace filter)
6. Verify that all returned entries have
   `k8s.namespace.name=tenant-alpha-ns`
7. Explicitly query for logs from `tenant-beta-ns`:
   ```
   {k8s_namespace_name="tenant-beta-ns"}
   ```
8. Verify that zero entries are returned

**Expected Results**:
- Tenant-alpha's Loki query returns only log entries from
  `tenant-alpha-ns`
- Querying for `tenant-beta-ns` logs as tenant-alpha returns
  zero entries
- No log entries from `tenant-beta-ns` are visible to
  tenant-alpha under any query

**Notes**: To be filled later in the process.
