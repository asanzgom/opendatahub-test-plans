---
test_case_id: TC-PIPE-005
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-PIPE-005: Collection pipeline is additive to existing observability

**Objective**: Verify that enabling the logging pipeline does not
disrupt existing observability resources managed by
odh-observability (Prometheus MonitoringStack, Tempo, OTel
collectors for metrics/traces).

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- odh-observability module reconciling with existing Prometheus
  MonitoringStack, Tempo, and OTel collector resources healthy
- Logging NOT yet enabled in Monitoring CR

**Test Steps**:
1. Record existing observability resource state:
   ```bash
   oc get monitoringstack,tempostack,opentelemetrycollector \
     -n redhat-ods-monitoring -o name > /tmp/pre-logging-resources.txt
   ```
2. Verify existing metrics and traces are flowing:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/prometheus-stack -- \
     promtool query instant 'up{job="vllm"}'
   ```
3. Enable logging in the Monitoring CR:
   ```bash
   oc patch monitoring/default-monitoring -n redhat-ods-applications \
     --type merge -p '{"spec":{"logging":{"enabled":true}}}'
   ```
4. Wait for reconciliation (up to 120 seconds).
5. Re-check existing observability resources:
   ```bash
   oc get monitoringstack,tempostack,opentelemetrycollector \
     -n redhat-ods-monitoring -o name > /tmp/post-logging-resources.txt
   diff /tmp/pre-logging-resources.txt /tmp/post-logging-resources.txt
   ```
6. Verify metrics and traces still flow:
   ```bash
   oc exec -n redhat-ods-monitoring deploy/prometheus-stack -- \
     promtool query instant 'up{job="vllm"}'
   ```
7. Confirm new logging resources were added (not replacing
   existing):
   ```bash
   oc get all -n redhat-ods-monitoring -l component=logging
   ```

**Expected Results**:
- Step 5: The diff shows only additions (new logging resources),
  no deletions or modifications to pre-existing observability
  resources
- Step 6: Prometheus metrics query returns results, confirming
  metrics pipeline is unaffected
- Step 7: New logging collection resources exist alongside the
  existing MonitoringStack, Tempo, and OTel collector resources

**Notes**: To be filled later in the process.
