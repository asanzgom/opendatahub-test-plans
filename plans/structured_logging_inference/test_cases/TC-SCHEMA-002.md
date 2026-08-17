---
test_case_id: TC-SCHEMA-002
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-002: llm-d EPP structured JSON log schema conformance

**Objective**: Verify that the llm-d EPP (Endpoint Picker) emits
structured JSON logs conforming to the OTel Logs Data Model with
all required fields.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Inference pool deployed with llm-d EPP in namespace
  `inference-test-ns`
- At least one vLLM engine pod registered with the inference pool

**Test Steps**:
1. Send an inference request routed through the EPP:
   ```bash
   curl -s -X POST \
     "https://inference-pool-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b-instruct", "prompt": "Hello", "max_tokens": 10}'
   ```
2. Capture the EPP container logs:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=epp \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20
   ```
3. Parse the most recent log entry as JSON and validate required
   fields:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=epp \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=5 | tail -1 | jq '{
       has_timestamp: (.timestamp != null),
       has_severity: (.severity != null),
       has_body: (.body // .msg != null),
       has_service_name: (.["service.name"] != null),
       has_pod_name: (.["k8s.pod.name"] != null),
       has_namespace: (.["k8s.namespace.name"] != null)
     }'
   ```
4. Validate the `timestamp` is ISO 8601 and `severity` is valid
   OTel text.

**Expected Results**:
- Every log line from the EPP container parses as valid JSON
- Each log entry contains `timestamp` in ISO 8601 format
- Each log entry contains `severity` using OTel severity text
- Each log entry contains a non-empty message body
- Each log entry contains `service.name` identifying the EPP
- Each log entry contains `k8s.pod.name` matching the actual pod
- Each log entry contains `k8s.namespace.name` matching
  `inference-test-ns`

**Notes**: To be filled later in the process.
