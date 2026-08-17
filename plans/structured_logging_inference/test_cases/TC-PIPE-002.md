---
test_case_id: TC-PIPE-002
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PIPE-002: Collection resources select logs from inference component namespaces

**Objective**: Verify that the logging pipeline input selectors
include namespaces where LLMInferenceService and InferenceService
pods run, plus the platform application namespace for MaaS
components.

**Preconditions**:
- Logging enabled in Monitoring CR and collection resources created
  (TC-PIPE-001 passed)
- LLMInferenceService deployed in user namespace `inference-ns-a`
- MaaS controller running in platform namespace
  `redhat-ods-applications`
- A non-inference namespace `unrelated-ns` with unrelated pods

**Test Steps**:
1. Inspect the collection resource input selectors:
   ```bash
   oc get <collection-resource> -n redhat-ods-monitoring -o yaml \
     | grep -A 20 'selector\|namespaceSelector\|include'
   ```
2. Deploy a test pod in `inference-ns-a` that emits a structured
   JSON log line with a unique marker string:
   ```bash
   oc run log-test -n inference-ns-a --image=busybox \
     --restart=Never -- sh -c \
     'echo "{\"severity\":\"INFO\",\"body\":\"TC-PIPE-002-inference-marker\",\"service.name\":\"test\"}" && sleep 30'
   ```
3. Deploy a test pod in `unrelated-ns` that emits a structured
   JSON log line with a different marker:
   ```bash
   oc run log-test -n unrelated-ns --image=busybox \
     --restart=Never -- sh -c \
     'echo "{\"severity\":\"INFO\",\"body\":\"TC-PIPE-002-unrelated-marker\",\"service.name\":\"test\"}" && sleep 30'
   ```
4. Wait 60 seconds for log collection and delivery.
5. Query Loki for the inference marker:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/logging-query -- \
     logcli query '{namespace="inference-ns-a"}' \
     --grep="TC-PIPE-002-inference-marker"
   ```
6. Query Loki for the unrelated marker:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/logging-query -- \
     logcli query '{namespace="unrelated-ns"}' \
     --grep="TC-PIPE-002-unrelated-marker"
   ```

**Expected Results**:
- Step 1: Input selectors reference inference component namespaces
  (user namespaces with LLMInferenceService pods) and the platform
  application namespace
- Step 5: The inference marker log line appears in Loki query
  results
- Step 6: The unrelated namespace marker does NOT appear in Loki
  (collection pipeline only selects inference namespaces)

**Notes**: To be filled later in the process.
