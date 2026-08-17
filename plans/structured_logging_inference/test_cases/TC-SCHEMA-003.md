---
test_case_id: TC-SCHEMA-003
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-003: MaaS structured JSON log schema conformance

**Objective**: Verify that the MaaS (models-as-a-service) API server
and controller emit structured JSON logs conforming to the OTel Logs
Data Model.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- MaaS deployment configured with at least one model subscription
  in namespace `maas-platform-ns`
- A tenant namespace `tenant-a-ns` with a valid subscription

**Test Steps**:
1. Trigger a MaaS API request as an identified tenant:
   ```bash
   curl -s -X POST \
     "https://maas-api-maas-platform-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <tenant-a-token>" \
     -d '{"model": "granite-3b-instruct", "prompt": "Summarize", "max_tokens": 20}'
   ```
2. Capture the MaaS API server logs:
   ```bash
   kubectl logs -n maas-platform-ns \
     $(kubectl get pods -n maas-platform-ns -l app=maas-api \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20
   ```
3. Parse a recent log entry and validate OTel schema fields:
   ```bash
   kubectl logs -n maas-platform-ns \
     $(kubectl get pods -n maas-platform-ns -l app=maas-api \
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

**Expected Results**:
- Every log line from MaaS containers parses as valid JSON
- Each log entry contains `timestamp`, `severity`, `body`,
  `service.name`, `k8s.pod.name`, and `k8s.namespace.name`
- Severity values use OTel severity text
- `service.name` identifies the MaaS component (API server or
  controller)

**Notes**: To be filled later in the process.
