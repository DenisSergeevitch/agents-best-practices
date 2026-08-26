# Speculative Tool Execution

## Status and scope

Speculative tool execution is an experimental, post-MVP scheduling profile. It lets a harness start an eligible tool call before the model's complete program or action has been committed, then use the result only if authoritative execution reaches the exact matching call.

Use this profile only after a reliable sequential baseline and ordinary committed-call parallelism have been measured. It is most relevant to code-mode or programmatic-tool harnesses where expensive calls become identifiable early enough to overlap with continued model generation or local program execution.

This reference owns only speculation-specific contracts. Reuse the existing canonical guidance for:

| Concern | Source of truth |
|---|---|
| ordinary loop budgets, retries, stopping, and committed parallel calls | [agentic loop](agentic-loop.md) |
| tool schemas, risk classes, permissions, approvals, and sandboxes | [tools and permissions](tools-and-permissions.md) |
| program environments, recursive calls, and retained children | [self-refining recursive harnesses](self-refining-recursive-harnesses.md) |
| late-bound discovery, validation, binding, and drift | [environment-adaptive tools](environment-adaptive-tools.md) |
| threat modeling, trace storage, redaction, and incident response | [security and observability](security-observability.md) |
| evaluation methodology, ablations, metrics, and launch gates | [evals](evals.md) |

## Contents

