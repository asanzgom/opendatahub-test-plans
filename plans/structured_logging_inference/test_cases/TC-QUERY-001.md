---
test_case_id: TC-QUERY-001
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUERY-001: Query logs by service.name in OpenShift Console

**Objective**: Verify that filtering by `service.name` in the
OpenShift Console Observe > Logs interface returns only log entries
from the specified inference component.

**Preconditions**:
- LokiStack is deployed and receiving logs from the collection
  pipeline
- At least two inference components (e.g., vLLM and EPP) are
  running and emitting structured JSON logs
- Logs from both components are queryable in Loki

**Test Steps**:
1. Open the OpenShift Console and navigate to Observe > Logs
2. Enter a LogQL filter for `service.name` matching a specific
   inference component (e.g., `{service_name="vllm-runtime"}`)
3. Execute the query and review the returned log entries
4. Verify that every returned entry has `service.name` matching
   the filter value
5. Verify that no entries from other components (e.g., EPP, MaaS)
   appear in the results

**Expected Results**:
- All returned log entries contain `service.name` equal to the
  filtered value
- Zero log entries from other inference components appear in the
  result set
- The Console renders the filtered results without errors

**Notes**: To be filled later in the process.
