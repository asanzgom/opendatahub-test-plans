---
test_case_id: TC-SCHEMA-004
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-004: KServe controller lifecycle log schema conformance

**Objective**: Verify that KServe controllers and odh-model-controller
emit structured JSON logs conforming to the OTel Logs Data Model for
reconciliation and lifecycle events.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- KServe and odh-model-controller running in the RHOAI application
  namespace (e.g., `redhat-ods-applications`)
- An LLMInferenceService resource deployed in `inference-test-ns`

**Test Steps**:
1. Trigger a reconciliation by modifying the LLMInferenceService:
   ```bash
   oc annotate llminferenceservice granite-3b-instruct \
     -n inference-test-ns \
     test-trigger="$(date +%s)" --overwrite
   ```
2. Capture odh-model-controller logs:
   ```bash
   kubectl logs -n redhat-ods-applications \
     $(kubectl get pods -n redhat-ods-applications \
       -l app=odh-model-controller \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20
   ```
3. Parse a reconciliation log entry and validate schema:
   ```bash
   kubectl logs -n redhat-ods-applications \
     $(kubectl get pods -n redhat-ods-applications \
       -l app=odh-model-controller \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=10 | grep -i "reconcil" | tail -1 | jq '{
       has_timestamp: (.timestamp // .ts != null),
       has_severity: (.severity // .level != null),
       has_body: (.body // .msg != null),
       has_service_name: (.["service.name"] != null),
       has_pod_name: (.["k8s.pod.name"] != null),
       has_namespace: (.["k8s.namespace.name"] != null)
     }'
   ```

**Expected Results**:
- Controller log lines parse as valid JSON
- Reconciliation event entries contain `timestamp`, `severity`,
  `body`, `service.name`, `k8s.pod.name`, `k8s.namespace.name`
- Severity values use OTel severity text
- Log entries reference the reconciled resource name and namespace

**Notes**: To be filled later in the process.
