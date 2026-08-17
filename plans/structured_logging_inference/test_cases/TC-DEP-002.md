---
test_case_id: TC-DEP-002
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-DEP-002: Monitoring CR status indicates unhealthy Loki backend

**Objective**: Verify that the Monitoring CR status includes a
condition indicating the Loki backend is unhealthy when Loki is
deployed but unreachable (AC 16).

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Loki Operator installed with LokiStack CRD available
- LokiStack deployed but made unreachable (e.g., scale LokiStack
  pods to 0 or delete the LokiStack service)
- Logging enabled in Monitoring CR

**Test Steps**:
1. Scale the LokiStack pods to zero to simulate an unhealthy
   backend:
   ```bash
   oc scale statefulset -n openshift-logging \
     --selector app.kubernetes.io/name=lokistack --replicas=0
   ```
2. Wait for the odh-observability controller to detect the
   unhealthy state (up to 120 seconds).
3. Read the Monitoring CR status conditions:
   ```bash
   oc get monitoring/default-monitoring -n redhat-ods-applications \
     -o jsonpath='{.status.conditions}' | jq \
     '.[] | select(.type | contains("Logging"))'
   ```
4. Restore the LokiStack:
   ```bash
   oc scale statefulset -n openshift-logging \
     --selector app.kubernetes.io/name=lokistack --replicas=1
   ```
5. Wait for reconciliation and re-read status conditions.

**Expected Results**:
- Step 3: A status condition exists with:
  - `status: "False"`
  - `message` identifying that the Loki backend is unhealthy or
    unreachable (distinct from "not installed")
- Step 5: After restoring LokiStack, the condition transitions to
  `status: "True"` within the reconciliation period

**Notes**: To be filled later in the process.
