---
test_case_id: TC-ROUTE-004
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-ROUTE-004: MaaS routing rejection log on model unavailable

**Objective**: Verify that MaaS emits a structured JSON rejection log
entry when a request is rejected because the requested model is not
available, including tenant identity, model name, and rejection reason.

**Preconditions**:
- MaaS deployment with tenant subscriptions configured
- The requested model is NOT deployed or is in a failed state

**Test Steps**:
1. Identify the MaaS API server pod:
   ```bash
   MAAS_POD=$(oc get pods -n $MAAS_NS \
     -l app=models-as-a-service -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send a request for a model that does not exist:
   ```bash
   curl -v -H "Authorization: Bearer $TENANT_A_TOKEN" \
        -H "traceparent: 00-dd0011223344556677889900aabbccdd-aabbccdd11223344-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "nonexistent-model-7b", "prompt": "Hello:", "max_tokens": 10}' \
        https://$MAAS_ENDPOINT/v1/completions
   ```
3. Retrieve MaaS pod logs and filter for rejection entries:
   ```bash
   oc logs $MAAS_POD -n $MAAS_NS | \
     jq -c 'select(.body | test("reject|unavailable|not found"; "i"))'
   ```

**Expected Results**:
- MaaS pod logs contain a structured JSON rejection log entry
- The rejection log includes tenant identity matching the test tenant
- The rejection log includes model name `nonexistent-model-7b`
- The rejection log includes a rejection reason indicating model
  unavailable (e.g., "model not available", "model unavailable")
- The rejection log includes `trace_id` matching
  `dd0011223344556677889900aabbccdd`
- The HTTP response is a 404 or 503 status

**Notes**: To be filled later in the process.
