---
test_case_id: TC-TRACE-002
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TRACE-002: trace_id and span_id omitted when no traceparent
header is present

**Objective**: Verify that when an inference request is sent
without a W3C traceparent header, the `trace_id` and `span_id`
fields are omitted entirely from the structured log entry -- not
present as null or empty string (AC 12).

**Preconditions**:
- A deployed LLMInferenceService with vLLM runtime
- Structured JSON logging is enabled

**Test Steps**:
1. Send an inference request without a traceparent header:
   ```bash
   curl -H "Content-Type: application/json" \
     -d '{"prompt": "Hello", "max_tokens": 10}' \
     https://<inference-endpoint>/v1/completions
   ```
2. Retrieve structured log entries from the vLLM engine pod:
   ```bash
   oc logs <vllm-pod> -n <inference-ns> | \
     jq -c '. | keys'
   ```
3. Parse each log entry as JSON
4. Verify that the key `trace_id` does not exist in the JSON
   object
5. Verify that the key `span_id` does not exist in the JSON
   object

**Expected Results**:
- The JSON log entry does not contain a `trace_id` key
- The JSON log entry does not contain a `span_id` key
- Neither field is present as `null`, `""`, or `"0000..."`

**Notes**: To be filled later in the process.
