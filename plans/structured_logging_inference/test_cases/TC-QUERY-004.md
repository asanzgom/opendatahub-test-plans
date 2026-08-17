---
test_case_id: TC-QUERY-004
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUERY-004: Query logs by time range

**Objective**: Verify that filtering by time range in the OpenShift
Console Observe > Logs interface returns only log entries within
the specified window.

**Preconditions**:
- At least one inference component is running and emitting
  structured JSON logs continuously
- Logs are collected and queryable in Loki

**Test Steps**:
1. Note the current timestamp T1
2. Send several inference requests over a 2-minute window
3. Note the ending timestamp T2
4. Wait 2 additional minutes, then send another batch of requests
   (producing logs after T2)
5. Open OpenShift Console Observe > Logs
6. Set the time range filter to [T1, T2]
7. Review the returned log entries
8. Verify that every entry has a `timestamp` within the [T1, T2]
   window
9. Verify that no entries from after T2 appear

**Expected Results**:
- All returned log entries have `timestamp` values within the
  specified [T1, T2] range
- Zero log entries from outside the time range appear in the
  result set

**Notes**: To be filled later in the process.
