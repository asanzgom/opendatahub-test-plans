---
test_case_id: TC-ROUTE-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-ROUTE-003: MaaS routing rejection log on quota exceeded

**Objective**: Verify that MaaS emits a structured JSON rejection log
entry with tenant identity, model name, rejection reason, and
`trace_id` when a request is rejected due to tenant quota being
exceeded.

**Preconditions**:
- MaaS deployment configured with tenant subscriptions and quota
  limits
- A test tenant with a low quota configured (e.g., 10 requests/min)
- W3C traceparent header propagation is active

**Test Steps**:
1. Identify the MaaS API server pod:
   ```bash
   MAAS_POD=$(oc get pods -n $MAAS_NS \
     -l app=models-as-a-service -o jsonpath='{.items[0].metadata.name}')
   ```
2. Exhaust the test tenant's quota by sending requests up to the
   limit:
   ```bash
   for i in $(seq 1 10); do
     curl -s -H "Authorization: Bearer $TENANT_A_TOKEN" \
          -H "Content-Type: application/json" \
          -d '{"model": "granite-3b", "prompt": "Request '$i':", "max_tokens": 10}' \
          https://$MAAS_ENDPOINT/v1/completions
   done
   ```
3. Send one more request that exceeds the quota:
   ```bash
   curl -v -H "Authorization: Bearer $TENANT_A_TOKEN" \
        -H "traceparent: 00-ee0011223344556677889900aabbccdd-1122334455667788-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "granite-3b", "prompt": "Over quota request:", "max_tokens": 10}' \
        https://$MAAS_ENDPOINT/v1/completions
   ```
4. Retrieve MaaS pod logs and filter for rejection entries:
   ```bash
   oc logs $MAAS_POD -n $MAAS_NS | \
     jq -c 'select(.body | test("reject|denied|quota"; "i"))'
   ```

**Expected Results**:
- MaaS pod logs contain a structured JSON rejection log entry
- The rejection log includes tenant identity matching the test tenant
- The rejection log includes model name `granite-3b`
- The rejection log includes a rejection reason indicating quota
  exceeded (e.g., "tenant quota exceeded")
- The rejection log includes `trace_id` matching
  `ee0011223344556677889900aabbccdd`
- The HTTP response to the over-quota request is a 429 or 503 status

**Notes**: To be filled later in the process.
