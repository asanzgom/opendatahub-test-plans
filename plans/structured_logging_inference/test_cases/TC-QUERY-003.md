---
test_case_id: TC-QUERY-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUERY-003: Query logs by severity level

**Objective**: Verify that filtering by severity in the OpenShift
Console Observe > Logs interface returns only log entries at the
specified severity level.

**Preconditions**:
- At least one inference component is running and emitting
  structured JSON logs at multiple severity levels (e.g., INFO,
  WARNING, ERROR)
- Logs are collected and queryable in Loki

**Test Steps**:
1. Trigger inference workloads that produce logs at different
   severity levels (e.g., a successful request for INFO, an
   invalid request for ERROR)
2. Open OpenShift Console Observe > Logs
3. Filter by severity `ERROR`
4. Verify that all returned entries have `severity` equal to
   `ERROR`
5. Change the filter to severity `INFO`
6. Verify that all returned entries have `severity` equal to
   `INFO` and no ERROR entries appear

**Expected Results**:
- Filtering by `ERROR` returns only entries with
  `severity=ERROR`
- Filtering by `INFO` returns only entries with `severity=INFO`
- No entries with mismatched severity appear in either result set

**Notes**: To be filled later in the process.
