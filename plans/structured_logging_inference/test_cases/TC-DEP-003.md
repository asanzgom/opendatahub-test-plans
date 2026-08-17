---
test_case_id: TC-DEP-003
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-DEP-003: Inference workloads continue when logging dependencies missing

**Objective**: Verify that inference workloads (LLMInferenceService
pods) continue to operate normally and logs remain accessible via
`kubectl logs` when Loki and/or CLO are not installed.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Loki Operator is NOT installed
- LLMInferenceService deployed with a vLLM runtime in
  `inference-ns-a`
- Logging enabled in Monitoring CR (so the controller attempts
  to create resources)

**Test Steps**:
1. Confirm inference pods are running:
   ```bash
   oc get pods -n inference-ns-a -l serving.kserve.io/inferenceservice
   ```
2. Send an inference request:
   ```bash
   curl -X POST "http://vllm-svc.inference-ns-a:8000/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model":"test-model","prompt":"Hello","max_tokens":5}'
   ```
3. Verify the request succeeds with HTTP 200.
4. Access logs via kubectl:
   ```bash
   oc logs -n inference-ns-a \
     $(oc get pod -n inference-ns-a \
       -l serving.kserve.io/inferenceservice -o name | head -1) \
     --tail=20
   ```
5. Verify structured JSON log entries appear in stdout.

**Expected Results**:
- Step 1: Inference pods are in Running state
- Step 3: Inference request returns HTTP 200 with a valid
  completion response
- Step 4: `oc logs` returns log output (logs are still emitted
  to stdout regardless of collection pipeline status)
- Step 5: Log entries are valid JSON containing `timestamp`,
  `severity`, and `body` fields

**Notes**: To be filled later in the process.
