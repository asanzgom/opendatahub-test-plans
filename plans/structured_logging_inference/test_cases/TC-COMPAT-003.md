---
test_case_id: TC-COMPAT-003
source_key: RHAISTRAT-2413
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-COMPAT-003: kubectl logs works regardless of log format

**Objective**: Verify that `kubectl logs` returns readable output in
both JSON and text log modes, ensuring no disruption to existing
developer workflows.

**Preconditions**:
- Two vLLM deployments available: one with `LOG_FORMAT=json`
  (default), one with `LOG_FORMAT=text`
- Both deployments have processed at least one inference request

**Test Steps**:
1. Capture logs from the JSON-format deployment:
   ```bash
   oc logs $(oc get pods -l app=vllm-json -o name | head -1) \
     --tail=10
   ```
2. Verify the JSON output is readable (valid UTF-8, no binary
   content, no truncation).
3. Capture logs from the text-format deployment:
   ```bash
   oc logs $(oc get pods -l app=vllm-text -o name | head -1) \
     --tail=10
   ```
4. Verify the text output is readable (valid UTF-8, standard
   Python log format, no binary content).
5. Test log filtering with `kubectl logs` flags in both modes:
   ```bash
   # Timestamp-based filtering
   oc logs <pod> --since=5m

   # Follow mode
   oc logs <pod> --follow --tail=5
   # (send a request while following, verify new lines appear)
   ```

**Expected Results**:
- `oc logs` / `kubectl logs` returns readable output in both
  JSON and text modes
- No encoding errors, binary artifacts, or truncated lines in
  either mode
- `--since`, `--tail`, and `--follow` flags work identically in
  both modes
- Switching between `LOG_FORMAT=json` and `LOG_FORMAT=text`
  requires only a pod restart, not a redeployment

**Notes**: To be filled later in the process.
