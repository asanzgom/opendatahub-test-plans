---
test_case_id: TC-COMPAT-001
source_key: RHAISTRAT-2413
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-COMPAT-001: LOG_FORMAT=text produces unstructured output

**Objective**: Verify that configuring vLLM with `LOG_FORMAT=text`
produces unstructured text log output that does not parse as valid
JSON (AC 15).

**Preconditions**:
- vLLM runtime deployed via LLMInferenceService with
  `LOG_FORMAT=text` set as an environment variable

**Test Steps**:
1. Deploy vLLM with text format logging:
   ```yaml
   env:
     - name: LOG_FORMAT
       value: "text"
   ```
2. Send an inference request:
   ```bash
   curl -X POST https://<inference-endpoint>/v1/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "<model-name>",
       "prompt": "Hello world",
       "max_tokens": 10
     }'
   ```
3. Capture log output and attempt JSON parsing:
   ```bash
   oc logs $(oc get pods -l app=vllm -o name | head -1) \
     --tail=20 | while IFS= read -r line; do
     if echo "$line" | jq . >/dev/null 2>&1; then
       echo "FAIL: Line parses as JSON: $line"
     else
       echo "PASS: Unstructured text: $line"
     fi
   done
   ```

**Expected Results**:
- vLLM log lines do NOT parse as valid JSON
- Log output is human-readable unstructured text (e.g., standard
  Python logging format with timestamp, level, module, message)
- The vLLM runtime operates normally with `LOG_FORMAT=text` — no
  startup errors or degraded functionality

**Notes**: To be filled later in the process.
