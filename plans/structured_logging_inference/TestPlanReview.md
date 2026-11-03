---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
score: 7
pass: false
verdict: Rework
scores:
  specificity: 2
  grounding: 2
  scope_fidelity: 1
  actionability: 0
  consistency: 2
auto_revised: false
last_updated: '2026-11-03'
before_score: null
before_scores: null
error: null
---
# Test Plan Review — Inference Request Log Collection

**Current strategy (Jira description revised 2026-08-26): 7/10 — Rework.**

The original 10/10 Ready review assessed the superseded August 17 strategy with
20 acceptance criteria. It is not comparable to or valid for the current five-AC scope.
This is a new assessment, not a decline in the quality of the old plan.

## Rubric Scores

| Criterion | Score | Evidence |
|-----------|-------|----------|
| Specificity | 2/2 | Feature-specific P0 dependency/collection and P1 tenant/exclusion cases. |
| Grounding | 2/2 | Four strategy-grounded interfaces; unknown CR fields, resource kinds and RBAC not invented. |
| Scope Fidelity | 1/2 | Five ACs and three NFRs covered; P1 opt-in content logging has no AC and needs a coverage decision. |
| Actionability | 0/2 | Pipeline/CR/RBAC contracts, native request-log samples and safe failure procedure are missing. |
| Consistency | 2/2 | Section 4 matches coverage matrices; priorities and AC5 full-output/selector checks align. |

**Total: 7/10 — Verdict: Rework.** The rubric requires every criterion to be
nonzero for Revise or Ready; a zero in Actionability forces Rework even at 7/10.

## Grounding and Acceptance Criteria

- **AC #1** → Objective #1; TC-E2E-001 (request stdout/stderr in Loki).
- **AC #2** → Objective #2; TC-E2E-002 and TC-E2E-006 (missing/unreachable dependency).
- **AC #3** → Objective #3; TC-E2E-003 (logging disabled).
- **AC #4** → Objective #4; TC-E2E-004 (bidirectional tenant isolation).
- **AC #5** → Objective #5; TC-E2E-005 (controller reconciliation excluded).
- **NFRs** → Objectives #6–#8; TC-NFR-001 and TC-E2E-001/002/006.

The high-level P1 opt-in request/response content-logging requirement has no AC,
flag name or runtime-specific contract. Do not fabricate either; obtain a scope
and validation decision before adding a case. Console filtering and existing trace
correlation are context, not new log-format acceptance criteria.

## Required Source Documents and Follow-up

1. Sprint-zero pipeline choice and resource/namespace selectors (Path A vs Path B).
2. Monitoring CR logging toggle, condition and reconciliation API (RHAISTRAT-2416).
3. Loki tenancy/RBAC/query access and synthetic tester roles.
4. Representative native vLLM, EPP and MaaS request-time log examples.
5. Safe missing/unreachable backend fault-injection and recovery procedure.

See [TestPlanGaps.md](TestPlanGaps.md) for these and the version and performance
follow-ups. Re-score after the implementation contracts exist. Do not reopen the
closed PR as merge-ready or execute the conditional cases as written.

## Revision History

- 2026-11-03: Rebased plan on current five-AC strategy, replaced 51 stale cases
  with seven focused drafts; independently scored after consistency corrections.
