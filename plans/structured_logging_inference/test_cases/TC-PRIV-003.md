---
test_case_id: TC-PRIV-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PRIV-003: Content logging flag surfaceable via LLMInferenceServiceConfig

**Objective**: Verify that the `VLLM_LOG_REQUESTS_CONTENT` flag can
be set per-model or fleet-wide through the LLMInferenceServiceConfig
template using the versioned base config mechanism (overlay 0022).

**Preconditions**:
- LLMInferenceServiceConfig resource available in the cluster
- vLLM runtime deployed via LLMInferenceService referencing a
  base config

**Test Steps**:
1. Create or update an LLMInferenceServiceConfig that includes the
   content logging flag:
   ```yaml
   apiVersion: serving.kserve.io/v1alpha1
   kind: LLMInferenceServiceConfig
   metadata:
     name: content-logging-enabled
     namespace: <inference-ns>
   spec:
     vllm:
       env:
         - name: VLLM_LOG_REQUESTS_CONTENT
           value: "true"
   ```
2. Deploy an LLMInferenceService referencing this config:
   ```yaml
   apiVersion: serving.kserve.io/v1alpha1
   kind: LLMInferenceService
   metadata:
     name: test-model-content-log
     namespace: <inference-ns>
   spec:
     configRef:
       name: content-logging-enabled
   ```
3. Wait for the model pods to become ready.
4. Verify the environment variable is present on the vLLM container:
   ```bash
   oc get pod -l app=test-model-content-log -o json | \
     jq '.items[0].spec.containers[0].env[] |
         select(.name=="VLLM_LOG_REQUESTS_CONTENT")'
   ```
5. Send an inference request and verify content appears in logs
   (same verification as TC-PRIV-002).
6. Update the config to disable content logging
   (`VLLM_LOG_REQUESTS_CONTENT=false`) and verify the change
   propagates after pod restart.

**Expected Results**:
- The vLLM container has `VLLM_LOG_REQUESTS_CONTENT=true` in its
  environment when the config is applied
- Content appears in logs when the flag is enabled via the config
- After changing the flag to `false` and restarting, content no
  longer appears in logs
- The flag is set per-model (scoped to the LLMInferenceService
  that references the config), not globally

**Notes**: To be filled later in the process.
