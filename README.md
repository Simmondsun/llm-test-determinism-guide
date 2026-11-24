# Reliable LLM Regression Tests in CI


> **Decision:** Use this guide to make LLM application regression tests reproducible enough for CI by separating exact contracts from sampled quality evidence; skip it for coding-agent task leaderboards or promises of deterministic model prose.
>
> **For:** application test engineers, CI maintainers, and LLM platform teams  ·  **Not for:** teams seeking byte-identical generations or a coding-agent benchmark

LLM output is not made deterministic by putting it in a unit-test runner. Reliable CI separates exact software contracts from sampled quality evidence and freezes every controllable source of drift.

This guide targets **application regression CI**. It does not compare coding agents on repository tasks (the purpose of `coding-agent-eval-starter`) and does not red-team live third-party systems.

## Freeze the test envelope

Record with every run:

- provider, exact model identifier/version where exposed, endpoint and region;
- prompt/template and system-policy digests;
- tool schemas, retrieval corpus/index commit, and application commit;
- temperature/top-p/seed **only if supported**—a seed may improve repeatability but is not a guarantee;
- evaluator type/version/prompt and cache policy;
- timestamp, request count, timeout/retry policy, and known provider changes.

A model alias that silently moves is an uncontrolled variable. If an immutable version is unavailable, report that limitation rather than calling the test deterministic.

## Test pyramid

See [`assets/ci-test-pyramid.yaml`](assets/ci-test-pyramid.yaml).

1. **Exact contract tests (per commit):** schema, tool name/argument validation, authorization, no-secret/no-PII rules, citations present where required, parser/error behavior. These should use deterministic code assertions whenever possible.
2. **Golden semantic cases (per PR):** small fixed inputs; score factual slots or rubric dimensions, not exact prose. Review fixture changes like code.
3. **Repeated sampled checks (scheduled or risk-triggered):** run each case multiple times and report success proportion plus uncertainty; compare against a frozen baseline.
4. **Human adjudication (release/risk change):** inspect borderline or high-impact cases. An LLM judge cannot be the only authority for safety or release.

## Minimal statistical protocol

For a binary criterion with `k` passes in `n` independent-ish repeats, report `k/n` and a Wilson 95% interval. Do not turn a single pass into “100% reliable.” Choose `n`, the minimum acceptable rate, and allowed regression **before** seeing the candidate run.

Example decision policy (author recommendation, not a universal standard):

- exact safety/contract case: any confirmed violation blocks;
- quality case: compare candidate Wilson interval and effect size with the frozen baseline;
- if intervals are wide or overlap materially, mark **INCONCLUSIVE**, do not retry until green;
- infrastructure/provider errors are separate from quality failures and have a bounded retry budget (for example one retry for declared transient classes).

Retries conceal flakiness if only the final attempt is reported. Retain all attempts and classify why each retry was allowed.

## Judge variance

If an LLM judge is used:

- freeze judge model/version, prompt, rubric, and examples;
- blind candidate/baseline ordering and randomize presentation;
- calibrate on a human-labeled set;
- measure disagreement and periodically re-adjudicate samples;
- require deterministic validators for hard constraints;
- never let the same unversioned judge rewrite the baseline and judge the candidate in one opaque step.

## Framework roles

- [promptfoo](https://github.com/promptfoo/promptfoo): prompt/config matrices, assertions, and test cases.
- [DeepEval](https://github.com/confident-ai/deepeval): Python test cases and evaluator metrics.
- [Ragas](https://github.com/explodinggradients/ragas): RAG-focused datasets/metrics.
- [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai): datasets, solvers, tools, scorers, and logs for more involved evals.
- [Giskard](https://github.com/Giskard-AI/giskard): application test/scan workflows—generated findings require confirmation.
- [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness): research model tasks; adjacent, not the default application CI runner.
- [OpenAI Evals](https://github.com/openai/evals): eval specifications/datasets/runner patterns; check current activity/API assumptions before adoption.
- [garak](https://github.com/NVIDIA/garak): adversarial probes; security testing is complementary and requires authorization/isolation.

These sources establish framework roles. Repeat counts, thresholds, and combinations remain project-owned decisions that must be pre-registered and validated.

## CI state machine

```text
FROZEN INPUTS
  → CONTRACT GATE (pass / confirmed block)
  → QUALITY SAMPLE (pass / regress / inconclusive)
  → HUMAN GATE when risk or uncertainty requires it
  → ACCEPT BASELINE or HOLD
```

Never automatically promote a new baseline because tests failed after a provider/model change. A baseline update needs a reviewed reason and preserved old result.

## Failure triage

Classify every red result as application regression, model/provider drift, retrieval/data drift, evaluator drift, infrastructure error, or fixture defect. Change one variable per diagnosis. Keep unknown as unknown; do not fill it with zero or pass.

## Maintenance

Re-run baseline calibration after a deliberate model/evaluator/corpus change. Preserve old reports. Add a framework only if it changes the test pyramid; popularity alone is irrelevant.

Sources: [`sources/sources.json`](sources/sources.json).

## Adjacent guides

- [coding-agent-eval-starter](https://github.com/Dallashubgit/coding-agent-eval-starter) — Use it when the unit under evaluation is a coding agent completing repository or terminal tasks.
- [code-rag-chunking-benchmark](https://github.com/Mustafa1801/code-rag-chunking-benchmark) — Use it when retrieval strategy, rather than generation alone, is the variable under test.
- [ai-research-reproducibility](https://github.com/Lindenberg668/ai-research-reproducibility) — Use it when a CI result must graduate into a reviewable research claim packet.

## License

Original protocol and CI-pyramid artifact are available under CC BY 4.0. Linked frameworks retain their licenses.

If this saved you a decision, star the repo — it is how the next reader finds it.
