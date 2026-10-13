# Always-on Agents and Durable Runtime

## Taxonomy and boundaries

An **always-on agent** is a service that remains available to accept inputs or scheduled events across client disconnections and individual turns. Its worker can sleep or restart between events; availability does not require continuous inference. A **durable runtime** preserves accepted work and committed state so compatible workers can recover it after interruption. These are separate properties: an available service can lose work, and a durable session can remain offline.

This is an advanced, post-MVP architecture profile built from command admission, journaled state, supervised tasks, and recovery. It introduces neither a new model/training method nor a new autonomy level. Always-on operation does not imply recursion, self-refinement, distributed ownership, or permission to act indefinitely.

The trusted host owns admission, storage, scheduling, executable definitions, cancellation, publication, and recovery. The model proposes actions and consumes bounded context; it cannot turn persisted text, a checkpoint, or a memo into authority. Reuse these canonical owners:

| Concern | Canonical owner |
|---|---|
| goals, budgets, deadlines, and stopping | [planning-and-goals.md](planning-and-goals.md) |
| ordinary loop and model routing | [agentic-loop.md](agentic-loop.md) |
| instruction/tool configuration replay | [architecture.md](architecture.md#runtime-instruction-and-tool-configuration-events) |
| external effects, permissions, approvals, and retry policy | [tools-and-permissions.md](tools-and-permissions.md) |
| prompt reduction and summary publication | [context-memory-compaction.md](context-memory-compaction.md) |
| recursive child admission and message delivery | [self-refining-recursive-harnesses.md](self-refining-recursive-harnesses.md#recursive-child-session-protocol) |
| planned work packets and integration | [workflow-orchestration.md](workflow-orchestration.md#state-and-resume-behavior) |
| traces, security, and incident controls | [security-observability.md](security-observability.md) |
| failure probes and release gates | [evals.md](evals.md#always-on-and-durable-runtime-evals) |

## Records and atomic publication

Use a journaled local unit of work for related transcript entries, task checkpoints, application documents, command receipts, and accounting updates. Prepare the batch, settle storage, adopt its committed state, then publish a new view. Serialize mutations for an owner; observers must not expose prepared or rejected state, block commit settlement, or recursively mutate the same commit line.

A reconstructable record model includes:

```text
Session: identity, status, owner generation, committed sequence, schema versions
Submission: request identity, stable intent digest, disposition, placement/run refs
Task: kind/version, owner, foreground/background boundary, phase/checkpoint,
      invocation generation, abort intent, wait refs, held outcome, terminal receipt
Document: scope/owner, incarnation, schema version, history/fork policy, value
Attempt: logical operation, physical attempt, effect identity, intent/outcome,
         implementation/policy provenance, usage and uncertainty
```

Define failure semantics before implementing the adapter. A guaranteed rejection means the batch had no effects and permits another transaction. An uncertain storage result, or failure adopting an acknowledged commit, seals further mutations until reopening and reconciliation establish the committed state. Caller cancellation after commit admission must not abandon storage settlement; a cancelled waiter does not undo the batch. Close seals new admission and lets admitted commits settle before releasing storage.

Keep model calls, tool execution, network requests, processes, and human interaction outside the local transaction. A committed intent followed by an external effect followed by a committed outcome leaves a crash window between each step. Local atomicity cannot close that window or roll back an external action; use the existing effect identity and reconciliation policy in [tools-and-permissions.md](tools-and-permissions.md).

State the storage failure model explicitly: memory-only, process-crash recovery, host/power-loss durability, or replicated availability. An append marker, write acknowledgement, batching interval, or conformance suite alone does not prove the stronger classes. Bound document sizes, loaded state, pending work, observer payloads, and journal growth separately; paged history does not bound arbitrary application memory.

## Accepted inputs and control lanes

Admission atomically records a submission and its queue placement or initial turn. Deduplicate in an explicit session/conversation namespace before deciding whether the worker is busy. Bind the identity to stable intent: reject changed payloads or return the original with an explicit conflict/disposition policy. Never silently reinterpret an old identity as new work.

Distinguish **accepted**, **queued**, **placed in the transcript**, **assigned to a run**, **answered**, and **withdrawn**. An admission receipt proves only its recorded stage. Retries after disconnect recover that receipt; they do not imply one physical model request or exactly-once external effects. Define receipt retention so a forgotten identity cannot accidentally admit the same command again.

Expose separate input lanes with host-defined ordering and applicability:

| Lane | Contract |
|---|---|
| Follow-up | Waits for the active turn's safe completion boundary before starting or joining subsequent work. |
| Steering | Enters at an explicit safe boundary within active work; acceptance cannot promise immediate preemption of an in-flight request. |
| Passive write | Appends evidence or state without implicitly starting inference. |
| Control | Cancellation, status, and shutdown remain usable while ordinary work is busy; authority comes from the host. |

Specify ordering across lanes, whether a boundary drains one or all items, and behavior after failed runs. Withdrawing a queued input differs from cancelling its run; neither can retract an already consumed observation or completed external effect. Persist dispositions so recovery cannot drop, duplicate, or silently revive queued work. Input withdrawal and passive-write retention need separate policies.

## Durable tasks and owned work

Implement long-lived work as typed phase machines rather than serialized promises or process stacks. Each invocation receives a durable checkpoint, can commit progress or park on explicit waits, and must produce a durable transition before yielding. Recovery resolves the task kind and checkpoint version against an installed executable registry; code, closures, and credentials are not recovered from transcript text.

Persist intent before an effect and its outcome afterward. Reentry at the intent phase may mean the effect already happened. Record replay eligibility at admission and check current eligibility again on recovery; default to interrupted/unknown unless replay safety is established. Resolve current bindings and authority through their existing owners. A recorded model request can preserve cutoff and settings while hooks, implementations, or provider behavior change; recovery is compatible reexecution, not deterministic replay.

Checkpoint migrations must be versioned, pure, and atomic with reservation of the new invocation. Missing definitions, unsupported versions, and failed migration leave work explicitly blocked and live. Do not mark it successful or discard it. Define whether an authorized abort can settle orphaned work without cleanup, and retain unresolved effects for reconciliation. Reload affects new invocations; a running handler keeps its implementation until it settles. Noncooperative code requires worker/process isolation for forced termination.

Separate ownership from waiting. Owning a task determines lifetime and cancellation propagation; waiting on it determines when an invocation may resume. A failed child does not decide its parent's outcome unless that policy is explicit. Fail-fast waits mark eligible owned siblings, then await their terminal receipts before resuming integration.

Use an owned-work completion barrier:

```text
pending -> running -> waiting -> runnable
running -> completing(held outcome) -> terminal receipt
abort requested -> descendants settled -> owner cleanup -> terminal receipt
missing/incompatible definition -> blocked -> repaired or explicit orphan disposition
```

The held outcome is final, but ordinary owned work can still be live. A published result entry, stopped generation, idle queue, or cancellation acknowledgement therefore cannot claim the task tree has settled. Commit the terminal receipt and resolve completion waiters only after the defined foreground subtree has drained. Keep task-owned documents alive until that terminal boundary.

Mark background ownership boundaries explicitly. Background work can outlive an ordinary turn and be excluded from its idle/completion barrier; it still has a durable owner, budgets, status, and an explicit full-stop policy. Distinguish cancelling a wait, aborting a task, aborting foreground conversation work, stopping background work, and closing the runtime. Close suspends recoverable work; it is not cancellation, compensation, or rollback.

Commit abort intent before signalling invocations. Fence subsequent runtime mutations by invocation identity/generation and abort state, then settle descendants before owner cleanup. State whether full cancellation covers only work present at admission or also seals future child admission. Cooperative signals cannot prevent ignored signals from completing external actions; terminal ordering follows committed receipts, and uncertain effects remain visible.

Hooks may rerun after recovery. A first-write-wins memo can retain a committed decision, but cannot atomically couple an external approval or payment to the memo. Store scope/version/provenance and recheck current authority at execution using [approval records](security-observability.md#approval-records); a persisted boolean is insufficient.

## Application documents and forks

Keep typed application state distinct from conversational memory and model-visible configuration. Define four independent document contracts: scope/lifetime, schema version, historical retention, and fork inheritance. A session-scoped document may stay shared; conversation-scoped copies become independent child incarnations; task-scoped documents retire with terminal work and should not manufacture copied live tasks.

| Fork policy | Meaning |
|---|---|
| Historical | Copy committed state at the selected ancestor's documented transaction boundary; requires retained history. |
| Current | Copy the parent's committed value at fork admission, independently of the selected transcript ancestor. |
| Fresh | Initialize a new child incarnation rather than copying parent state. |

Name the historical boundary precisely. If several transcript entries share a commit, forking at an entry may inherit the final document state of that whole commit rather than the state after that entry alone. Expose this granularity; do not imply finer history than storage retains. Fork reads and copy publication use one consistent source view, with source-changing mutations in that batch rejected or assigned an explicit ordering.

Copy stored values and versions, not incidental migrated read caches. Older versions may migrate on typed access without rewriting history; newer or unmigratable versions fail visibly. Read-only migration and committed write migration are distinct. Preserve unaccessed or unavailable definitions as opaque stored data rather than silently deleting them. A recreated document has a new incarnation; old handles and subscriptions cannot acquire its new lifetime accidentally.

Authority/configuration inheritance still follows [configuration events](architecture.md#runtime-instruction-and-tool-configuration-events). Copying a document is not approval to copy credentials, extend access, inherit later sibling configuration, or restart a parent's tasks.

## Observation and reconnection

Acquire the initial snapshot and live attachment atomically against publication so no transition can fall between them. Deliver only committed views, with serialized callbacks off the mutation line. Choose and document one of two contracts: convergence to current state, or replay of retained transitions. A convergent watch can coalesce missed intermediate states and reconnect from a root snapshot; it is not a lossless audit log.

Keep commit sequence, backend paging cursor, live-delivery cursor, and durable replay cursor separate. A reconnect token must bind its namespace, incarnation, retention range, and gap behavior if replay is promised. An in-memory attachment position cannot establish durable resume. Fall back to a fresh snapshot when retention or overflow creates a gap.

Bound queue count **and bytes**, preserving the currently executing callback while replacing or rejecting a pending suffix according to policy. Document whether stop/close only prevent future deliveries or also cancel/join active callbacks. Document retirement emits a terminal absence for that incarnation and does not follow recreation. Host observers consume trusted immutable/detached revisions; read-only types alone do not enforce this against hostile shared-object mutation.

Separate live structural views from append-only audit/effect evidence in [security-observability.md](security-observability.md). Neither observer delivery nor a user-visible final response proves that owned work or side effects have settled.

## Resident ownership and scheduled wakeups

A resident runtime separates client connectivity from session execution. The supervisor owns discovery, leases, recovery, and routing; a worker owns a session's admitted tasks and current loop. A live process does not establish authority to continue.

### Session lease

Use a renewable lease or fencing token when workers can take over storage ownership. A higher owner generation fences stale writers before another worker resumes. A package that assumes one process and lacks cross-process locking requires an external ownership boundary; do not infer distributed safety from local serialization.

Persist session status/generation, lease/liveness, committed cursor, state/checkpoint references, active goal/budgets, owned-work registry, pending approvals, wakeups, and recovery result. Reconcile unknown host calls before another external attempt. Expire or adopt orphaned children under the [recursive child protocol](self-refining-recursive-harnesses.md#recursive-child-session-protocol), and invalidate capabilities that cannot be rebound safely.

### Scheduled wakeup

A schedule is a host-owned request to reconsider a durable goal at a bounded time. It is not permission to continue indefinitely. Minimal wakeup record:

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

Recheck current policy, approvals, goal state, deadline, budgets, and source freshness before each turn. Coalesce duplicate wakeups, fence concurrent claims, and record late-run disposition. A heartbeat establishes a liveness or reconsideration signal, not progress or authorization. Cancel schedules on completion, expired/revoked authority, or repeated no-progress outcomes under [goal stopping rules](planning-and-goals.md).

## Recurring collection and delivery bookkeeping

For explicitly requested recurring monitoring, distinguish **collection progress**, **source coverage**, and **delivery progress**. These are application records, separate from the [observation/replay cursors](#observation-and-reconnection). The host owns their transitions; model-written notes and a successful tool call alone cannot establish complete coverage or confirmed delivery. Reuse [local commit and external-effect boundaries](#records-and-atomic-publication), [record provenance](tools-and-permissions.md#record-provenance-and-authoritative-fields), and the existing [retry policy](agentic-loop.md#retry-policy).

Minimal records:

```text
SourceScan: source/partition, query revision, prior checkpoint, scan cutoff,
            proven boundary, coverage (complete | partial | failed), evidence/reason
Edition: stable identity, destination, source-scan refs, selected item versions,
         status (prepared | posting | confirmed | suppressed | held | unknown), receipt
DeliveryLedger: source/item/version, edition identity, confirmed receipt
```

### Collection checkpoints and coverage

Resume each source partition from its last committed collection checkpoint rather than a fixed lookback relative to the schedule. Define first-run initialization and how changed queries or newly added partitions establish their own checkpoints. A missed run extends the collection interval; it must not silently discard the intervening period.

Specify the source's ordering, pagination, scan cutoff, timestamp ties, and late-arrival/update behavior. Use a source-supported continuation boundary or an explicit overlap/rescan policy with stable item/version deduplication. The newest observed timestamp is not proof that earlier pages or later updates were covered. Exhaust the declared interval before marking it complete; rate limits, expired history, truncation, and unreadable pages must expose a gap rather than manufacture completeness.

Failed or partial scans preserve that partition's last complete checkpoint. Other partitions may progress independently. Persist coverage for every configured partition, including successful empty reads; surface omitted or incomplete sources in the report or its associated run status. Do not infer "nothing changed" from missing tools, authentication failure, or an incomplete read. Use the existing [tool results and errors](tools-and-permissions.md#tool-result-format) for the underlying observations.

### Edition outcomes and progress commits

Bind an edition identity to its destination and stable content intent; retain selected item versions and scan references. Record the publication attempt before dispatch. Mark an edition confirmed and add delivered-item ledger entries only after a destination-specific success receipt with a resolvable message/effect identity. HTTP success, a generated draft, or a recent-title lookup alone cannot establish delivery or exclude concurrent attempts. Apply the existing ownership/fencing and external-effect reconciliation contracts rather than introducing a second retry mechanism.

Commit collection checkpoints, candidate dispositions, and edition outcomes as a related local batch. A complete scan may advance after confirmed delivery or explicitly permitted suppression. If collection advances before delivery, durably retain its pending candidates in that same batch so a failed send cannot lose them. Failed/unknown sends never create delivered-item entries; without retained candidates they also hold the collection checkpoint. Reconcile an unknown publication before attempting another send.

A deliberately quiet edition is **suppressed**, with its policy reason and coverage evidence; it has no delivery receipt and cannot mark unsent items delivered. Suppression cannot relabel an unreadable source as a healthy empty read. Keep edition delivery and source coverage separate from run health: a budget pause, missing run, held publication, and unknown effect are distinct operational outcomes under [observability and incident controls](security-observability.md). Validate these contracts with the [recurring collection probes](evals.md#always-on-and-durable-runtime-evals).

## Recovery and build sequence

Start from the measured request-scoped single-agent baseline. Add one durable session with one storage owner, then atomic submission receipts and local publication. Add typed task phases and effect reconciliation before background tasks; add owned-work barriers before accepting completion claims. Introduce document forks and live views only for demonstrated needs. Finally add supervised takeover and bounded schedules, keeping recursion and online refinement optional.

Recovery opens and verifies storage, resolves compatible executable definitions, reconstructs committed records, rechecks authority and budgets, reconciles uncertain attempts, then admits runnable work under the current owner. Recover accepted inputs and held terminal outcomes before serving new work. Rebuild process-local observers and provider sessions explicitly; use [concurrent compaction publication](context-memory-compaction.md#concurrent-compaction-publication) for summaries rather than overwriting a live transcript.

Validate the failure boundaries in [durable-runtime evals](evals.md#always-on-and-durable-runtime-evals) before enabling unattended use. Report demonstrated failure classes, recovery time, unresolved effects, data loss exposure, task quality, and full completion cost. Retain an operator path to suspend admission, inspect blocked work, revoke authority, and reconcile or explicitly abandon unresolved effects.

Avoid claiming exactly-once actions from input deduplication, completion from idle inference, durable resume from an observer cursor, fresh authority from persisted approvals, unlimited memory from paged history, or high availability from a single-owner store. These are separate contracts with separate evidence.
