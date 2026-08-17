---
test_case_id: TC-TRACE-004
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-TRACE-004: Log-to-trace navigation in OpenShift Console

**Objective**: Verify that clicking a `trace_id` in a log entry
in the OpenShift Console Observe > Logs interface navigates to
the corresponding distributed trace in Tempo.

**Preconditions**:
- LokiStack and Tempo are deployed and receiving data
- OpenShift Console logging integration is configured with a
  Tempo datasource
- At least one traced inference request has been processed,
  producing both log entries in Loki and trace spans in Tempo

**Test Steps**:
1. Send an inference request with a W3C traceparent header to
   produce correlated logs and traces
2. Open the OpenShift Console and navigate to Observe > Logs
3. Query logs to find an entry containing a `trace_id` value
4. Click the `trace_id` link in the log entry
5. Verify that the Console navigates to the Tempo trace view
6. Verify that the displayed trace contains spans corresponding
   to the inference request

**Expected Results**:
- Clicking the `trace_id` in a log entry opens the Tempo trace
  view for that trace
- The Tempo trace view displays spans from the inference request
  path (vLLM, EPP, and/or MaaS)
- The trace ID shown in Tempo matches the `trace_id` from the
  log entry

**Notes**: To be filled later in the process.
