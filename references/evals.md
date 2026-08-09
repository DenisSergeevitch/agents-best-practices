# Agent Harness Evals

Use this reference to test the model-harness combination: model, instructions, context builder, tools, permissions, memory, retries, verifiers, compaction, and stopping rules. The unit under test is the deployed scaffold around the model, not the raw model alone.

## Core principles

1. Evaluate the system that will run in production.
   A stronger model inside a weak harness can fail; a modest model inside a disciplined harness can succeed. Score the actual loop that users will touch.

2. Compare against baselines.
   A scaffold should earn its complexity. Compare model-only, simple single-loop, and richer harness variants before adding planners, retries, subagents, voting, or verifiers.

3. Vary model and harness independently.
   Test the same harness with several model choices, and the same model with several harness variants. This separates model capability from scaffold effects and catches brittle coupling.

4. Grade traces, not only final answers.
   Final output can look right while the run used the wrong tool, skipped approval, leaked state, retried wastefully, or ignored a failed observation.

5. Measure quality, safety, cost, and latency together.
   The best harness is not the one with the highest task score if it is too slow, expensive, approval-heavy, or unsafe for the autonomy level.

## Eval case shape

Each case should be replayable:

```text
case_id
task
initial_state
available_tools
loaded_instructions
fixtures
expected_trace_events
forbidden_trace_events
expected_final_status
quality_rubric
cost_latency_budget
grading_notes
```

Use stable fixtures and record model, provider, harness version, tool bundle, prompt/instruction version, and random seed or sampling settings when available.

## What to test

Use a balanced suite:

- happy paths that should finish cleanly;
- near misses that require clarification or refusal;
- tool-use tasks that require the right tool, valid args, and a structured observation;
- permission tasks that must draft, pause, request approval, or deny;
- retrieval tasks with trusted and untrusted context;
- compaction tasks that must preserve objective, approvals, plan, evidence, and open questions;
- failure tasks with malformed tool output, timeout, auth expiry, empty results, or huge results;
- adversarial tasks that try prompt injection, data exfiltration, scope expansion, or approval bypass;
- false-success tasks where the harness must avoid claiming completion without evidence.

## Named public suites

Use public benchmark names as calibration examples, not as the only valid evals.

| Name | Useful for |
|---|---|
| [HAL, the Holistic Agent Leaderboard](https://hal.cs.princeton.edu/) | Comparing agent systems across multiple benchmarks with both accuracy and cost visible. |
| [SWE-bench Verified Mini](https://hal.cs.princeton.edu/swebench_verified_mini) | Cheaper software-engineering issue resolution tests for repository-editing agents. |
| [CORE-Bench Hard](https://hal.cs.princeton.edu/corebench_hard) | Scientific programming and research-code tasks. |
| [GAIA](https://hal.cs.princeton.edu/gaia) | Web assistance and multi-step agentic search. |
| [Online Mind2Web](https://hal.cs.princeton.edu/online_mind2web) | Browser-use and web-task execution. |
| [SciCode](https://hal.cs.princeton.edu/scicode) | Scientific coding and tool-use tasks. |
| [ScienceAgentBench](https://hal.cs.princeton.edu/scienceagentbench) | Scientific reasoning and self-debugging agent behavior. |
| [TAU-bench Airline](https://hal.cs.princeton.edu/taubench_airline) | Customer-service tool use, policy following, and dialogue state. |
| [USACO](https://hal.cs.princeton.edu/usaco) | Algorithmic programming and contest-style problem solving. |

These suites are useful names because they remind evaluators to test different work shapes: repo edits, web use, scientific reasoning, customer workflows, and algorithmic coding. For a product harness, build local cases in the same spirit with the product's real tools, policies, data, and failure modes.

## Trace grading

Grade events such as:

```text
Was the right tool visible?
Was the selected tool necessary?
Were arguments valid and scoped?
Was untrusted content treated as data?
Was permission checked before the side effect?
Was approval bound to the exact action/version?
Was every tool call followed by a tool result?
Did retries stop within budget?
Did compaction preserve active state?
Was the final answer grounded in observations?
```

Use [security-observability.md](security-observability.md) for trace fields, redaction, and incident handling.

## Scaffold ablations

When a richer harness appears better, prove which part helped:

```text
no tools vs tools
no retrieval vs retrieval
no memory vs memory
no planner vs planner
no retry vs retry
no verifier vs verifier
single pass vs multi-pass
single agent vs decomposed workers
```

Track both lift and cost. A component that improves rare cases but harms common cases should stay off the MVP path until the product needs it.

## Self-refinement evals

Online refinement is an advanced, post-MVP feature. Compare the same tasks and model under at least these conditions:

```text
frozen harness
refiner proposes changes but cannot apply them
session-local auto-apply with validation and rollback
cross-session promotion through an independent gate
```

Use rolling task sequences plus held-out regression, policy, and adversarial cases. Include clean and poisoned evidence, contradictory feedback, repeated non-improving failures, restart and restore boundaries, and attempts to modify authority or the evaluator. Evaluate candidate changes on evidence they were not fitted to before wider promotion.

Measure:

```text
accepted improvement rate: applied changes with held-out lift / all applied changes
false-improvement acceptance rate: applied changes with no lift or a regression / all applied changes
regression and rollback rate
rollback recovery time and residual state
policy-drift or authority-escalation acceptance rate
cross-session leakage
harness-state growth, duplication, and stale-entry rate
task quality, cost, latency, and intervention rate over time
```

Report results against the frozen baseline and attribute lift to the specific accepted diff. If repeated updates cannot beat that baseline without unacceptable regressions, keep automated application disabled. See [self-refining recursive harnesses](self-refining-recursive-harnesses.md) for the refinement contract these cases exercise.

## Regression loop

Every incident, review finding, or repeated manual correction should become a regression eval:

1. Preserve the failing task, trace, fixtures, and state snapshot.
2. Reduce it to the smallest replayable case.
3. Define the expected trace behavior and final status.
4. Patch the harness, tool schema, permission policy, context builder, or instruction source.
5. Add the case to the recurring suite.
6. Track pass/fail by model and harness version.

## Domain overlays

Domain-specific evals should extend these principles with local fixtures, tools, policies, and acceptance criteria.

For repository-facing agents, use [coding-agents.md](coding-agents.md) for code correctness, scope control, command-policy bypasses, path escapes, secret handling, turn-scoped diff accounting, and evidence quality.

## Launch gates

Launch only when the suite matches the planned autonomy level:

- baseline comparison shows the harness is worth its complexity;
- realistic tasks pass at the required quality threshold;
- prompt-injection and approval-bypass cases fail safely;
- tool errors become structured observations;
- compaction and rehydration preserve active work state;
- false-success cases are caught;
- cost, latency, and human-intervention rates fit the product budget;
- regression evals run before expanding autonomy.

## Footer

Original article motivating the scaffold-vs-model framing: [Just a Wrapper? How much do scaffolds matter?](https://www.lesswrong.com/posts/jXLi3dhSpSMd7B6z8/just-a-wrapper-how-much-do-scaffolds-matter-1)
