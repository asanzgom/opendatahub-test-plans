---
test_case_id: TC-TENANT-002
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TENANT-002: Namespace admin sees all inference logs in
their namespace

**Objective**: Verify that a namespace admin with llm-d-as-a-
service tenancy has full visibility to all inference logs
(vLLM, EPP, controller) within their namespace.

**Preconditions**:
- llm-d-as-a-service deployment in namespace `llmd-tenant-ns`
- Namespace admin user with RBAC scoped to `llmd-tenant-ns`
- Inference workloads (vLLM, EPP) running and emitting
  structured JSON logs in the namespace
- Logs are collected and queryable in Loki

**Test Steps**:
1. Send multiple inference requests to generate logs from vLLM
   and EPP within `llmd-tenant-ns`
2. Authenticate to the OpenShift Console as the namespace admin
3. Navigate to Observe > Logs
4. Query for all logs in `llmd-tenant-ns`:
   ```
   {k8s_namespace_name="llmd-tenant-ns"}
   ```
5. Verify that log entries appear from multiple components
   (vLLM, EPP) by checking distinct `service.name` values
6. Verify that queue lifecycle logs, routing decision logs, and
   request completion logs are all visible

**Expected Results**:
- Namespace admin can query and view all inference component logs
  within their namespace
- Log entries from vLLM (request lifecycle, queue events) are
  visible
- Log entries from EPP (routing decisions) are visible
- Token counts, trace IDs, and severity levels are all visible
  in the log entries

**Notes**: To be filled later in the process.
