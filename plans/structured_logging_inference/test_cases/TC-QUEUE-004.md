---
test_case_id: TC-QUEUE-004
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUEUE-004: Queue lifecycle logs include engine identity for
per-engine debugging

**Objective**: Verify that queue lifecycle log entries include
`k8s.pod.name` to enable per-engine debugging in a multi-engine
inference pool (Use Case 2: one engine held a request for 15 min,
another processed it in 5s).

**Preconditions**:
- LLMInferenceService deployed with an inference pool of at least 2
  vLLM engine pods
- `LOG_FORMAT=json` on all engine pods
- W3C traceparent header propagation is active

**Test Steps**:
1. List all vLLM engine pods in the inference pool:
   ```bash
   VLLM_PODS=$(oc get pods -n $INFERENCE_NS \
     -l app=vllm-engine -o jsonpath='{.items[*].metadata.name}')
   ```
2. Send multiple inference requests to distribute across engine pods:
   ```bash
   for i in $(seq 1 10); do
     curl -s -H "traceparent: 00-ccdd${i}00000000000000000000000000-0000000000000001-01" \
          -H "Content-Type: application/json" \
          -d '{"model": "granite-3b", "prompt": "Request '$i':", "max_tokens": 20}' \
          https://$INFERENCE_ENDPOINT/v1/completions &
   done
   wait
   ```
3. Collect queue lifecycle logs from each engine pod:
   ```bash
   for pod in $VLLM_PODS; do
     echo "=== $pod ==="
     oc logs $pod -n $INFERENCE_NS | \
       jq -c 'select(.body | test("enqueue|dequeue"; "i")) | {pod: .["k8s.pod.name"], trace_id, body}'
   done
   ```
4. Verify that `k8s.pod.name` in each log entry matches the pod it
   was collected from.
5. Verify that logs from different pods can be distinguished by
   filtering on `k8s.pod.name`.

**Expected Results**:
- Queue lifecycle log entries from each pod include `k8s.pod.name`
  matching that pod's actual name
- Logs from pod A do not contain pod B's `k8s.pod.name` and vice
  versa
- Filtering logs by `k8s.pod.name` isolates a single engine's queue
  activity, enabling per-engine debugging

**Notes**: To be filled later in the process.
