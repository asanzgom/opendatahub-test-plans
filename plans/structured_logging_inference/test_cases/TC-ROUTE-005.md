---
test_case_id: TC-ROUTE-005
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-ROUTE-005: MaaS tenant/token log for chargeback

**Objective**: Verify that MaaS emits a structured JSON log entry
containing tenant identity, model name, per-request token counts
(`gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`), and
`trace_id` for a successful inference request, enabling
chargeback/showback (Use Case 4).

**Preconditions**:
- MaaS deployment with a configured tenant subscription
- A model deployed and serving inference requests
- W3C traceparent header propagation is active

**Test Steps**:
1. Identify the MaaS API server pod:
   ```bash
   MAAS_POD=$(oc get pods -n $MAAS_NS \
     -l app=models-as-a-service -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send a successful inference request as an identified tenant:
   ```bash
   curl -H "Authorization: Bearer $TENANT_A_TOKEN" \
        -H "traceparent: 00-cc0011223344556677889900aabbccdd-5566778899001122-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "granite-3b", "prompt": "What is Kubernetes?", "max_tokens": 100}' \
        https://$MAAS_ENDPOINT/v1/completions
   ```
3. Verify the request completes successfully (HTTP 200).
4. Retrieve MaaS pod logs and filter for tenant/token log entries:
   ```bash
   oc logs $MAAS_POD -n $MAAS_NS | \
     jq -c 'select(.trace_id == "cc0011223344556677889900aabbccdd")'
   ```
5. Validate the chargeback-relevant fields in the log entry.

**Expected Results**:
- MaaS pod logs contain a structured JSON log entry for the completed
  request
- The log entry includes tenant identity matching the test tenant
- The log entry includes model name `granite-3b`
- The log entry includes `gen_ai.usage.input_tokens` with a positive
  integer value
- The log entry includes `gen_ai.usage.output_tokens` with a positive
  integer value
- The log entry includes `trace_id` matching
  `cc0011223344556677889900aabbccdd`
- Token counts in the log entry are consistent with the response's
  `usage` field

**Notes**: To be filled later in the process.
