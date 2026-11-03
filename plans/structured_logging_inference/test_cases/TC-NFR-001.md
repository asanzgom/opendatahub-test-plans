---
test_case_id: TC-NFR-001
source_key: RHAISTRAT-2413
objectives: [6, 7]
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-11-03"
---
# TC-NFR-001: Collection overhead and best-effort delivery baseline

**Objective**: Characterize inference hot-path overhead without new runtime logging
serialization and document best-effort stdout/stderr collection (Performance and Delivery NFRs).

**Preconditions**:

- Dedicated inference cluster and representative synthetic traffic generator; decide
  representative concurrency, duration and cluster/build with the feature owner.
- Healthy collector and Loki; resolve Monitoring CR toggle and Loki query mechanism from
  shipped design. Document collector/Loki load and retention settings.

**Test Steps**:

1. Record the runtime image and logging configuration; verify no formatter/library
   changes are required to enable the external collection pipeline.
2. Run identical warm-up and timed request workloads with collection disabled, then
   enabled. Record completed request counts, p50/p95 latency and runtime CPU for each run.
3. Record collector/Loki resource usage and inspect a bounded sample of native entries
   in pod stdout/stderr and in Loki; note any missed or delayed entries without treating
   a difference in counts as an automatic product failure.
4. Repeat runs to account for noise; attach raw measurements and test environment data.

**Expected Results**:

- No runtime logging-library or serialization change was required by collection setup.
- A reproducible before/after overhead and delivery observation is available for PM and
  engineering review; the strategy specifies no numeric performance or loss threshold,
  so this case must not claim a pass on an invented limit.

**Notes**: To be filled later in the process.
