---
test_case_id: TC-SCHEMA-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-001: vLLM structured JSON log schema conformance

**Objective**: Verify that vLLM emits structured JSON logs to stdout
conforming to the OTel Logs Data Model with all required fields.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- LLMInferenceService deployed with a vLLM runtime in namespace
  `inference-test-ns` using model `granite-3b-instruct`
- vLLM configured with `LOG_FORMAT=json` (default for RHOAI)

**Test Steps**:
1. Send an inference request to the vLLM endpoint:
   ```bash
   curl -s -X POST \
     "https://granite-3b-instruct-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b-instruct", "prompt": "What is Kubernetes?", "max_tokens": 50}'
   ```
2. Capture the vLLM container logs:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20
   ```
3. Parse the most recent log entry as JSON and validate required
   fields:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=5 | tail -1 | jq '{
       has_timestamp: (.timestamp != null),
       has_severity: (.severity != null),
       has_body: (.body != null),
       has_service_name: (.resource.service_name // .["service.name"] != null),
       has_pod_name: (.resource.k8s_pod_name // .["k8s.pod.name"] != null),
       has_namespace: (.resource.k8s_namespace_name // .["k8s.namespace.name"] != null)
     }'
   ```
4. Validate the `timestamp` field is in ISO 8601 format:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=5 | tail -1 | jq -r '.timestamp' | \
     grep -E '^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}'
   ```
5. Validate the `severity` field uses OTel severity text values:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=5 | tail -1 | jq -r '.severity' | \
     grep -E '^(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)$'
   ```

**Expected Results**:
- Every log line from vLLM stdout parses as valid JSON (no parse
  errors from `jq`)
- Each log entry contains `timestamp` in ISO 8601 format
- Each log entry contains `severity` using OTel severity text
  (TRACE, DEBUG, INFO, WARN, ERROR, or FATAL)
- Each log entry contains a non-empty `body` field with the log
  message
- Each log entry contains `service.name` identifying the vLLM
  component
- Each log entry contains `k8s.pod.name` matching the actual pod
  name
- Each log entry contains `k8s.namespace.name` matching
  `inference-test-ns`

**Notes**: To be filled later in the process.
