---
test_case_id: TC-COMPAT-002
source_key: RHAISTRAT-2413
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-COMPAT-002: LOG_FORMAT=json (default) produces structured output

**Objective**: Verify that the default vLLM configuration (no
`LOG_FORMAT` set or `LOG_FORMAT=json`) produces valid structured
JSON log output conforming to the OTel Logs Data Model.

**Preconditions**:
- vLLM runtime deployed via LLMInferenceService with default
  configuration (no `LOG_FORMAT` environment variable set)

**Test Steps**:
1. Deploy vLLM with default log configuration (no `LOG_FORMAT`
   override).
2. Send an inference request:
   ```bash
   curl -X POST https://<inference-endpoint>/v1/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "<model-name>",
       "prompt": "What is 2+2?",
       "max_tokens": 10
     }'
   ```
3. Capture and validate log output:
   ```bash
   oc logs $(oc get pods -l app=vllm -o name | head -1) \
     --tail=20 | while IFS= read -r line; do
     if echo "$line" | jq . >/dev/null 2>&1; then
       echo "PASS: Valid JSON"
       echo "$line" | jq '{timestamp, severity, body,
         service_name: .["service.name"],
         pod: .["k8s.pod.name"]}'
     else
       echo "FAIL: Not valid JSON: $line"
     fi
   done
   ```

**Expected Results**:
- vLLM log lines parse as valid JSON
- Each JSON log entry contains the required OTel fields:
  `timestamp`, `severity`, `body`, `service.name`,
  `k8s.pod.name`, `k8s.namespace.name`
- JSON is the default output format when `LOG_FORMAT` is not
  explicitly set

**Notes**: To be filled later in the process.
