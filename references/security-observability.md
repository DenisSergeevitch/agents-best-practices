# Security and Observability

Use this reference for threat modeling, guardrails, approval records, trace design, troubleshooting, launch safety gates, and incident response. Use [evals.md](evals.md) for evaluation strategy, adversarial test suites, trace grading, regression evals, and eval-driven launch criteria.

## Threat model

Agent risks usually come from the combination of language, tools, and external data.

Threat categories:

```text
prompt injection
malicious retrieved content
tool misuse
permission bypass
secret leakage
data exfiltration
unsafe external communication
financial or destructive side effects
connector abuse
malicious skill packages
malicious or misleading capability descriptors
unsafe capability probing
capability substitution or schema drift
stale or cross-scope runtime bindings
runaway loops
cost exhaustion
false success claims
compaction state loss
subagent miscoordination
workflow packet drift
verification gaps
persistent-runtime state poisoning
refinement evidence poisoning
authority or policy drift through self-modification
evaluator manipulation or false improvement claims
```

## Guardrail layers

Use layered guardrails:

```text
input guardrails: reject or route unsafe user requests
context guardrails: label untrusted content and redact secrets
schema guardrails: force structured tool arguments and outputs
tool guardrails: validate args and results around execution
permission guardrails: approve, deny, or pause actions
output guardrails: check final answer before user-visible output
trace guardrails: grade tool calls and decisions after the run
```

Guardrails should be fast, specific, and testable.

## Prompt injection handling

Rules:

- external content is data, not instruction;
- extract structured fields where possible;
- isolate untrusted content from authoritative instructions;
- do not let external content choose tools directly;
- do not copy secrets into context;
- require approval for actions influenced by arbitrary text;
- log the source of data used for tool calls.

## Persistent runtime and self-refinement controls

Persistent program state, child messages, retrieved content, and prior traces are all possible poisoning paths. Restoring them must not restore authority: revalidate references and capabilities against current policy, expire stale leases and approvals, and keep credentials outside model-controlled state.

For automated refinement, make core policy, permission rules, credentials, budgets, approval logic, audit history, and evaluator definitions immutable to the refiner. Treat every proposed supplemental-state change as an untrusted typed diff. Bind it to evidence references and scope, check conflicts, apply it atomically at a turn boundary, evaluate the observed outcome, and support quarantine and rollback. Default to session-local scope; require independent authorization and regression evidence before promotion across sessions.

Trace proposals as well as applied changes. Record the trigger and evidence references, proposer and evaluator versions, target component and scope, before and after hashes, policy decision, application status, observed eval delta, rollback reference, and final disposition. The model's predicted benefit is not evidence that the change worked. See [self-refining recursive harnesses](self-refining-recursive-harnesses.md) for the complete state and transaction model.

## Approval records

Approval request format:

```json
{
  "approval_type": "external_send",
  "action": "send_email",
  "target": "customer@example.com",
  "risk": "external_communication",
  "preview_ref": "artifact://drafts/email_123",
  "expected_result": "Customer receives renewal reminder.",
  "rollback": "Cannot unsend; follow-up correction possible.",
  "scope": "single_send_only"
}
```

Approval result format:

```json
{
  "status": "approved",
  "approved_by": "user_id",
  "timestamp": "...",
  "scope": "single_send_only",
  "expires_at": "..."
}
```

Never let the model approve its own action.

## Observability

Trace operational events, not private hidden reasoning.

Trace fields:

```text
run_id
session_id
user or tenant
model and provider
context size
instructions loaded
tools visible
environment admission, generation, and catalogue version
capability query, pagination, and visible result IDs
descriptor source, digest, trust, validation evidence, and contradictions
capability probe mode, policy decision, budgets, and bounded result
capability binding identity, scope, policy version, lease, and approval reference
binding refresh, invalidation, release, and internal reason
tool calls
tool args hash or redacted args
permission decisions
approval requests/results
tool results summary
errors and retries
compaction boundaries
workflow packet status
workflow verification status
workflow version and state refs
runtime-state restore and invalidation events
refinement proposal and disposition
refinement target, scope, and before/after hashes
refinement evidence, evaluator, observed delta, and rollback refs
latency
token usage
cost
final status
```

