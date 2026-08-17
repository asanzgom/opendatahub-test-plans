---
test_case_id: TC-PIPE-003
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PIPE-003: Collection resources apply JSON parsing to container stdout

**Objective**: Verify that the logging pipeline parses structured
JSON from container stdout and extracts individual fields into Loki
labels or structured fields for querying.

**Preconditions**:
- Logging enabled and collection resources created (TC-PIPE-001
  passed)
- A vLLM runtime pod running in `inference-ns-a` processing
  inference requests and emitting structured JSON logs

**Test Steps**:
1. Send an inference request to the vLLM endpoint to generate a
   structured log entry:
   ```bash
   curl -X POST "http://vllm-svc.inference-ns-a:8000/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model":"test-model","prompt":"Hello","max_tokens":5}'
   ```
2. Wait 60 seconds for log collection and delivery.
3. Query Loki for logs from the vLLM pod, filtering by
   `service.name`:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/logging-query -- \
     logcli query '{service_name="vllm"}'
   ```
4. Verify that individual JSON fields are queryable as labels
   or structured fields:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/logging-query -- \
     logcli query '{service_name="vllm"} | json | severity="INFO"'
   ```

**Expected Results**:
- Step 3: Log entries appear with `service_name` as a queryable
  label or field (not just embedded in the raw log line)
- Step 4: The `severity` field is extractable via JSON parsing in
  LogQL, confirming the pipeline preserves JSON structure rather
  than treating logs as plain text

**Notes**: To be filled later in the process.
