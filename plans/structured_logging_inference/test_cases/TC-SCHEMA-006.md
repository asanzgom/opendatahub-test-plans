---
test_case_id: TC-SCHEMA-006
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-006: GenAI semconv fields absent on failed requests

**Objective**: Verify that GenAI semantic convention usage fields
(`gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`) are
omitted from vLLM log entries when a request fails before generating
token counts -- not present as zero or null.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- LLMInferenceService deployed with vLLM runtime in namespace
  `inference-test-ns`
- vLLM configured with `LOG_FORMAT=json`

**Test Steps**:
1. Send a request designed to fail before token generation (e.g.,
   reference a non-existent model or trigger a runtime error):
   ```bash
   curl -s -X POST \
     "https://granite-3b-instruct-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "nonexistent-model", "prompt": "test", "max_tokens": 10}'
   ```
2. Capture vLLM logs and filter for error-level entries:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20 | jq 'select(.severity == "ERROR")' 2>/dev/null
   ```
3. Check that GenAI usage fields are absent (not zero, not null):
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20 | jq 'select(.severity == "ERROR") | {
       has_input_tokens: (has("gen_ai.usage.input_tokens")),
       has_output_tokens: (has("gen_ai.usage.output_tokens"))
     }' 2>/dev/null
   ```

**Expected Results**:
- The error log entry does NOT contain the key
  `gen_ai.usage.input_tokens` (absent from JSON, not present as
  `0` or `null`)
- The error log entry does NOT contain the key
  `gen_ai.usage.output_tokens` (absent from JSON, not present as
  `0` or `null`)
- The `gen_ai.request.model` field may still be present if the
  model name was resolved before the failure

**Notes**: To be filled later in the process.