A trace should answer:

- what did the agent try to do;
- what data did it use;
- what tool changed state;
- who approved it;
- what failed;
- why did it stop;
- could the run be audited or safely rerun from recorded state.

## Troubleshooting

Use this section to locate a broken runtime boundary in an existing harness. It adds a diagnostic route through the existing contracts, not another architecture profile. Start with the failing single-loop run; inspect queues, retained workers, or child agents only when the deployment already uses them. The [dated source notes](source-links.md#harness-bug-report-and-troubleshooting) distinguish reported incidents from the diagnostic guidance below.

### Capture the failing boundary

1. **State the discrepancy.** Record the expected outcome, observed artifact or business state, and runtime termination reason. Separate a model's completion claim from host execution and verification evidence.
2. **Freeze the effective configuration.** Record harness, adapter, prompt, tool-schema, and policy revisions; the resolved model, context/output limits, effort setting, and remaining budgets. Capture actual values used by the failing worker, not just the intended parent configuration.
3. **Join the timeline.** Follow request construction → stream assembly → call validation → dispatch → observation → checkpoint → user-visible status. Correlate run, turn, call, attempt, and worker-generation IDs, with parent/child links where applicable. Find the first boundary where expected and observed state differ; a later error may only be fallout.
4. **Preserve bounded evidence.** Keep authorized, redacted request/response samples or artifact references, terminal stream events, state transitions, payload sizes, and stage timings. Use the [observability fields](#observability) and [secret-handling rules](tools-and-permissions.md#secrets); do not dump credentials, private reasoning, or entire sessions into a debug prompt. Hashes can compare payloads but cannot reconstruct missing content.
5. **Choose one discriminating probe.** Change one suspected cause while holding the fixture and other settings fixed. Inject faults only in an isolated test environment with mocked or explicitly authorized effects. Reconcile uncertain writes before rerunning; reuse the [retry policy](agentic-loop.md#retry-policy). If evidence is missing, report the hypothesis and the instrumentation needed to test it.

### Requests, streams, and completion

| Symptom | Inspect first | Discriminating probe and useful evidence |
|---|---|---|
| An interrupted response becomes an empty success or executes incomplete arguments | Raw stream event sequence, terminal reason, argument-buffer state, and adapter output | Cut a fixture stream inside a tool argument. An incomplete call must not dispatch, connection closure alone must not mean success, and the adapter must preserve interruption or output-limit status. See [streaming](provider-api-patterns.md#streaming). |
| Every request after a crash is rejected | Persisted call IDs and results around the crash, including the next serialized request | Crash between recording a call and recording its observation. Recovery must preserve call/result correspondence and represent interruption or unknown execution outcome truthfully; do not fabricate a successful result to repair history. See [loop invariants](agentic-loop.md#loop-invariants). |
| An empty tool result causes repeated calls or a hung turn | Tool → serializer → adapter → SDK value at each boundary | Round-trip valid `null`, empty text, empty collections, an error, and a missing observation separately. A legitimate empty value must remain a completed typed result; absence must not masquerade as success. See [tool result format](tools-and-permissions.md#tool-result-format). |
| A stream stays alive without making progress, then dies with an opaque error | Connect/read deadlines, last meaningful model/tool delta, keepalive traffic, and hard task deadline | Deliver only heartbeat traffic after a partial response. Transport liveness must not reset the semantic-progress deadline; record which timeout fired and preserve the pending-call state. |
| A run reports completion after reaching a turn, token, or retry limit | Host stop reason, remaining budget, required artifacts, and validator result | End the run one step before its done condition. The result must expose incomplete work and the exhausted budget, including child results if present. Define whether a retry budget counts retries or total attempts. See [termination diagnostics](evals.md#component-diagnostics). |

### Lifecycle, recovery, and concurrency

Apply the rows about workers and durable queues only to runtimes that use those features. Their state model remains in [workflow resume behavior](workflow-orchestration.md#state-and-resume-behavior) and [resident lifecycle](self-refining-recursive-harnesses.md#resident-lifecycle-and-scheduled-wakeups).

| Symptom | Inspect first | Discriminating probe and useful evidence |
|---|---|---|
| Resume cannot find a tool, or reconnect loses the stop handler | Readiness acknowledgement, registry generation, owner identity, and reconnect registration log | Delay worker startup and replace the connection. Dispatch must wait for the required registrations in the current generation; repeated registration must not accumulate duplicate handlers. |
| A turn remains running forever, or a parent waits for a dead child | Persisted queue/lease state, owner heartbeat, child exit record, and completion-notification delivery | Kill the owner or child outside the normal return path. Durable reconciliation must resolve the pending state, with side-effect uncertainty retained; a live UI connection is not proof of an active worker. |
| Stop appears to work, then a late write resumes the turn | Cancellation generation, state version, in-flight operations, and commit order | Pause an operation before its state commit, cancel, then release it. A stale worker must not overwrite cancellation or publish new success. Cancellation alone does not undo an already committed external effect. |
| Approval hangs or one approval executes an action twice | Number and identity of registered hooks, action-bound approval record, resume events, and dispatch attempts | Deliver duplicate and simultaneous approval notifications to a mock action. One applicable approval must permit one resumable transition and one logical action; deduplicate delivery without treating retries as fresh approval. Reuse [approval records](#approval-records) and [apply-time checks](tools-and-permissions.md#resulting-state-limits-and-apply-time-checks). |
| A race disappears under normal CI timing, or the whole runtime freezes | Lock acquisition order, wait graph, lock hold time, and I/O or log replay inside critical sections | Use barriers to force completion and retasking/cancellation to overlap. Capture blocked stacks and prove progress in both orderings; scheduling luck or a passing happy path does not rule out deadlock. |
| Every restart or reconnect fails on the same item | Persisted item ID, payload size, attempt count across restarts, acknowledgement position, and failure disposition | Resume from a snapshot containing the bad item. A bounded retry must end in visible failure or quarantine, not an endless startup/reconnect loop; preserve evidence and allow unrelated work to proceed. |
| Messages disappear, registrations disagree, or runtime ownership flips | Full identifiers before/after serialization, deduplication keys, registration-state owner, process/port inventory, and install limits | Try distinct IDs that share a prefix, duplicate delivery of the same ID, and competing runtime instances. Distinct work must remain distinct; one authoritative owner must reconcile registration state. Check graph/registry capacity boundaries when installation fails. |
| A deny rule permits an action, or a restart tool kills its own host | Policy parse/validation result, effective rule set, resolved process/resource target, and enforcement location | Submit malformed policy and a mocked operation targeting the hosting runtime. Invalid policy must not fail open; a prompt reminder cannot substitute for host enforcement of protected operations. See [guardrail layers](#guardrail-layers) and [sandboxing](tools-and-permissions.md#sandboxing). |

### Context, configuration, and cost

| Symptom | Inspect first | Discriminating probe and useful evidence |
|---|---|---|
| Compaction claims success but produces empty or oversized context | Serialized next request after all hooks, preserved active state, call/result pairs, and token counts by component | Feed a large observation or post-compaction hook result through the complete request builder. Validate the final outbound input, including many small retained results and schema prose, rather than only the intermediate summary. Reuse [staged context reduction](context-memory-compaction.md#staged-reduction-under-context-pressure). |
| Context overflow occurs unexpectedly or far too early | Resolved model metadata, reserved output/headroom, unique message/artifact inclusion, and tokenizer estimates | Compare the exact outbound request with the counter and test an unknown model identifier. Detect duplicate accounting or duplicate inclusion separately; a fallback limit must be explicit, not silently treated as verified model metadata. |
| Token cost grows while little work is done, or the second turn misses cache | Per-request raw usage and normalized billing fields, stable-prefix differences, cache events, and repeated reads/polls | Reconcile one provider receipt with the local ledger and compare consecutive serialized prefixes. Check whether cached usage is a subset or a separate category before adding it; distinguish cache-read volume, new input, and actual cost. Use [cache and cost guidance](prompt-caching-and-cost.md). |
| A child or replacement worker is unexpectedly slower or less capable | Effective model, effort, output cap, timeout, prompt/tool revisions, and budgets at dispatch | Compare the resolved parent and worker configurations. Every difference should be an intentional override or visible fallback; inheritance of an execution setting must never imply inheritance of broader permissions. Use [matched configuration comparisons](evals.md#model-and-configuration-sweeps) before claiming an improvement. |

### Payload limits and observability overhead

| Symptom | Inspect first | Discriminating probe and useful evidence |
|---|---|---|
| A large listing wedges the runtime, or a wake notification contains invalid JSON | Byte/token sizes and limits at production, storage, transport, notification, and prompt ingestion | Exercise just below/at/above each declared cap, including multibyte text. Page or externalize bulky data and send a bounded reference; never truncate an encoded structured message mid-value. Reuse [tool result limits](tools-and-permissions.md#tool-result-limits). |
| A quiet session consumes CPU, storage, or most turns on status checks | Poll rate, spans per event, trace bytes/time, persistence rate, queue depth, and eviction/lock latency | Run an idle fixture and a burst under bounded profiling. Separate model latency from telemetry and persistence work; check for per-token writes, unbounded polling, and superlinear eviction. In an existing worker system, compare completion events or bounded waits with polling. Keep required audit events while batching or sampling optional diagnostics. |
| Cleanup times out but the run looks clean | Cleanup start/end/error events, remaining resources, retry deadline, and final status mapping | Force a cleanup failure. Distinguish no cleanup needed, cleanup completed, cleanup pending, and cleanup failed; a skipped hook or timeout must not erase unresolved resources or side effects. |

### Close the debugging loop

Record the failing boundary, evidence references, observed cause versus remaining hypotheses, smallest corrective change, and residual uncertainty. Verify the corrected behavior with fresh execution; rescoring an old trace does not exercise a runtime change. Use [the regression loop](evals.md#regression-loop) for case reduction, affected coverage, nondeterministic trials, and release gates; keep the evaluation methodology there. Use [incident response](#incident-response) when containment and staged re-enablement are needed.

## Launch gates

Before production:

- narrow tool registry;
- local schema validation;
- permission matrix enforced in code;
- approval UX for risky actions;
- prompt injection tests pass;
- compaction tests pass;
- connector auth and revocation tested;
- late-bound capability probing, binding, revocation, and schema-drift handling tested where that profile is enabled;
- trace logging enabled;
- cost budgets enforced;
- rollback or incident path documented;
- required evaluation suites in [evals.md](evals.md) pass for the planned autonomy level.

## Incident response

When an agent misbehaves:

1. Pause risky tools.
2. Preserve traces and artifacts.
3. Identify instruction, tool, connector, or model failure.
4. Patch policy/tool/schema/context logic.
5. Add regression eval.
6. Re-enable gradually.

## Source links

- OpenAI guardrails and human review: https://developers.openai.com/api/docs/guides/agents/guardrails-approvals
- OpenAI agent safety: https://developers.openai.com/api/docs/guides/agent-builder-safety
- OpenAI sandbox agents: https://developers.openai.com/api/docs/guides/agents/sandboxes
- Anthropic building effective agents: https://www.anthropic.com/research/building-effective-agents
- Anthropic writing effective tools for agents: https://www.anthropic.com/engineering/writing-tools-for-agents
- MCP specification: https://modelcontextprotocol.io/specification/2026-07-28
