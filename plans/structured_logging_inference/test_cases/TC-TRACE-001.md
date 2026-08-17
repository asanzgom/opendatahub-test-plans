---
test_case_id: TC-TRACE-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TRACE-001: trace_id present in log entries when traceparent
header is available

**Objective**: Verify that when a W3C traceparent header is
present on an inference request, all inference component log
entries include the matching `trace_id` and `span_id` fields.

**Preconditions**:
- A deployed LLMInferenceService with vLLM runtime and EPP
- W3C traceparent header propagation is configured across the
  inference request path
- Structured JSON logging is enabled on all components

**Test Steps**:
1. Generate a W3C traceparent header with a known trace ID:
   ```bash
   TRACE_ID="0af7651916cd43dd8448eb211c80319c"
   SPAN_ID="b7ad6b7169203331"
   TRACEPARENT="00-${TRACE_ID}-${SPAN_ID}-01"
   ```
2. Send an inference request with the traceparent header:
   ```bash
   curl -H "traceparent: ${TRACEPARENT}" \
     -H "Content-Type: application/json" \
     -d '{"prompt": "Hello", "max_tokens": 10}' \
     https://<inference-endpoint>/v1/completions
   ```
3. Retrieve structured log entries from the vLLM engine pod:
   ```bash
   oc logs <vllm-pod> -n <inference-ns> | \
     jq -r 'select(.trace_id != null)'
   ```
4. Parse each log entry as JSON
5. Verify that `trace_id` equals the value from the traceparent
   header (`0af7651916cd43dd8448eb211c80319c`)
6. Verify that `span_id` is present and non-empty

**Expected Results**:
- Log entries from vLLM contain `trace_id` matching the
  traceparent header value
- Log entries contain a non-empty `span_id` field
- The `trace_id` and `span_id` fields are string values, not
  null or empty

**Notes**: To be filled later in the process.
