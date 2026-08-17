---
test_case_id: TC-PRIV-002
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PRIV-002: Content logging enabled includes prompt and completion

**Objective**: Verify that when `VLLM_LOG_REQUESTS_CONTENT=true` is
set, vLLM log entries include prompt and completion content with
`trace_id` correlation (AC 14).

**Preconditions**:
- vLLM runtime deployed via LLMInferenceService with
  `VLLM_LOG_REQUESTS_CONTENT=true` set as an environment variable
- Logging collection pipeline active and routing logs to Loki

**Test Steps**:
1. Enable content logging on the vLLM deployment:
   ```yaml
   # In LLMInferenceService or pod spec
   env:
     - name: VLLM_LOG_REQUESTS_CONTENT
       value: "true"
     - name: LOG_FORMAT
       value: "json"
   ```
2. Send an inference request with a known prompt:
   ```bash
   curl -X POST https://<inference-endpoint>/v1/completions \
     -H "Content-Type: application/json" \
     -H "traceparent: 00-abcdef1234567890abcdef1234567890-fedcba0987654321-01" \
     -d '{
       "model": "<model-name>",
       "prompt": "Explain quantum computing in one sentence",
       "max_tokens": 50
     }'
   ```
3. Capture the vLLM pod logs:
   ```bash
   oc logs $(oc get pods -l app=vllm -o name | head -1) | \
     jq 'select(.body | contains("quantum computing"))'
   ```
4. Query Loki for the same content using the injected trace_id:
   ```bash
   # OpenShift Console > Observe > Logs:
   # {namespace="<ns>"} | json | trace_id="abcdef1234567890abcdef1234567890"
   ```

**Expected Results**:
- vLLM log entry contains the prompt text "Explain quantum
  computing in one sentence" in the log body or a dedicated
  content field
- The same log entry contains a non-empty completion text
- The log entry includes
  `trace_id: "abcdef1234567890abcdef1234567890"` matching the
  injected traceparent
- The log entry parses as valid JSON with all required OTel fields
  (`timestamp`, `severity`, `service.name`, `k8s.pod.name`)

**Notes**: To be filled later in the process.
