---
test_case_id: TC-PIPE-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PIPE-001: Logging collection resources created when Monitoring CR enables logging

**Objective**: Verify that the odh-observability controller creates
logging collection resources that route inference namespace logs to
Loki when the Monitoring CR logging subsection is enabled.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Loki Operator deployed with a healthy LokiStack instance
- odh-observability module reconciling via DSCInitialization
- At least one LLMInferenceService deployed with a vLLM runtime in
  a user namespace (e.g., `inference-ns-a`)
- Monitoring CR logging subsection is NOT yet enabled

**Test Steps**:
1. Verify no logging collection resources exist in the
   odh-observability namespace by running:
   ```bash
   oc get all -n redhat-ods-monitoring -l app.kubernetes.io/part-of=odh-observability \
     --selector component=logging
   ```
2. Enable the logging subsection in the Monitoring CR:
   ```bash
   oc patch monitoring/default-monitoring -n redhat-ods-applications \
     --type merge -p '{"spec":{"logging":{"enabled":true}}}'
   ```
3. Wait for the odh-observability controller to reconcile (up to
   120 seconds).
4. List logging collection resources:
   ```bash
   oc get all -n redhat-ods-monitoring -l component=logging
   ```
5. Inspect the collection resource configuration to confirm it
   references the Loki backend endpoint:
   ```bash
   oc get <collection-resource> -n redhat-ods-monitoring -o yaml
   ```

**Expected Results**:
- Step 1: No logging collection resources exist before enablement
- Step 4: At least one logging collection resource is created
  (OTel collector config or CLO ClusterLogForwarder, depending on
  the collection path chosen during sprint-zero)
- Step 5: The collection resource configuration includes an output
  section referencing the LokiStack endpoint and correct
  authentication (e.g., `lokistack-application-logs-writer`
  service account)

**Validation**:
- Query Monitoring CR status conditions to confirm logging is
  reported as healthy:
  ```bash
  oc get monitoring/default-monitoring -n redhat-ods-applications \
    -o jsonpath='{.status.conditions[?(@.type=="LoggingReady")]}'
  ```
- Condition should show `status: "True"`

**Notes**: To be filled later in the process.
