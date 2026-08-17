---
test_case_id: TC-TRACE-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TRACE-003: Cross-component trace correlation via trace_id

**Objective**: Verify that a single traced inference request
produces log entries from all components in the request path
(vLLM, EPP, MaaS) and that every entry carries the identical
`trace_id` value (AC 8).

**Preconditions**:
- Full inference stack deployed: MaaS API, EPP, vLLM engine pool
- W3C traceparent propagation is configured across all components
- Structured JSON logging enabled on all components
- Logs from all components are collected and queryable in Loki

**Test Steps**:
1. Generate a unique trace ID and traceparent header:
   ```bash
   TRACE_ID="d4cda95b652f4a1592b449d5929fda1b"
   TRACEPARENT="00-${TRACE_ID}-b7ad6b7169203331-01"
   ```
2. Send a single inference request through the MaaS API with the
   traceparent header:
   ```bash
   curl -H "traceparent: ${TRACEPARENT}" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b", "prompt": "Test", "max_tokens": 5}' \
     https://<maas-endpoint>/v1/completions
   ```
3. Wait for the request to complete
4. Query Loki for all log entries with the trace ID:
   ```
   {trace_id="d4cda95b652f4a1592b449d5929fda1b"}
   ```
5. Group the returned entries by `service.name`
6. Verify that entries appear from at least vLLM, EPP, and MaaS
7. Verify that every returned entry carries the identical
   `trace_id` value

**Expected Results**:
- Log entries are returned from at least three components: vLLM,
  EPP, and MaaS
- Every returned log entry has `trace_id` equal to
  `d4cda95b652f4a1592b449d5929fda1b`
- No entries with a different `trace_id` appear in the result set

**Notes**: To be filled later in the process.
