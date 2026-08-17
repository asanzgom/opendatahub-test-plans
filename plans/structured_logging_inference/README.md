# Structured Logging for Inference Components

Structured JSON logging for RHOAI inference components (vLLM, llm-d EPP,
MaaS, KServe controllers) with OTel-based log collection and Loki
storage.

## Links

- **Strategy**: [RHAISTRAT-2413](https://redhat.atlassian.net/browse/RHAISTRAT-2413)
- **Source RFE**: [RHAIRFE-2776](https://redhat.atlassian.net/browse/RHAIRFE-2776)
- **Test Plan**: [TestPlan.md](TestPlan.md)
- **Gaps**: [TestPlanGaps.md](TestPlanGaps.md)

## Test Cases

**51 test cases** (24 P0, 24 P1, 3 P2) across 12 categories.

- **Index**: [test_cases/INDEX.md](test_cases/INDEX.md)
- **Review**: [TestPlanReview.md](TestPlanReview.md) (10/10, Ready)

## Test Implementation

Automated tests will be implemented in the appropriate component
repositories (odh-observability, llm-d, models-as-a-service) and in
the downstream E2E test suite for cross-component integration scenarios.
