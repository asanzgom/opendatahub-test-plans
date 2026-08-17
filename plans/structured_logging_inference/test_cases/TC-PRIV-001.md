---
test_case_id: TC-PRIV-001
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PRIV-001: No prompt or completion content in logs by default

**Objective**: Verify that no inference request/response content
appears in any log entry when opt-in content logging is not enabled
(AC 7).

**Preconditions**:
- vLLM runtime deployed via LLMInferenceService with default
  configuration (`VLLM_LOG_REQUESTS_CONTENT` not set or set to
  `false`)
- Logging collection pipeline active and routing logs to Loki

**Test Steps**:
1. Send an inference request with a distinctive, searchable prompt:
   ```bash
   curl -X POST https://<inference-endpoint>/v1/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "<model-name>",
       "prompt": "PRIVACY_CANARY_a1b2c3: Explain the Doppler effect",
       "max_tokens": 50
     }'
   ```
2. Wait for the response to complete and logs to propagate to Loki
   (allow 30-60 seconds for collection pipeline delivery).
3. Search all inference component logs for the canary string:
   ```bash
   # Via kubectl logs (direct stdout)
   for pod in $(oc get pods -l app=vllm -o name); do
     oc logs "$pod" | grep -c "PRIVACY_CANARY_a1b2c3"
   done

   # Via Loki query (collected logs)
   # In OpenShift Console > Observe > Logs, search for:
   # {namespace="<inference-ns>"} |= "PRIVACY_CANARY_a1b2c3"
   ```
4. Repeat the search across EPP, MaaS, and KServe controller logs
   in the same namespace.

**Expected Results**:
- Zero matches for the canary string across all inference component
  logs (vLLM, EPP, MaaS, KServe controllers)
- Log entries for the request exist (with `trace_id`, `severity`,
  `service.name`) but contain no prompt or completion text
- No partial content leakage (e.g., truncated prompt fragments)

**Notes**: To be filled later in the process.
