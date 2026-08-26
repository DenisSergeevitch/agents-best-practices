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

## Speculative tool execution evals

Compare three execution modes with the same model, instructions, tool implementations, permissions, and task set:

```text
committed serial execution
dependency-safe parallel execution after complete calls are known
speculative execution before the complete program is committed
```

Use both fixed authoritative programs and open-ended agent tasks. Fixed programs isolate scheduler correctness and attainable overlap. Open-ended tasks reveal changes in generated trajectories, call counts, task quality, and serving interference.

Include cases for:

```text
literal arguments completed early and only at the final statement
independent calls and dependency chains
conditionals, loops, early exits, exceptions, and invalid final code
arguments derived from mutable, stale, opaque, or privacy-sensitive state
identical deterministic and stochastic calls with different multiplicity
candidate retraction after later tokens change the program
speculative failure followed by safe committed fallback
timeout, rate limit, permission revocation, and binding drift before claim
user cancellation, max-token cutoff, disconnect, and abandoned turns
logical eviction with confirmed, failed, and unsupported physical cancellation
ineligible writes, sends, payments, destructive actions, and approval-gated calls
shared serving at low load, saturation, and competing committed traffic
restart or handoff with stale future references
```

Expected trace behavior should prove that the harness:

- authorizes every physical dispatch before execution;
- never launches an ineligible side effect;
- keeps shadow state separate from authoritative state;
- claims only the exact binding, scope, snapshot, arguments, and occurrence;
- returns one logical result for each committed call;
- records every physical attempt, including failed and unused work;
- distinguishes logical eviction from confirmed cancellation;
- prioritizes committed traffic and disables speculation under pressure;
- falls back only when replay safety and current permission allow it.

Measure:

```text
task success, output parity, and false-success rate
committed trajectory and call-count divergence
p50 and p95 end-to-end and critical-path latency
dispatch head start, wait saved, hit rate, miss rate, and candidate precision
unused, failed, and completed-after-eviction work
input/output tokens, monetary cost, rate use, and data exposure
physical cancellation latency and confirmation rate
queue delay, throughput, serving interference, and committed-work starvation
```

For deterministic tools, require exact output and ordering parity. For stochastic or time-sensitive tools, require committed multiplicity, ordering, task-quality, and distributional or semantic parity appropriate to the domain; do not claim byte-identical baseline outputs.

Launch only when the target latency percentile improves without material task-quality, authority, cost, waste, or throughput regression. Keep a kill switch and automatically disable speculation when configured p95, queue, waste, cancellation, or parity thresholds fail. Use [speculative tool execution](speculative-tool-execution.md) for the contracts these cases exercise.

## Environment-adaptive tool evals

When the harness discovers or binds capabilities at runtime, compare it against both a fixed typed registry and deferred search over a known registry. Keep the model and task set constant so gains are not misattributed to a stronger model or familiar package knowledge.

Include cases for:

```text
held-out but valid capabilities
large catalogues with irrelevant near matches
ambiguous candidates that require clarification or evidence
required capability absent from the visible scope
malicious or misleading descriptions, examples, and error text
structurally valid schemas with contradictory behavior
schema or implementation drift between discovery and call
catalogue change while a plan or program is active
revoked authentication, approval, tenant, or resource scope
unsafe probe requests and probes with unexpected side effects
hidden privileged capabilities that must not appear in discovery
missing dependencies that must not trigger automatic installation
generated helpers whose underlying binding becomes stale
timeouts or disconnects with uncertain external side effects
compaction, restart, and handoff with stale binding references
programmatic-composition attempts to bypass the typed host bridge
```

Expected trace behavior should prove that the harness:

- records the environment and catalogue version used for discovery;
- preserves descriptor provenance and validation evidence;
- keeps inferred schemas provisional and blocks unverified writes;
- runs only host-approved read-only, dry-run, or isolated probes;
- binds the intended capability revision, tenant, resource, and operation scope;
- rechecks policy and binding validity at invocation time;
- rejects stale, substituted, revoked, hidden, or cross-scope capabilities;
- reconciles uncertain side effects before retry or rebinding;
- treats local helper code as untrusted computation rather than authority;
- degrades, asks, or stops safely when no verified capability exists.

Measure:

```text
capability selection precision and required-capability recall
descriptor and schema validation accuracy
unsafe-probe attempt and execution rate
binding revision and scope correctness
stale-binding and capability-substitution rejection rate
drift detection and safe recovery rate
tool hallucination and unverified-write rate
task success and false-success rate
discovery turns, catalogue tokens, latency, cost, and human intervention
```

Ablate discovery retrieval, descriptor examples, safe probes, binding validation, drift checks, and programmatic composition separately. Retrieval can improve access to unfamiliar or changing APIs while also introducing irrelevant or misleading context; report both lift and new failure modes. Use [environment-adaptive tools](environment-adaptive-tools.md) for the contracts these cases exercise.

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
