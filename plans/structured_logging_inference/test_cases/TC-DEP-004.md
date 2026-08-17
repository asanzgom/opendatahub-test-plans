---
test_case_id: TC-DEP-004
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-DEP-004: Status condition distinguishes "not installed" from "misconfigured"

**Objective**: Verify that the Monitoring CR status message clearly
distinguishes between a missing logging dependency (operator not
installed) and a present-but-unhealthy dependency (operator
installed but misconfigured or unreachable).

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Logging enabled in Monitoring CR

**Test Steps**:
1. Ensure Loki Operator is NOT installed and read the Monitoring
   CR status condition:
   ```bash
   oc get monitoring/default-monitoring -n redhat-ods-applications \
     -o jsonpath='{.status.conditions}' | jq \
     '.[] | select(.type | contains("Logging"))' \
     > /tmp/condition-not-installed.json
   cat /tmp/condition-not-installed.json
   ```
2. Install the Loki Operator and deploy a LokiStack but
   intentionally misconfigure it (e.g., invalid storage backend):
   ```bash
   # Install Loki Operator via OLM
   # Deploy LokiStack with invalid storage config
   ```
3. Wait for reconciliation and read the status condition again:
   ```bash
   oc get monitoring/default-monitoring -n redhat-ods-applications \
     -o jsonpath='{.status.conditions}' | jq \
     '.[] | select(.type | contains("Logging"))' \
     > /tmp/condition-unhealthy.json
   cat /tmp/condition-unhealthy.json
   ```
4. Compare the two condition messages:
   ```bash
   diff /tmp/condition-not-installed.json \
     /tmp/condition-unhealthy.json
   ```

**Expected Results**:
- Step 1: Condition `reason` or `message` clearly indicates the
  operator is not installed (e.g., `reason: "DependencyNotFound"`)
  and references installation docs
- Step 3: Condition `reason` or `message` clearly indicates the
  operator is present but unhealthy (e.g.,
  `reason: "DependencyUnhealthy"`) and describes the health issue
- Step 4: The two conditions use different `reason` values and
  distinct `message` text, making it unambiguous which scenario
  is occurring

**Notes**: To be filled later in the process.
