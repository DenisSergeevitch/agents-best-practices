# Self-Refining Recursive Harnesses

## Status and scope

A self-refining recursive harness is an advanced, post-MVP architecture profile. It combines a persistent program environment, recursive full-agent sessions, versioned supplemental harness state, and a resident runtime that can resume work after the initiating turn.

This profile is not a new autonomy or maturity level. A read-only research agent and a policy-bounded actor can both use it. Authority still comes from the permission model, not from persistence, recursion, or self-refinement.

Use this profile only after a simpler single-agent loop has measured limits that programmable context, recursive delegation, or online adaptation can address. The base harness still owns model calls, authorization, approvals, budgets, traces, and external side effects as described in [architecture.md](architecture.md).

This reference owns only the composition-specific contracts. Reuse the existing guidance for:

| Concern | Source of truth |
|---|---|
| goals, checkpoints, and done conditions | [planning-and-goals.md](planning-and-goals.md) |
| prompt context, memory, compaction, and rehydration | [context-memory-compaction.md](context-memory-compaction.md) |
| planned packets, worker verification, and integration | [workflow-orchestration.md](workflow-orchestration.md) |
| tool schemas, permissions, approvals, and sandboxes | [tools-and-permissions.md](tools-and-permissions.md) |
| late-bound capability discovery, verification, binding, and drift | [environment-adaptive-tools.md](environment-adaptive-tools.md) |
| speculative prelaunch and exact claiming of programmatic calls | [speculative-tool-execution.md](speculative-tool-execution.md) |
| installed skills and connector governance | [skills-and-connectors.md](skills-and-connectors.md) |
| human-directed harness improvement | [agent-legibility-feedback-loops.md](agent-legibility-feedback-loops.md) |
| threat modeling, traces, and incident response | [security-observability.md](security-observability.md) |
| general eval design, ablations, and launch gates | [evals.md](evals.md) |

## Contents

