---
test_case_id: TC-ROUTE-002
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-ROUTE-002: EPP routing decision log includes queue depth and KV-cache utilization

**Objective**: Verify that EPP routing decision log entries include
queue depth and KV-cache utilization metrics at the time of the routing
decision, enabling operators to understand scheduling behavior.

**Preconditions**:
- Inference pool deployed with llm-d EPP and at least 2 vLLM engine
  pods under varying load
- EPP configured with OTel SDK integration (>= 1.39.1)

**Test Steps**:
1. Identify the EPP pod:
   ```bash
   EPP_POD=$(oc get pods -n $INFERENCE_NS \
     -l app=llm-d-epp -o jsonpath='{.items[0].metadata.name}')
   ```
2. Generate several concurrent inference requests to create
   measurable queue depth variation:
   ```bash
   for i in $(seq 1 20); do
     curl -s -H "Content-Type: application/json" \
          -d '{"model": "granite-3b", "prompt": "Request '$i':", "max_tokens": 100}' \
          https://$INFERENCE_ENDPOINT/v1/completions &
   done
   wait
   ```
3. Retrieve EPP routing decision logs:
   ```bash
   oc logs $EPP_POD -n $INFERENCE_NS | \
     jq -c 'select(.body | test("routing|selected"; "i")) |
       {queue_depth, kv_cache_utilization, body}'
   ```
4. Validate that routing decision entries include scheduling metrics.

**Expected Results**:
- Routing decision log entries contain a queue depth value (numeric
  field indicating requests waiting per engine)
- Routing decision log entries contain a KV-cache utilization value
  (numeric field indicating cache usage per engine at decision time)
- Both metrics are present in the same log entry alongside the
  selected engine and selection reason
- Metrics reflect the actual state at decision time (non-zero queue
  depth when load is applied)

**Notes**: To be filled later in the process.
