---
test_case_id: TC-QUERY-005
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUERY-005: Free text search in logs

**Objective**: Verify that free text search in the OpenShift
Console Observe > Logs interface returns log entries whose body
contains the search term.

**Preconditions**:
- At least one inference component is running and emitting
  structured JSON logs
- Logs are collected and queryable in Loki

**Test Steps**:
1. Send an inference request that produces a log entry with a
   known, unique string in the body (e.g., a specific model name
   like `granite-3b-code-instruct`)
2. Open OpenShift Console Observe > Logs
3. Enter the unique string as a free text search term
4. Execute the query
5. Verify that returned entries contain the search term in their
   `body` field
6. Verify that entries not containing the term are excluded

**Expected Results**:
- All returned log entries contain the searched string in the
  `body` field
- Entries that do not contain the search term are not present in
  the result set

**Notes**: To be filled later in the process.
