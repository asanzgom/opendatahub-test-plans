---
test_case_id: TC-EMIT-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-EMIT-003: MaaS emits structured log on successful tenant request

**Objective**: Verify that MaaS emits a structured log entry for a
successful tenant inference request containing tenant identity,
model name, token counts, and `trace_id`.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- MaaS deployment in namespace `maas-platform-ns`
- Tenant A configured with a valid subscription and namespace
  `tenant-a-ns`
- W3C traceparent propagation enabled in the request path

**Test Steps**:
1. Generate a traceparent header for correlation:
   ```bash
   TRACE_ID=$(python3 -c "import secrets; print(secrets.token_hex(16))")
   SPAN_ID=$(python3 -c "import secrets; print(secrets.token_hex(8))")
   TRACEPARENT="00-${TRACE_ID}-${SPAN_ID}-01"
   ```
2. Send a request as Tenant A with traceparent:
   ```bash
   curl -s -X POST \
     "https://maas-api-maas-platform-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <tenant-a-token>" \
     -H "traceparent: ${TRACEPARENT}" \
     -d '{"model": "granite-3b-instruct", "prompt": "Summarize AI.", "max_tokens": 20}'
   ```
3. Capture MaaS logs and filter for the request by trace_id:
   ```bash
   MAAS_POD=$(kubectl get pods -n maas-platform-ns \
     -l app=maas-api -o jsonpath='{.items[0].metadata.name}')
   kubectl logs -n maas-platform-ns "$MAAS_POD" --tail=30 | \
     jq "select(.trace_id == \"${TRACE_ID}\")" 2>/dev/null
   ```
4. Validate tenant and token fields:
   ```bash
   kubectl logs -n maas-platform-ns "$MAAS_POD" --tail=30 | \
     jq "select(.trace_id == \"${TRACE_ID}\") | {
       tenant: (.tenant_identity // .tenant),
       model: (.model // .model_name),
       input_tokens: .[\"gen_ai.usage.input_tokens\"],
       output_tokens: .[\"gen_ai.usage.output_tokens\"],
       trace_id: .trace_id
     }" 2>/dev/null
   ```

**Expected Results**:
- A structured log entry appears with `trace_id` matching the
  injected value
- The entry contains tenant identity identifying Tenant A
- The entry contains the model name `granite-3b-instruct`
- The entry contains `gen_ai.usage.input_tokens` as a positive
  integer
- The entry contains `gen_ai.usage.output_tokens` as a positive
  integer
- All standard OTel fields are present

**Notes**: To be filled later in the process.
