---
test_case_id: TC-QUEUE-003
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUEUE-003: vLLM emits discard log with KV cache pressure reason

**Objective**: Verify that vLLM emits a structured JSON log entry when
a request is discarded due to KV cache pressure, including the discard
reason, engine identity, and `trace_id`.

**Preconditions**:
- GPU-equipped cluster provisioned via Hive (dedicated QE cluster)
- LLMInferenceService deployed with a vLLM runtime and
  `LOG_FORMAT=json`
- The vLLM model is loaded with constrained KV cache capacity to
  trigger pressure conditions
- W3C traceparent header propagation is active
- This test runs on weekly cadence, not per-PR CI

**Test Steps**:
1. Identify a vLLM engine pod:
   ```bash
   VLLM_POD=$(oc get pods -n $INFERENCE_NS \
     -l app=vllm-engine -o jsonpath='{.items[0].metadata.name}')
   ```
2. Generate concurrent inference requests with large prompt contexts
   to exhaust KV cache capacity:
   ```bash
   for i in $(seq 1 50); do
     curl -s -H "traceparent: 00-aabb${i}00000000000000000000000000-0000000000000001-01" \
          -H "Content-Type: application/json" \
          -d '{"model": "granite-3b", "prompt": "'$(python3 -c "print('token ' * 2000)")'", "max_tokens": 500}' \
          https://$INFERENCE_ENDPOINT/v1/completions &
   done
   wait
   ```
3. Retrieve the vLLM engine pod logs and filter for discard/timeout
   events:
   ```bash
   oc logs $VLLM_POD -n $INFERENCE_NS | \
     jq -c 'select(.body | test("discard|timeout"; "i"))'
   ```
4. Validate that at least one discard log entry exists with the
   expected fields.

**Expected Results**:
- At least one JSON log entry references a discard or timeout event
- The log entry includes a discard reason field indicating KV cache
  pressure (e.g., "KV cache full")
- The log entry includes `k8s.pod.name` matching `$VLLM_POD`
- The log entry includes `trace_id` from the traceparent header of
  the discarded request
- The log entry includes `timestamp` and `severity`

**Notes**: To be filled later in the process.
