---
test_case_id: TC-QUERY-002
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUERY-002: Query logs by k8s.pod.name to isolate a
specific engine

**Objective**: Verify that filtering by `k8s.pod.name` returns
only log entries from that specific engine pod and no entries from
other pods in the inference pool (AC 10).

**Preconditions**:
- A deployed inference pool with at least two vLLM engine pods
  (e.g., `vllm-pool-engine-0` and `vllm-pool-engine-1`)
- Both pods are processing inference requests and emitting
  structured JSON logs to Loki

**Test Steps**:
1. Identify two vLLM engine pod names in the inference pool:
   ```bash
   oc get pods -n inference-ns -l app=vllm-runtime \
     -o jsonpath='{.items[*].metadata.name}'
   ```
2. Open OpenShift Console Observe > Logs
3. Filter by `k8s.pod.name` matching the first pod name
   (e.g., `{k8s_pod_name="vllm-pool-engine-0"}`)
4. Review the returned log entries
5. Verify that every entry has `k8s.pod.name` matching the
   filtered pod
6. Verify that zero entries from the second pod appear

**Expected Results**:
- All returned log entries contain `k8s.pod.name` equal to the
  filtered pod name
- Zero log entries from other pods in the inference pool appear
  in the result set

**Notes**: To be filled later in the process.
