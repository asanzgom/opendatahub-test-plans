---
test_case_id: TC-SCHEMA-005
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-SCHEMA-005: GenAI semantic convention fields on vLLM completion logs

**Objective**: Verify that vLLM completion log entries include GenAI
semantic convention fields (`gen_ai.request.model`,
`gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`) when
token usage data is available.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- LLMInferenceService deployed with vLLM runtime in namespace
  `inference-test-ns` using model `granite-3b-instruct`
- vLLM configured with `LOG_FORMAT=json`

**Test Steps**:
1. Send an inference request that produces a completion with token
   usage:
   ```bash
   curl -s -X POST \
     "https://granite-3b-instruct-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b-instruct", "prompt": "Explain containers in one sentence.", "max_tokens": 30}'
   ```
2. Capture vLLM logs and filter for completion entries:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20 | jq 'select(.body | test("complet"; "i"))' 2>/dev/null
   ```
3. Validate presence of GenAI semconv fields on the completion
   entry:
   ```bash
   kubectl logs -n inference-test-ns \
     $(kubectl get pods -n inference-test-ns -l app=granite-3b-instruct \
       -o jsonpath='{.items[0].metadata.name}') \
     --tail=20 | jq 'select(.body | test("complet"; "i")) | {
       model: .["gen_ai.request.model"],
       input_tokens: .["gen_ai.usage.input_tokens"],
       output_tokens: .["gen_ai.usage.output_tokens"]
     }' 2>/dev/null
   ```

**Expected Results**:
- Completion log entries contain `gen_ai.request.model` with value
  `granite-3b-instruct`
- Completion log entries contain `gen_ai.usage.input_tokens` as a
  positive integer
- Completion log entries contain `gen_ai.usage.output_tokens` as a
  positive integer matching the number of generated tokens
- All three fields are present as top-level or nested attributes in
  the JSON log entry

**Notes**: To be filled later in the process.
