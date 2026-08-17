---
test_case_id: TC-DEP-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-DEP-001: Monitoring CR status indicates missing Loki backend

**Objective**: Verify that the Monitoring CR status includes a
condition identifying the missing Loki operator with an
installation documentation reference when Loki is not installed.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Loki Operator is NOT installed (no LokiStack CRD available)
- Logging enabled in Monitoring CR

**Test Steps**:
1. Confirm the Loki Operator CRD is not present:
   ```bash
   oc get crd lokistacks.loki.grafana.com 2>&1
   ```
2. Enable logging in the Monitoring CR:
   ```bash
   oc patch monitoring/default-monitoring -n redhat-ods-applications \
     --type merge -p '{"spec":{"logging":{"enabled":true}}}'
   ```
3. Wait for the odh-observability controller to reconcile (up to
   120 seconds).
4. Read the Monitoring CR status conditions:
   ```bash
   oc get monitoring/default-monitoring -n redhat-ods-applications \
     -o jsonpath='{.status.conditions}' | jq .
   ```
5. Check the DSCInitialization status for propagated conditions:
   ```bash
   oc get dscinitializations -o jsonpath='{.items[0].status.conditions}' \
     | jq '.[] | select(.type | contains("Logging"))'
   ```

**Expected Results**:
- Step 1: CRD not found (confirms Loki Operator is absent)
- Step 4: A status condition exists with:
  - `type` containing a logging dependency reference (e.g.,
    `LoggingDependencyReady`)
  - `status: "False"`
  - `message` identifying the specific missing operator (Loki
    Operator) and referencing installation documentation
- Step 5: The condition is propagated to DSCInitialization status

**Notes**: To be filled later in the process.
