---
test_case_id: TC-PIPE-004
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PIPE-004: Logging resources absent when Monitoring CR does not enable logging

**Objective**: Verify that no logging collection resources are
created when the Monitoring CR does not have the logging subsection
enabled (AC 9).

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- odh-observability module reconciling via DSCInitialization
- Monitoring CR exists but logging subsection is either absent or
  explicitly set to `enabled: false`

**Test Steps**:
1. Confirm the Monitoring CR does not enable logging:
   ```bash
   oc get monitoring/default-monitoring -n redhat-ods-applications \
     -o jsonpath='{.spec.logging}'
   ```
2. List all resources in the odh-observability namespace with the
   logging component label:
   ```bash
   oc get all -n redhat-ods-monitoring -l component=logging
   ```
3. Verify no ClusterLogForwarder or OTel collector configs exist
   for logging:
   ```bash
   oc get clusterlogforwarder -A 2>/dev/null | grep -i inference || true
   ```

**Expected Results**:
- Step 1: The logging subsection is absent or shows
  `enabled: false`
- Step 2: No resources with the `component=logging` label exist
- Step 3: No logging pipeline resources targeting inference
  namespaces exist

**Notes**: To be filled later in the process.