- [Taxonomy](#taxonomy)
- [Reference architecture](#reference-architecture)
- [Program environment contract](#program-environment-contract)
- [Four continuity layers](#four-continuity-layers)
- [Recursive child-session protocol](#recursive-child-session-protocol)
- [Supplemental harness ledger](#supplemental-harness-ledger)
- [Two-loop operating model](#two-loop-operating-model)
- [Immutable authority boundary](#immutable-authority-boundary)
- [Resident lifecycle and scheduled wakeups](#resident-lifecycle-and-scheduled-wakeups)
- [Safe build sequence](#safe-build-sequence)
- [Evaluation requirements](#evaluation-requirements)
- [Anti-patterns](#anti-patterns)

## Taxonomy

Keep four independent design choices separate.

### Strict recursive language model

A strict recursive language model, or strict RLM, keeps a large input outside the root model context. The model receives a reference to that input, examines and transforms it through a program environment, and can invoke bounded recursive model calls over selected portions.

The defining property is not merely code execution or subagents. It is that the external input is the primary working context and the root model programmatically chooses what enters each model call.

### Hybrid programmable-context agent

A hybrid programmable-context agent receives an ordinary conversational prompt and also has a persistent program environment. It can assign named variables, retain parsed data and intermediate results, inspect external resources, and move selected material into later model calls.

This is RLM-inspired, but it is not a strict RLM when the full task prompt is already inserted into the root context. Use the hybrid label rather than claiming strict RLM behavior.

### Recursive full-agent harness

A recursive full-agent harness lets a parent session create child sessions that have their own model-tool loops, scoped context, budgets, and durable identity. Creating a child returns an admission handle, not necessarily the child result. Results arrive later through typed messages or durable artifacts.

This differs from a nested model call, which returns one model response, and from a planned workflow packet, whose lifecycle and integration are defined in a versioned workflow artifact. Recursive children are useful for dynamic exploration; [workflow orchestration](workflow-orchestration.md) remains the better fit for predictable decomposition, independent verification, and controlled fan-in.

### Self-refining harness

A self-refining harness runs an outer adaptation loop that proposes and evaluates changes to versioned supplemental harness state while the ordinary action loop continues serving tasks.

“Self-refining” means changing harness configuration or reusable guidance. It does not imply model-weight training. If a system also updates model weights, treat that as a separate training pipeline with its own data governance, evaluations, and release controls.

The combined profile can therefore be described without conflating its dimensions:

```text
context representation: strict external context | hybrid programmable context
recursive execution: none | nested model call | full child session
adaptation: frozen harness | propose-only | validated online refinement
lifecycle: request-scoped | resumable session | resident scheduled session
authority: read-only | approval-gated | policy-bounded actor
```

## Reference architecture

Keep the trusted host authoritative and the program environment model-directed.

```text
client
  -> supervisor
      -> resident session worker
          -> action loop
          -> context builder and compactor
          -> persistent program environment
              -> typed host bridge
                  -> tool and skill registry
                  -> recursive child registry
                  -> artifact and message store
                  -> goal and schedule controller
          -> supplemental harness ledger
          -> refinement controller
      -> durable event, state, and artifact stores
```

The persistent program environment may expose one flexible execution surface to the model, but it must not become the authority boundary. Sensitive operations cross a typed host bridge whose implementations enforce the same schema validation, permissions, approvals, budgets, and audit rules as ordinary tools.

The host must own:

- authoritative session, child, goal, schedule, budget, and approval state;
- tool and skill discovery, argument validation, and execution;
- identity, credentials, permission checks, and external commits;
- lifecycle transitions, durable storage, tracing, and recovery;
- acceptance, quarantine, promotion, and rollback of harness refinements.

The program environment may own:

- named variables, imports, and helper functions;
- temporary transformations and local control flow;
- references to external inputs, messages, artifacts, and child handles;
- model-proposed calls to typed host capabilities;
- disposable caches that can be recreated from durable state.

Treat all program-environment code and state as model-directed and potentially unsafe. Process separation can contain failures and simplify restarts, but it does not create a security sandbox. Use the boundaries in [tools-and-permissions.md](tools-and-permissions.md) for generated code and external effects.

## Program environment contract

A reconstructable program environment needs a small explicit contract:

```text
execute(code, timeout, output_limit) -> execution_result
inspect(name_or_handle, slice) -> bounded_value
host.call(capability, typed_arguments) -> typed_result
children.spawn(spec) -> admission_handle
children.status(handle) -> child_status
children.list(cursor, filters) -> child_page
children.cancel(handle, reason) -> cancellation_result
messages.send(recipient_handle, type, payload_refs, correlation_id) -> delivery_receipt
messages.receive(cursor, limit) -> message_page
artifacts.write(name, content_or_ref, metadata) -> artifact_reference
artifacts.read(reference, range) -> bounded_content
```

Requirements:

- state survives ordinary model turns within a session;
- every execution has wall-time, output, memory, and cancellation limits;
- large values remain behind handles and are inspected in bounded slices;
- host calls return structured errors rather than leaking host exceptions;
- capability discovery returns only the tools and skills visible in the current scope;
- child, message, and artifact handles are opaque and tenant-scoped;
- the environment cannot read host secrets, forge approvals, or edit audit state;
- restoring a program snapshot never bypasses current policy or capability checks.

If those host capabilities are not fully known before the run, their discovery and rebinding must follow the [environment-adaptive tool lifecycle](environment-adaptive-tools.md); the program environment does not own the catalogue or binding authority.

For strict RLM behavior, bind the external input to a stable handle or variable without placing its full content in the root prompt. For hybrid behavior, state clearly which prompt material is already in model context and which values are available only through the program environment.

## Four continuity layers

Persistence is not one thing. Define four layers and their failure semantics separately.

| Layer | Purpose | Durability | Failure behavior |
|---|---|---|---|
| conversation and event history | authoritative record of requests, actions, observations, and compaction boundaries | durable | rebuild the next model context from events and the latest valid compaction handoff |
| live program state | variables, imports, helpers, open handles, and in-flight local control state | process-local | loss is expected on kernel or worker failure; never make it the only copy of required task state |
| program snapshot | best-effort serialization of selected recoverable variables and handle metadata | checkpointed | skip unsupported values, record omissions, and restore only into a fresh environment under current policy |
| durable resources and registries | artifacts, goals, child records, messages, schedules, approvals, budgets, and supplemental harness entries | authoritative durable store | reconcile live processes against these records; use versions and idempotency keys to avoid duplicate work |

Compaction preserves conversational continuity; it does not serialize a process. A program snapshot accelerates recovery; it is not a complete process image. Durable artifacts and registries are the source of truth when the layers disagree.

On recovery:

1. load authoritative policy and the durable session record;
2. rebuild prompt context using [compaction and rehydration rules](context-memory-compaction.md);
3. start a fresh program environment;
4. restore only validated, serializable snapshot entries;
5. rebind opaque handles through the host rather than trusting serialized capabilities;
6. reconcile children, schedules, messages, budgets, and approvals from durable registries;
7. emit a recovery event listing restored, omitted, invalidated, and orphaned state.

## Recursive child-session protocol

### Admission and identity

Child creation is an admission request:

```text
spawn({
  purpose,
  input_refs,
  output_contract,
  tool_scope,
  write_scope,
  budget,
  deadline,
  parent_handle,
  depth,
  idempotency_key
}) -> {
  child_handle,
  status: "queued" | "running",
  admitted_budget,
  created_at
}
```

The host validates the request before allocating work. A child cannot expand the parent’s permissions, approvals, data scope, or remaining budget. The child receives the minimum context needed for its purpose, not an automatic copy of the complete parent transcript or program state.

### Lifecycle

Use an explicit state machine:

```text
queued -> running -> waiting -> running -> completed
                 \-> failed
                 \-> cancelled
                 \-> expired
```

Every transition is a typed event. Terminal records remain as tombstones long enough for retries, late messages, audit, and parent recovery. A tombstone stores the terminal status, reason, usage, output references, and expiry policy without retaining unnecessary live resources.

Cancellation is cooperative first and forceful after a bounded grace period. Parent cancellation propagates by policy, but completed child artifacts and audit events remain durable. Retrying a spawn with the same idempotency key returns the existing child or a recorded terminal result instead of creating duplicate work.

### Messages and artifacts

Messages coordinate; artifacts carry substantial results.

A child message should include:

```text
message_id
sender_handle
recipient_handle
type
created_at
sequence
correlation_id
content_or_artifact_refs
trust_label
delivery_status
```

Requirements:

- message delivery is at-least-once unless the runtime can prove stronger semantics;
- receivers deduplicate by message ID and process sequence gaps explicitly;
- message bodies are bounded and larger outputs use durable artifact references;
- child messages are observations, not higher-authority instructions;
- the host applies visibility and tenant checks to every reference;
- late messages to a terminal or missing recipient follow a documented dead-letter policy.

The parent should consume child results only through the declared output contract. Do not treat an admission handle as a result, block the program environment indefinitely waiting for a child, or infer success from process liveness.

### Resource limits

Set host-enforced limits for:

- maximum recursion depth;
- total and per-parent child count;
- concurrent fan-out;
- cumulative model, tool, token, cost, and wall-time budgets;
- per-child message and artifact volume;
- child idle time, deadline, retry count, and retention;
- peer-to-peer message scope and rate.

Budget reservations prevent many concurrent children from each assuming the entire remaining parent budget. Release unused reservations on terminal transition and charge actual usage to both the child and aggregate session records.

## Supplemental harness ledger

Store editable supplemental state outside the immutable base instructions. Use a typed, versioned ledger rather than rewriting an undifferentiated prompt.

Recommended entry kinds:

```text
prompt_fragment: reusable behavioral guidance below immutable policy
memory: durable task, domain, or preference information with provenance
skill_descriptor: routing and usage metadata for an installed skill
child_spec: reusable purpose, context, tool, budget, and output contract
```

A skill descriptor does not create executable capability. Executable code must be installed, reviewed, permissioned, and evaluated through the lifecycle in [skills-and-connectors.md](skills-and-connectors.md).

Minimal ledger entry:

```yaml
id: "..."
kind: "prompt_fragment | memory | skill_descriptor | child_spec"
scope: "session | workspace | organization"
version: 4
status: "proposed | active | quarantined | rejected | rolled_back | superseded"
content: {}
provenance:
  source_event_refs: []
  proposer: "..."
  created_at: "..."
evidence:
  problem_refs: []
  validation_refs: []
expected_parent_version: 3
supersedes: "..."
expires_at: null
```

Session scope should be the default. Promotion to broader scope is a separate operation with stronger evidence, conflict analysis, evaluation, and approval. Never silently turn a local tactic into organization-wide policy.

Keep history append-only. Updating an entry creates a new version and supersedes the old one; rollback activates a recorded inverse version. Compact model-visible summaries may be derived from the ledger, but the summary is not the source of truth.

## Two-loop operating model

The action loop performs the current task. The refinement loop proposes and validates reusable harness changes.

```text
action loop
  -> model step
  -> typed host action
  -> observation
  -> task progress or completion
  -> trace evidence

refinement loop
  -> bounded evidence trigger
  -> typed candidate diff
  -> policy and conflict validation
  -> snapshot and safe-boundary apply
  -> observed evaluation
  -> accept, quarantine, or rollback
```

Do not let the refinement loop run as an unbounded second agent. Give it independent frequency, token, time, cost, change-count, and state-size budgets. Serialize refinement transactions per scope.

### Trigger policy

Useful triggers include:

- the same classified failure occurs more than once;
- a human correction reveals a reusable rule;
- a tactic succeeds repeatedly across eligible tasks;
- compaction or session completion creates a safe review boundary;
- a scheduled maintenance pass detects stale or conflicting entries.

Do not refine from one surprising result, unverified retrieved instructions, a single model self-critique, or the absence of explicit negative feedback. The trigger records why the evidence is sufficient and which events are eligible.

### Refinement transaction

Use this transaction:

1. **Select evidence.** Build a bounded, typed evidence window from event and artifact references. Label trust and exclude secrets or unrelated tenant data.
2. **Propose a diff.** Return typed create, update, delete, or no-change operations against exact ledger IDs and expected versions. Prefer the smallest change that explains the evidence.
3. **Validate authority.** Reject changes outside editable kinds or scopes and changes that would affect permissions, approvals, budgets, evaluation policy, or audit history.
4. **Validate content.** Apply schema, size, reference, provenance, injection, secret, contradiction, dependency, and duplicate checks.
5. **Detect conflicts.** Compare each expected version with the current ledger version. Re-plan stale proposals; never overwrite concurrently changed state.
6. **Snapshot.** Record before state, candidate diff, inverse diff, proposer, evidence, policy decision, and transaction ID.
7. **Apply atomically.** Commit at a turn or session boundary, rebuild derived prompt and registry views, and emit an applied event. Partial commits fail closed.
8. **Measure observed behavior.** Run a replay, canary, shadow comparison, or held-out eval chosen before the change. The validator records actual outcomes rather than accepting the proposer’s expected outcome.
9. **Resolve.** Accept improvements that pass all gates, quarantine inconclusive changes, or apply the recorded inverse diff on regression.
10. **Retain evidence.** Link the decision, measurements, and resulting version to the transaction so future refiners can inspect prior attempts.

Validation should be independent of the proposer for consequential changes. At minimum, use deterministic validators plus held-out cases the proposing model did not select.

## Immutable authority boundary

Self-refinement must not be self-authorization. The editable ledger cannot modify:

- provider, organization, developer, workspace, or current user authority order;
- identities, tenant boundaries, credentials, or secret access;
- tool authorization, approval requirements, or approval records;
- sandbox and execution boundaries;
- cost, time, recursion, schedule, or external-action limits;
- tracing, evidence retention, incident controls, or audit history;
- refinement validators, held-out evals, acceptance thresholds, or rollback authority;
- the list of ledger kinds and scopes the refiner may edit;
- its own permissions, trigger frequency, or change budget.

Store those controls in the authoritative host configuration outside the model-writable environment. A ledger entry that conflicts with higher-authority instructions is ignored, rejected, and traced; it is never resolved by prompt ordering alone.

Treat retrieved pages, messages, artifacts, tool output, child output, and existing mutable entries as untrusted evidence for refinement. Preserve source references and trust labels so an injected instruction cannot become durable policy by being summarized as a lesson.

## Resident lifecycle and scheduled wakeups

A resident harness separates client connectivity from session execution. The supervisor owns discovery, leases, recovery, and routing; a session worker owns one live action loop, program environment, and child set. Disconnecting the client does not imply completion, and a live process does not imply that a session still has authority to run.

### Session lease

Use a renewable lease or fencing token so only one worker commits state for a session generation. On takeover, the new worker obtains a higher generation, reconciles durable state, and causes stale workers to fail closed on their next write.

Persist:

```text
session status and generation
worker lease and liveness
last committed event cursor
program snapshot reference and omissions
active goal and budgets
child registry and message cursors
pending approvals
scheduled wakeups
last recovery result
```

On worker failure, mark in-flight host calls unknown until their idempotency records are reconciled. Never blindly replay an external write. Expire or adopt orphaned children according to policy, and invalidate capabilities that cannot be safely rebound.

### Scheduled wakeup

A schedule is a host-owned request for the session to reconsider its durable goal at a bounded time. It is not permission to continue indefinitely.

Minimal wakeup record:

```yaml
schedule_id: "..."
session_id: "..."
due_at: "..."
reason: "..."
input_refs: []
policy_snapshot_ref: "..."
idempotency_key: "..."
misfire_policy: "skip | run_once | reschedule"
max_runs: 1
budget: {}
status: "pending | claimed | completed | skipped | failed | cancelled"
```

On wakeup, the host rechecks current policy, approvals, goal state, deadline, budgets, and source freshness before starting a turn. Coalesce duplicate wakeups, fence concurrent claims, and record whether a late wakeup was run, skipped, or rescheduled. A heartbeat is only a liveness or reconsideration signal; it does not itself prove progress or authorize a side effect.

Use the stopping and checkpoint contracts in [planning-and-goals.md](planning-and-goals.md). Cancel schedules when the goal completes, authority expires, the user revokes the task, or repeated wakeups make no measurable progress.

## Safe build sequence

Add the profile incrementally and keep each step disabled until its focused evals pass:

```text
1. reliable request-scoped single-agent loop
2. persistent program environment with no privileged direct access
3. typed host bridge with existing permission and budget enforcement
4. bounded external-input handles and named program state
5. depth-one child admission, status, cancellation, and artifact results
6. durable child, message, and artifact registries with crash reconciliation
7. session-local supplemental ledger in propose-only mode
8. manual transaction apply and tested atomic rollback
9. automated local apply behind independent observed validation
10. resident detach, reattach, leases, and recovery
11. bounded scheduled wakeups with idempotency and misfire handling
12. broader-scope promotion behind explicit approval and held-out evals
```

Do not add recurrence, broad-scope refinement, retained children, or executable-skill mutation merely because the runtime can support them. Move to the next step only when the previous step solves a measured problem without weakening the base harness.

## Evaluation requirements

Use the profile-specific comparisons and metrics in [evals.md](evals.md); do not create a second evaluation framework here. Before enabling this profile, demonstrate independently observed held-out lift over a frozen harness, preserve the capability floor on simpler tasks, exercise crash and wakeup recovery, reject injected refinement evidence, and prove rollback for every editable kind. A change that improves average quality while creating severe tail regressions does not pass.

Use [security-observability.md](security-observability.md) for the trace fields, alerts, launch gates, and incident response that make those results auditable.

## Anti-patterns

- Calling any agent with a code tool an RLM.
- Calling ordinary synchronous model calls recursive full-agent sessions.
- Treating persistence, recursion, or schedules as a higher autonomy level.
- Giving the program environment direct access to credentials or privileged host objects.
- Assuming a worker process boundary is a sandbox.
- Treating a program snapshot as complete or authoritative state.
- Returning a child handle where the caller expects a completed result.
- Allowing children to inherit broad context, tools, or the parent’s full remaining budget by default.
- Deleting terminal child records before late messages, retries, and audits are reconciled.
- Letting mutable skill descriptions masquerade as installed executable capability.
- Rewriting a monolithic prompt instead of applying typed, versioned supplemental diffs.
- Learning durable rules from untrusted content or one unexplained outcome.
- Accepting the refiner’s predicted benefit as observed validation.
- Letting the refiner edit permissions, evals, acceptance gates, or its own authority.
- Applying concurrent refinement diffs without version checks and atomic rollback.
- Promoting session-local tactics to shared scope automatically.
- Replaying unknown external writes after recovery without idempotency evidence.
- Treating heartbeats as evidence of progress or scheduled wakeups as standing authorization.
- Adding the profile before a single-agent baseline and held-out eval set exist.

## Design rule

The value of this profile is continuity plus controlled adaptation, not unconstrained self-modification. Keep authoritative policy in the host, model-directed computation disposable, durable state typed and versioned, recursive work bounded, and every accepted refinement tied to observed evidence and a tested rollback path.
