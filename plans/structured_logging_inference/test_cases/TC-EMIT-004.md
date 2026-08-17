---
test_case_id: TC-EMIT-004
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-EMIT-004: KServe controller emits lifecycle logs

**Objective**: Verify that KServe controllers emit structured
lifecycle log entries for LLMInferenceService reconciliation events
and status transitions.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- KServe and odh-model-controller running in RHOAI application
  namespace (e.g., `redhat-ods-applications`)
- An LLMInferenceService `granite-3b-instruct` deployed in
  `inference-test-ns`

**Test Steps**:
1. Identify the controller pod:
   ```bash
   CTRL_POD=$(kubectl get pods -n redhat-ods-applications \
     -l app=odh-model-controller \
     -o jsonpath='{.items[0].metadata.name}')
   ```
2. Record current log length:
   ```bash
   LOG_BEFORE=$(kubectl logs -n redhat-ods-applications \
     "$CTRL_POD" | wc -l)
   ```
3. Trigger a reconciliation by updating the
   LLMInferenceService:
   ```bash
   oc annotate llminferenceservice granite-3b-instruct \
     -n inference-test-ns \
     test-reconcile-trigger="$(date +%s)" --overwrite
   ```
4. Wait for reconciliation and capture new log entries:
   ```bash
   sleep 5
   kubectl logs -n redhat-ods-applications "$CTRL_POD" \
     --tail=$(($(kubectl logs -n redhat-ods-applications \
       "$CTRL_POD" | wc -l) - LOG_BEFORE))
   ```
5. Filter for reconciliation and status transition entries:
   ```bash
   kubectl logs -n redhat-ods-applications "$CTRL_POD" \
     --tail=20 | jq 'select(
       .body | test("reconcil|status|transition"; "i")
     )' 2>/dev/null
   ```

**Expected Results**:
- At least one reconciliation log entry appears after the
  annotation update
- Log entries include the reconciled resource name
  (`granite-3b-instruct`) and namespace (`inference-test-ns`)
- Status transition entries record the old and new status values
- Error condition entries (if any) include the error message and
  affected resource
- All entries conform to OTel schema with `timestamp`, `severity`,
  `service.name`, `k8s.pod.name`, `k8s.namespace.name`

**Notes**: To be filled later in the process.