- [Taxonomy and boundaries](#taxonomy-and-boundaries)
- [Applicability gate](#applicability-gate)
- [Authority boundary](#authority-boundary)
- [Eligibility contract](#eligibility-contract)
- [State model](#state-model)
- [Launch and claim lifecycle](#launch-and-claim-lifecycle)
- [Call identity and multiplicity](#call-identity-and-multiplicity)
- [Shadow execution and dependency resolution](#shadow-execution-and-dependency-resolution)
- [Permissions and revocation](#permissions-and-revocation)
- [Budgets, backpressure, and cancellation](#budgets-backpressure-and-cancellation)
- [Failure and recovery semantics](#failure-and-recovery-semantics)
- [Observability and evidence](#observability-and-evidence)
- [Safe build sequence](#safe-build-sequence)
- [Evaluation requirements](#evaluation-requirements)
- [Anti-patterns](#anti-patterns)

## Taxonomy and boundaries

Keep these mechanisms separate:

| Mechanism | When work begins | Defining contract |
|---|---|---|
| committed serial execution | after the complete call is accepted | one call runs at a time in committed order |
| committed parallel execution | after a complete independent call set is accepted | known dependency-safe calls run concurrently |
| future-based asynchronous execution | after an explicit call is committed | the call returns a future while execution continues |
| speculative tool execution | before the complete program or action is committed | eligible predicted work is hidden until an exact committed call claims it |
| workflow orchestration | after a durable workflow artifact is admitted | packets, workers, verification, integration, and resumability |

Programmatic tool calling is an action representation: generated code composes calls with variables, loops, branches, and local transformations. Speculation is an optional scheduling overlay on that representation. It is not a new autonomy level, model architecture, training method, permission source, or workflow system.

Speculative tool execution may overlap tool work with:

- continued model token generation;
- independent local computation in the generated program;
- other eligible speculative calls whose dependencies are already resolved.

The committed program remains the sole source of truth. A speculative launch predicts work; it does not commit the call or make partial model output authoritative.

## Applicability gate

Consider this profile only when measurements show all of the following:

- tool latency is material on the end-to-end critical path;
- complete tool identities and arguments commonly appear before generation ends;
- calls are safe to execute and discard before control flow is committed;
- unused work can be bounded, observed, and preferably cancelled;
- the serving system has spare capacity or workload-aware admission control;
- task quality and logical call semantics can be compared against speculation-off.

Prefer ordinary committed parallelism when the complete call set is available cheaply. Leave speculation disabled when calls are fast, arguments arrive at the end of generation, dependencies are mostly sequential, queue pressure is high, cancellation is ineffective, or the waste budget would dominate the possible latency saving.

## Authority boundary

Partial model output never grants authority.

The model or model-directed program may reveal a candidate call. The trusted host must:

- resolve the candidate to an already admitted tool binding;
- validate the complete candidate arguments;
- decide whether the operation is eligible for speculation;
- authorize the physical invocation before dispatch;
- enforce separate speculative resource budgets;
- isolate speculative computation from authoritative state;
- decide whether a later committed call may claim the result;
- cancel, evict, redact, or discard unused work;
- record every physical attempt and logical outcome.

Approval-required operations and actions whose intended effect changes user-visible, business, financial, identity, security, or other authoritative state must not execute speculatively. The harness may prepare a local draft or validate arguments early, but the intended external effect waits for the ordinary commit path.

## Eligibility contract

Speculation must be an explicit host-owned property, not an inference from a tool name, description, HTTP verb, or model claim.

A useful eligibility record includes:

```text
tool binding ID and exact implementation version
speculation class: never | discardable occurrence | deterministic reusable
side-effect and external-observability class
principal, tenant, resource, and data scope
input snapshot or freshness semantics
privacy and data-residency constraints
estimated latency, token use, monetary cost, and rate-limit footprint
concurrency and shared-resource footprint
per-call eligibility gate
claim-key function and reuse lifetime
cancellation support and confirmation semantics
```

“Discardable” should mean that an unused result can be ignored without changing authoritative application state or producing the user- or business-visible effect controlled by the committed program. It does not mean physically effect-free: retrieval, search, and submodel calls can still disclose data, consume quota, create provider logs, incur cost, affect caches, or contend with committed work. Those operational effects must be acceptable under policy and budget before dispatch.

Determinism is separate from discardability. A stochastic submodel call may be safe to discard but must not be reused across identical occurrences unless the contract explicitly permits that behavior.

## State model

Keep speculative state outside model context in a host-owned store. A candidate record should contain:

```text
candidate and turn IDs
source span or program-node digest
tool binding and implementation version
principal, tenant, scope, policy, and environment generations
input snapshot version
canonical argument digest and occurrence index
eligibility and permission decisions
dispatch, ready, claim, eviction, and cancellation timestamps
physical attempt status and cost
logical committed-call linkage
result or error artifact reference with redaction policy
```

Use an explicit lifecycle:

```text
candidate
  -> rejected
  -> admitted -> dispatched -> ready | failed
                         \-> cancel_requested
ready -> claimed | evicted
failed -> safe committed fallback | terminal committed failure
cancel_requested -> cancellation_confirmed | completed_after_eviction | cancellation_unknown
```

Logical eviction and physical cancellation are different facts. Removing a result from the claim store does not prove that remote or background work stopped.

## Launch and claim lifecycle

1. The streaming parser identifies a syntactically complete candidate call. Do not guess unfinished arguments.
2. The harness resolves arguments using literals, immutable snapshot data, or bounded isolated computation. Uncertainty produces no launch.
3. The host resolves the exact tool binding, validates arguments, checks eligibility and permission, and admits the call under speculative budgets.
4. The executor launches the physical call and stores a future that is invisible to the model and authoritative program.
5. Model generation continues. Candidate plans may be retracted as later tokens change the program.
6. After the complete program passes ordinary validation, authoritative execution begins against real state.
7. When execution reaches a tool call, the host recomputes its claim identity under current policy and environment state.
8. An exact eligible hit claims the future. A miss follows the normal committed execution path.
9. The model receives exactly one logical result at the authoritative call point, regardless of how many physical attempts occurred.
10. Completion, abort, timeout, invalid code, or user cancellation evicts all unclaimed candidates and requests physical cancellation where supported.

The speculative parser must not change the generated program, expose future values early, choose a different branch, or suppress the real call merely because a similar candidate exists.

## Call identity and multiplicity

A tool name plus serialized arguments is rarely a sufficient production claim key. Bind identity to all state that can change semantics:

```text
exact tool implementation or model version
tool and model configuration
principal, tenant, resource, and authority scope
policy and environment generation
data snapshot, cache, or freshness version
canonical arguments
occurrence index for non-reusable calls
allowed reuse lifetime
```

Do not log raw sensitive arguments merely to build the key; use canonical digests and protected artifact references.

For deterministic reusable tools, one result may satisfy repeated identical calls only when the declared snapshot and reuse lifetime still match. For stochastic or time-sensitive tools, preserve committed occurrence order: the kth identical committed call may claim only the kth compatible candidate. This preserves multiplicity, not exact equivalence with a later baseline sample.

## Shadow execution and dependency resolution

A code-mode harness may use a parser, abstract interpreter, or disposable shadow runtime to resolve arguments and dependencies before the complete program is available.

The shadow boundary must:

- receive copied or immutable working data, not authoritative mutable objects;
- exclude credentials, approval records, live binding handles, sockets, ambient network access, and privileged host objects;
- reach external capabilities only through the same policy-mediated host bridge;
- block or skip operations whose purity, dependencies, or control flow are uncertain;
- keep mutations, stdout, exceptions, and partial state disposable;
- apply hard compute, memory, output, and wall-time limits;
- stop without affecting the real program when parsing or execution fails.

An in-process interpreter with blocked builtins or imports is not a security sandbox. Treat such mechanisms as correctness filters inside an already isolated environment.

Dependency-aware speculation may launch a call when all required inputs are resolved and may chain later candidates from completed speculative values. It must not invent values for unresolved dependencies, evaluate unsafe helper functions, or assume that a conditional branch or loop iteration will execute. Conservative misses are preferable to unsafe or expensive bets.

## Permissions and revocation

The speculative dispatch is a real physical invocation, so ordinary authorization happens before launch, not when the result is claimed. The candidate receives no authority from partial output or from a prior similar call.

Recheck visibility, policy, binding validity, and data scope at claim time. If authorization, tenant, resource scope, environment generation, or binding state changed after dispatch, deny the claim and discard or redact the result. Do not release formerly authorized data merely because the work already finished.

If a speculative operation unexpectedly changes authoritative state or creates a prohibited user- or business-visible effect, treat it as a policy violation and incident. Do not normalize the behavior by adding rollback prose or retrying it automatically.

## Budgets, backpressure, and cancellation

Speculation needs separate hard budgets in addition to the normal loop budget:

```text
max speculative dispatches per turn
max speculative calls in flight
max speculative wall time
max speculative input and output tokens
max speculative monetary cost
max argument and result bytes
max per-tool and per-tenant rate
max unclaimed or failed-work cost
```

Committed work outranks speculative work for queues, rate limits, compute, and connection pools. Reduce or disable speculation when service saturation, queue delay, cancellation lag, hit rate, or waste crosses a configured threshold.

Cancellation is best effort unless the executor confirms otherwise. Record `cancellation_requested`, `cancellation_confirmed`, `completed_after_eviction`, or `cancellation_unknown`. Calls that cannot be stopped continue to count against concurrency, rate, time, and cost budgets until their physical terminal state is known.

## Failure and recovery semantics

Speculation must degrade to committed behavior without hiding changed physical execution.

Handle at least:

- incomplete or invalid streamed syntax;
- unresolved, stale, or mutated candidate arguments;
- shadow exceptions, timeouts, and resource exhaustion;
- duplicate or reordered identical calls;
- candidate retraction after later tokens;
- future failure or timeout before claim;
- permission or binding revocation between launch and claim;
- abandoned output, invalid final code, user cancellation, and process loss;
- server saturation and speculative work delaying committed work;
- cancellation failure or an uncertain physical outcome.

A claim miss runs the ordinary tool only if the committed call is still authorized and its retry semantics permit another physical attempt. A failed speculation followed by committed execution can double cost and attempts; make that behavior explicit and count both. Never use fallback or retry for an operation with an uncertain external effect.

Keep candidates turn-local by default. After restart or handoff, do not restore a serialized future as live authority. Durable speculative work requires a host-owned execution record, executor reconciliation, current policy validation, and an exact claim key; otherwise evict it and use the normal committed path.

## Observability and evidence

Use the canonical storage, redaction, and trace rules in [security and observability](security-observability.md). A speculative trace must distinguish:

- candidate detection from permissioned dispatch;
- logical committed calls from physical attempts;
- dispatch source, argument digest, binding, scope, and snapshot;
- hit, miss, wait, failure, retraction, eviction, and cancellation outcome;
- launch head start and critical-path time saved;
- tokens, monetary cost, rate consumption, queue delay, and completed waste;
- committed-call ordering and task or output parity evidence.

Do not expose speculative plumbing to the model as extra tool messages. The authoritative call still receives one normal tool result; speculation events remain host trace events unless a normal committed error must be returned.

## Safe build sequence

```text
1. measured committed serial baseline
2. dependency-safe parallelism after complete calls are known
3. fixed-program replay with deterministic, local, discardable mock tools
4. parser-only candidate detection with no dispatch
5. isolated dispatch for deterministic read-like tools under tiny budgets
6. exact claim, miss, multiplicity, eviction, and cancellation accounting
7. stochastic or submodel calls with occurrence-preserving identity
8. load-aware canary with automatic disablement and rollback
```

Keep each stage feature-flagged. Do not enable the next stage until quality, authority, cost, and throughput gates pass against speculation-off.

## Evaluation requirements

Use the speculative execution cases, baselines, and metrics in [evals](evals.md). Include both fixed authoritative programs, which isolate scheduler correctness, and open-ended agent tasks, which reveal trajectory and quality effects.

The launch gate must show that speculation preserves committed task behavior, never executes an ineligible effect, improves the target latency percentile without harming throughput, and stays inside configured waste and cost limits. A latency-only result is insufficient.

## Anti-patterns

- Treating partial tokens as an authorized tool call.
- Calling every read-only operation pure, private, free, or safe to discard.
- Speculating writes, sends, payments, permission changes, destructive actions, or approval-gated calls.
- Claiming by mutable tool name and arguments without version, scope, snapshot, and occurrence identity.
- Collapsing repeated stochastic calls into one cached result.
- Giving the shadow runtime ambient credentials, network, filesystem, bindings, or authoritative objects.
- Calling an in-process interpreter with an allowlist a security sandbox.
- Treating logical eviction as confirmed physical cancellation.
- Letting speculative work consume capacity needed by committed work.
- Retrying a failed candidate without counting the physical attempt or checking replay safety.
- Restoring futures after restart without reconciliation and policy revalidation.
- Reporting speedup without task parity, cost, waste, saturation, and ordinary-parallel baselines.
- Enabling speculation before a reliable sequential tool loop exists.

## Design rule

Predict work early; commit authority late. Only an exact, authorized call in the completed program may claim speculative work.
