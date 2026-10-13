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
   Final output can look right while the run skipped approval, leaked state, retried wastefully, or ignored a failed observation. Preserve these invariants while accepting different valid routes to the required outcome.

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
expected_final_state
expected_rendered_output
quality_rubric
cost_latency_budget
grading_notes
```

Use stable fixtures and record model, provider, harness version, tool bundle, prompt/instruction version, and random seed or sampling settings when available.

`initial_state` includes the harness and backend state needed to reproduce the decision: messages, resource provenance, identity and permissions, approval records, pending changes, memory, and the latest rendered component state and ordering where applicable. A stateless model API does not make the harness stateless. Restore consistent fixtures together; a transcript alone must not recreate authority.

Include long, busy, or contradictory preconditions that expose the behavior under test. Inject earlier state directly unless carrying or updating it across turns is itself the test. Simulated conversations can discover failures; reduce them to controlled cases for attributable regression measurement.

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

Pair each required behavior with a nearby case where it should be absent: serve/refuse, act/ask, load/skip a skill, or save/reject a memory. Include requests spanning neighboring capabilities and assert both obligations in the same outcome; separate passing suites can miss an answer that handles only half the request.

For tools that render records, change business state, or maintain user memory, include these contract cases:

- **Rendered references:** resolving “the second one” must use the final display after invalid records are filtered or the client changes their order; check displayed IDs and layout, authoritative fields, and exact required disclosures.
- **Record access:** an invented or copied ID, a previously seen ID after permission revocation, and an ID read only by a delegate exercise separate provenance and authorization checks; no case may grant write access merely because a record was visible.
- **Resulting-state limits:** repeated calls and concurrent workers targeting the same resource cannot combine to exceed a limit; applying a staged action must recheck current policy and reject stale target or approval bindings.
- **Memory lifecycle:** test eligible extraction, rejected sources including third-party text repeated by the assistant, and whether a stored fact changes a later answer. Cover operator/tenant isolation, retention, and correction or deletion while an older extraction is still running.

Use [tools and permissions](tools-and-permissions.md) and [context and memory](context-memory-compaction.md) for these contracts; keep their detailed eval cases here.

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

Grade final business state, emitted UI, and write arguments with code where fields decide correctness. Use a rubric for semantic requirements. Pin a tool choice or ordering only when it is part of the contract, such as a required grounding read or authorization before a write; otherwise accept any route satisfying the outcome and safety invariants.

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

## Component diagnostics

Use matched task outcomes to explain which limitation an intervention addresses, not only whether aggregate success changes. Cross representative context-window budgets with context policies while holding the model, tasks, and other components fixed. Separately ablate progress tracking and action profiles; if resource limits permit only selected combinations, state that scope instead of claiming a full factorial result or unmeasured interactions. Repeat runs where practical, report paired uncertainty, and keep tuning cases separate from held-out evaluation.

| Intervention | Matched comparison | Measurements that distinguish the mechanism |
|---|---|---|
| Context reduction | No cross-turn management, elision, summarization, and staged elision-before-summary at each budget | Success, context-overflow terminations, peak next-call input, reduction-stage counts, summarization calls/cost, and preservation of required active state. Distinguish keeping runs alive from changing behavior. |
| Historical-output recall | Elision with/without recall at identical thresholds; also test staged reduction with/without recall when proposing that combination | Invocation rate, fraction of tasks using recall, recoveries that affect a later decision, task lift, and total retrieval/re-read cost. Keep audit retention constant; lack of model usage does not justify deleting required records. |
| Progress scaffold | Tracking on/off with the same permission mode, completion criteria, and required checks | Time/calls until an actual source edit, abandonment before useful work, unmet done conditions at stop, post-edit verification on unchanged state, long-tail trajectory cost, and verification coverage. |
| Action interface | Explicit workspace tools versus a shell-centric or programmatic profile with equivalent host controls and diagnostics | Out-of-interface emissions, localization failures, actual mutations, repeated edits, action granularity, calls/tokens, success, and safety invariants. If controls or instructions differ, declare the bundled intervention. |

Record runtime termination reasons separately: completed, model stopped without meeting the done condition, context overflow, step/time/cost limit, repeated failure, approval pause, and policy denial. Phase-level call, token, and time totals can locate redundant work, but fewer turns or lower mean cost may simply reflect earlier failure. Apply the same quality floor and [completed-task economics](#model-and-configuration-sweeps) to all profiles, including failed attempts.

Derive physical writes and literal no-edit termination from runtime mutation evidence or observed file state, including shell-mediated changes. Semantic phase labels from a model judge are different measurements: a missing Fix label is not proof that no file was edited. When using judged phases or failure-stage attribution, publish their definitions and validate a sample against human labels; do not replace runtime permission, mutation, or completion evidence with judge agreement.

The behavior contracts remain in [context reduction and recall](context-memory-compaction.md#staged-reduction-under-context-pressure), [progress scaffolds](planning-and-goals.md#execution-time-progress-scaffold), and [coding action interfaces](coding-agents.md#model--and-workload-dependent-action-interfaces). Keep their evaluation matrix here rather than duplicating it in those owners.

## Model and configuration sweeps

Compare candidate models and effort settings against the same quality floor and task mix. First hold the harness and prompt fixed to isolate configuration effects; then allow comparable prompt calibration per candidate on separate tuning cases and report held-out results with each prompt version. Keep those two comparisons distinct so a prompt fitted to one model does not settle the selection unfairly.

Measure cost per successful task with failed attempts included in total cost, time to first useful rendered output, full-task latency, and tail percentiles as well as medians. Weight results by observed traffic where available and report difficult-task failures separately. Faster tokens or a cheaper call do not imply faster or cheaper completion. Use [prompt caching and cost](prompt-caching-and-cost.md) for cache measurements and stable-prefix design.

## Routing, configuration, and warming evals

Use the contracts in [per-request routing](agentic-loop.md#per-request-model-routing), [configuration events](architecture.md#runtime-instruction-and-tool-configuration-events), and [economic warming](prompt-caching-and-cost.md#economic-cache-warming). Establish fixed-model and no-warming baselines before enabling these post-MVP optimizations. Keep task outcomes and host controls matched; use [model/configuration sweeps](#model-and-configuration-sweeps) for completion economics rather than judging isolated cheap calls.

| Fixture | Required evidence |
|---|---|
| Route to a smaller window or incompatible modality/protocol | Capacity and compatibility are checked against the resolved destination before dispatch; compaction/conversion, a declared fallback, or a visible error follows policy without silent content loss. |
| Tool continuation, provider failure, classifier failure, and retry | Selected/dispatched identities remain distinct; continuation stickiness follows the declared policy; failed attempts and pending tool results retain correct attribution and pairing. |
| Fork, restore, concurrent state proposal, failed dispatch, and direct summary request | Branch-state inheritance, expected-version rejection, pre/post-dispatch commit timing, and direct-call state behavior match the contract; recovery cannot invent a successful transition. |
| Cross-provider route with opaque replay data and caller credentials | The adapter handles or rejects replay incompatibility explicitly; credentials resolve for the authorized destination without forwarding source-provider secrets or changing tool authority. |
| Instruction section/tool add, change, remove, and supported full checkpoint | Native projection and collapsed-checkpoint projection reconstruct the intended effective state; unsupported operations are rejected or use an explicit fallback. |
| Revoked tool, stale branch, reordered delta, and compaction during a change | Current host policy denies revoked execution; version/order checks prevent stale configuration; rehydration preserves effective state and authority labels. Inspect payloads separately when old text must be excluded. |
| Known/unknown TTL, low reuse, changed reasoning/output limit, and expired cache | Eligibility and the economic gate distinguish safe refreshes from unproven replay; expired or incompatible requests do not masquerade as cheap hits. |
| Delayed timer, new real turn, model/configuration change, cancellation, and late completion | Obsolete refreshes are not dispatched or adopted; real work has priority; abort requests, confirmations, completed work, and ambiguous billing remain distinguishable. |
| Long idle session, repeated refreshes, failures, and abandoned task | Total costs include routing/classification, warming, retries, compaction, and failed/unknown work. Report realized cache reads, refresh waste, completed-task latency, and quality; scheduled warming and estimated savings are not measured savings. |

Payload-shape tests establish adapter construction, not provider cache reuse or model compliance. Combine them with host-policy probes and measured provider usage when making behavioral or economic claims.

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

## Realtime voice agent evals

Use [realtime voice agents](realtime-voice-agents.md) for media and interaction contracts. Voice can be an MVP: compare a narrow useful scenario with its simplest viable voice baseline, including push-to-talk when appropriate. Treat native speech versus chained speech stages, duplex versus turn-based interaction, and in-session versus delegated execution as separate choices. Test the selected combination; a delegated backend or durable service is not a prerequisite for every voice agent.

Reuse [model/configuration comparisons](#model-and-configuration-sweeps), [timing methodology](#performance-optimization-and-regression-ratchets), [tool authority](tools-and-permissions.md), and the applicable [task lifecycle](always-on-agents.md#durable-tasks-and-owned-work). The matrix below owns voice-specific cases; it does not replace those contracts. Record model, prompt, adapter, tool/policy and media-configuration revisions, acoustic conditions, and playback-cursor provenance. A callback or generated transcript alone cannot establish audible delivery or completed work.

| Fixture | Required observed result |
|---|---|
| Long pause, hesitation, backchannel, cough, background speech, and user speech during output | Endpoint and yield decisions match the declared policy without spurious work or repeated speech; target noise and overlap do not silently transfer turn ownership. |
| Partial recognition, revised transcript, ambiguous name/ID/date/number, accent, and language switch | Accepted intent follows the declared audio/transcript evidence contract; provisional text alone cannot authorize an effect. Critical ambiguity is clarified; recognition accuracy and output pronunciation are scored separately. |
| Automatic detection and explicit/manual response triggers overlap | The effective response-admission owner deduplicates competing triggers; one intent does not produce duplicate speech or duplicate work admission. Test provider-side and framework-level policies together when both are configured. |
| User interrupts after generation finishes but before all output plays | Local and queued obsolete audio stop under the declared fence; later conversation state does not presume unheard output was delivered. Generation, queue drain, and playback receipts remain distinct; estimated cursors are labeled. |
| Interruption while speech generation and tool work are both active | Playback stopping, generation cancellation/yield, history repair, and work disposition each follow policy. A speech stop cannot be scored as tool cancellation, rollback, or settled completion. |
| User corrects a task or revokes a confirmation while work runs; old result arrives later | Current intent/task revision governs result adoption, spoken confirmation, and apply-time eligibility; obsolete results cannot restore old authority. Already committed effects retain their reconciliation outcome. |
| Slow, denied, failed, empty, or partial tool result | Appropriate silence or truthful bounded progress preserves responsiveness; acceptance and filler speech do not count as useful completion, and spoken success agrees with verified task state. |
| Queue saturation, stale queued audio, media deadline, device change, sample-rate mismatch, and dropped output | Duration and age bounds and declared overflow/deadline policy hold; loss and conversion failures are observable rather than reported as successful playback. Control remains responsive. |
| Disconnect, reconnect, retirement, or close with pending audio/work; old-session events arrive | Session/media epochs fence obsolete output and successor mutation. Backend work follows its explicit lifetime; reconnect neither proves cancellation nor permits blind effect retry. A retained transcript tail does not admit duplicate work. |
| Output requires a check before playback; check finishes after interruption or session replacement | Only a check bound to the current eligible segment can release buffered audio. Canceled or obsolete speech stays fenced; a check that finishes after playback cannot claim it prevented audible exposure. |
| Context pressure or voice-history compaction during a correction or pending task | Active intent, work identity, authority, and heard/unheard boundary survive through their existing owners; missing context or imprecise alignment has an explicit disposition rather than fabricated task success. |

Measure the defined input/intent trigger to first **useful audible answer**, endpoint errors, overlap, detection-to-output-silence, queue age, dropped media, task success, incorrect actions, and total completed-task cost. Report medians and tail percentiles under the recorded acoustic/device conditions. Measure acknowledgment/filler latency separately. For duplex interaction, define a causal intent trigger rather than assuming a completed turn; separate detector delay from the delay between detected interruption and stopped output. Use evaluator-owned timing and the existing clock-domain rules.

Start with repeatable synthetic speech, then representative human recordings and independent multi-turn caller scenarios, with listening review of the rendered output. Keep task-state correctness and audio intelligibility/delivery as independent acceptance gates. Set product thresholds before launch; do not accept a plausible transcript, fast acknowledgment, or good audio as evidence that actions and conversational state are correct. Optional delegation, durable recovery, and speculative work must justify their complexity against the applicable simpler baseline.

## Hardware agent evals

For [hardware agents](hardware-agents.md), extend the existing case/trace format with board/runtime revision, source/artifact hash, state schema, transport substitutions, clock basis, and physical observation duration. Separate native fixtures, component emulation, production-loop emulation, and physical commissioning; grade only paths each layer actually executes.

| Probe | Required observed result |
|---|---|
| Existing launcher with another app; compiled device with a different selected slot | Installer follows measured boot/discovery behavior; no guessed offset, base reflash, or removal of unrelated apps. |
| Image just beyond slot size; simulator image offered for upload | Deployment stops before writing. |
| Mount failure after identity exists; interrupted checkpoint replacement | No silent formatting/reinitialization; old valid state remains recoverable or writes pause. |
| Power cut before dispatch, after remote acceptance, and before receipt checkpoint | Pending action reconciles without blind replay; unknown outcome is explicit. |
| Largest context/catalogue and repeated TLS handshakes | Target parser/string/depth and application/native memory limits hold, or return bounded errors without leaking sockets. |
| Fragmented/chunked streams, long headers, narrow timeout overflow, missing completion | Exact deadline semantics hold; partial/ambiguous output cannot authorize an action. |
| Multi-byte network names and escaped surrogate pairs | Target runtime preserves valid text or reports failure; identifiers/arguments are never cosmetically clipped. |
| Token rotation followed by reset | New credentials persist under one refresh owner; secrets remain out of public traces/packages. |
| Clock absent/corrected, background sync, due jobs, and saved wakes | Verified TLS and logical time follow policy; sleep ceiling and server minimum holds survive restart/migration. |
| One successful turn followed by timeout; failed compaction | Retry policy does not accidentally suppress useful work indefinitely; context never exceeds its hard ceiling. |
| Low-memory recovery with a pending effect or remaining sleep | Recovery occurs only at a safe recorded boundary and does not create an extra cycle or reboot loop. |
| Concurrent serial reader, stale screen phase, queued wake keys | Missing logs are not mislabeled as reboot; status/input claims require the applicable physical evidence. |
| Embedded calculator/compiler optimization bypass | Metering is exercised on-device; unsafe programs fail without gaining file/network/credential access. |
| Wrong identifier type for inbox acknowledgement | Runtime reports the mismatch and does not claim unprocessed work acknowledged. |
| Rollback after state-schema/token/receipt evolution | Compatible current authority/effect state is preserved, or rollback stops for an explicit recovery decision. |

Launch gates additionally require physical boot and a cycle on the exact artifact, with remote resulting-state verification for authorized effects. Scheduled products also need sleep/wake evidence; resumable products need a safe reset/power-cycle recovery probe. Unattended or overnight reliability needs a declared, completed soak; a passing host-backed run or first hour is not a substitute.

## Adaptive agent-team evals

[Adaptive agent teams](adaptive-agent-teams.md) are an advanced, post-MVP profile. Evaluate the proposed coordination contracts and their measured contribution; published outcomes or team counts do not establish how a research system implemented them or which mechanism caused its gains.

Compare at least these conditions with the same model/settings, task fixtures, tools, authority, acceptance criteria, and total resource budget:

```text
single-agent research loop
ordinary parallel workers executing bounded packets
adaptive teams retaining distinct approach charters
```

A fixed team portfolio is a useful intermediate comparison before allowing allocation changes. Match available concurrency where the comparison permits it and report differences explicitly. If budgets cannot be matched, report the quality/cost frontier rather than attributing additional compute to coordination. Use held-out investigations with plausible competing explanations, misleading early leads, and independently checkable acceptance evidence.

Ablate these mechanisms separately; keep other team controls fixed and declare any coupled changes:

| Mechanism | Matched comparison | Diagnostic measurement |
|---|---|---|
| Varied formulations | Distinct formulations, assumptions, or methods versus the same approach under different wording | Validated coverage of competing explanations, overlap in explored questions, and recovery from misleading leads. |
| Work awareness | Work-intent visibility and overlap checks on/off with the same permitted replication policy | Unintended duplicate work, missed semantic overlap, false conflicts that block useful work, and registry/coordination cost. |
| Cross-team communication | Finding exchange on/off while keeping team-local communication fixed | Useful finding adoption, time saved, message overhead, unsupported-claim propagation, and dependence among apparent corroborations. |
| Adaptive allocation | Evidence-linked allocation changes versus a fixed portfolio with the same aggregate budget | Quality and cost to reach acceptance, abandoned approaches, retained challenge coverage, and failed or unnecessary transitions. |

State which interactions were actually tested. Do not infer benefits from formulation diversity or larger team count when communication or available compute changed at the same time.

Use controlled interleavings and restorable fixtures for these probes:

| Probe | Required observed result |
|---|---|
| An unsupported finding is shared, echoed by several teams, and sent to a reviewer | Provenance and consumption dependencies survive; echoes cannot count as independent validation or convert the claim to verified status. |
| A replication task receives the result it was meant to check before sealing its own artifact | The run reports the exposure and loses the claimed independence, or the declared visibility policy prevents it. |
| A promising early lead conflicts with later evidence from a minority approach | Contradictions remain visible; allocation and acceptance follow the declared evidence policy rather than message volume or consensus. |
| Different labels hide the same question/method/input scope; similar labels conceal distinct work | Semantic overlap and false-conflict rates are graded against fixture labels; useful distinct work is not suppressed merely by names. |
| Unintended overlap and explicitly reserved independent replication occur together | The harness distinguishes waste from declared replication and applies the replication visibility policy. |
| Two teams simultaneously claim overlapping work; one lease expires while its worker can still write | The ownership conflict is explicit and current fencing prevents conflicting writes; expiry alone cannot establish safe reassignment. |
| Duplicate or stale create/split/merge/retire/retask decisions arrive | Accepted identity/version checks prevent additional teams, repeated allocation debits, or silent overwrite of a newer portfolio. |
| A proposed change exceeds authority, changes the parent objective, or lacks capacity | The transition returns the applicable denial/conflict outcome without partially changing ownership or budget. |
| A result arrives after retasking, merging, or retirement | Its producing charter and intent remain intact; relevance is reassessed before adoption and it is not credited to the new approach. |
| A recipient is unavailable; a message arrives after its intended charter changes | Delivery has a bounded observable outcome; the late message cannot revive obsolete work or act as a current assignment. |
| Restart occurs between transition acceptance, worker admission, and handoff | Reconciliation recovers the recorded decision and outstanding work without duplicate admission, lost ownership, or blind side-effect replay. |
| Budget exhaustion occurs with reservations, in-flight work, and a pending transition | Aggregate accounting includes spent, reserved, and outstanding work under the existing budget contract; the run stops or degrades with an explicit incomplete outcome. |
| All teams become idle or retire with unresolved questions; a rollback leaves external effects | Quiescence is not scored as acceptance; the run reports remaining work and effects rather than claiming completion or restored spent budget. |

Measure task quality, validated coverage, false-success rate, cost to reach acceptance including failed approaches, elapsed time, unresolved/failure rate, and human intervention. Also measure unintended overlapping work separately from declared replication, coordination/message cost, allocation churn, stale-decision rejection, stale-result adoption, and independence-policy violations. Use runtime records for ownership, transitions, authority, and accounting; a model judge's interpretation of an approach is not evidence that these controls held.

Launch only when the profile improves a declared outcome over the simpler baselines at the required quality floor and within the resource envelope. Authority, ownership, accounting, stale-state, and independent-acceptance probes must have no unresolved control failures; report the trials and observed failures rather than treating a passing suite as universal proof. Keep adaptive allocation disabled when it cannot justify its overhead or when these invariants regress. Acceptance still follows the parent goal and validation policy, not team consensus, scale, or inactivity.

## Always-on and durable runtime evals

Use [always-on agents](always-on-agents.md) for runtime contracts and [concurrent compaction publication](context-memory-compaction.md#concurrent-compaction-publication) for summary placement. Compare a durable single worker with the request-scoped baseline before adding background work or takeover. Keep model, tasks, authority, and resource limits matched; measure accepted-work loss/duplication, recovery time, unresolved effects, false completion, observer lag/gaps, resource growth, and completed-task cost.

| Probe | Required observed result |
|---|---|
| Disconnect after admission; retry the same ID, then change its payload | One accepted intent/receipt under the declared conflict and retention policy; admission is not scored as an answer or exactly-once effect. |
| Busy steering/follow-up/write; withdrawal before and after placement; run failure | Declared ordering and boundary apply; control remains usable; recovery exposes every queued/placed/withdrawn disposition. |
| Reject storage; lose acknowledgement; fail adoption after commit | Rejection has no partial effect; uncertain state seals mutations and reconciles on reopen; no uncommitted view is published. |
| Crash before intent, after external acceptance, and before outcome commit | Recorded and current replay policies govern recovery; unknown effects reconcile without blind writes or fabricated success. |
| Missing/reloaded definition; old/new checkpoint version; migration failure | Compatible work resumes with provenance; incompatible work stays blocked or takes an explicit orphan disposition without invented cleanup. |
| Owner returns a result with a live foreground child; child fails during fail-fast | Terminal receipt waits for required owned work; held result is not scored as settled completion; parent outcome follows declared policy. |
| Cancel wait/task/conversation/full background set; noncooperative tool; close/reopen | Each lifetime boundary is observed; abort intent fences later runtime writes; compensation/unknown effects remain distinct from marking; close preserves recoverable work. |
| Fork inside a multi-entry commit with historical/current/fresh documents | Declared transaction granularity and policies hold; independent incarnations and stored versions survive; task state/current authority are not copied accidentally. |
| Snapshot attachment races a commit; observer stalls or reconnects after a gap | No acquisition gap; bounded count/bytes; snapshot convergence or retained replay matches the promise; no lossless-audit claim from coalesced views. |
| Retire/recreate a document; stop with a callback already running | Old incarnation does not follow recreation; active-callback stop/join policy and terminal delivery are explicit. |
| Concurrent summaries finish out of order; new tail/reset/edit appears; crash around submission/placement | Valid cut wins under the declared equal-cut policy; no tail/configuration loss or duplicate placement; stale/unknown cost remains accounted for. |
| Process kill and declared host/power failures; takeover with stale writer; duplicate/late wakeup | Recovery matches the storage failure class; fencing holds before takeover; wakeup admission rechecks authority and budgets; acknowledgement does not overstate durability. |
| Missed schedule; new partition or changed query; timestamp ties, late updates, and multiple source pages | Each partition resumes or initializes under its declared collection policy; complete coverage is established before advancing, with no loss hidden by a newest-seen timestamp. |
| One source succeeds empty while another is unavailable, truncated, or fails on a later page | Per-source complete/partial/failed coverage is visible; failed/partial partitions retain their last complete checkpoint and cannot be scored as quiet. |
| Publication fails or its receipt is lost; crash before bookkeeping; overlapping edition attempts | No false delivered-item entry; unknown publication reconciles under existing effect rules; collection advancement without confirmed delivery or permitted suppression retains pending candidates, and concurrent claims cannot bypass edition ownership. |
| Permitted silence; budget pause; held/unknown publication; missing run | Suppressed editions have a policy reason and no delivery receipt; source gaps remain visible; run health and delivery status are not scored as a complete empty scan. |

Use controlled interleavings and actual reopened storage, recording source/runtime/storage revisions and failure injection boundaries. Process-crash probes do not substitute for host/power-loss tests, and adapter conformance does not establish distributed ownership. Gate unattended rollout on no unresolved admission, authority, publication, ownership, or false-completion failures in the declared trials; suspend background/takeover features when they cannot justify their complexity.

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

## Performance optimization and regression ratchets

Use this contract when performance improvement is an explicit objective. A measured baseline and regression gate can serve an MVP; unattended optimization, recurring monitoring, and parallel workstreams remain post-MVP. Apply the existing [feedback loop](agent-legibility-feedback-loops.md#feedback-loops) within its authorized scope.

### Measurement contract

Name the target boundary: harness-local processing, provider inference, tool/network wait, or client rendering. Define each journey from its triggering interaction to an observable completion event. Use [existing latency and task-economics definitions](#model-and-configuration-sweeps) and [correlated stage evidence](security-observability.md#capture-the-failing-boundary); a local improvement alone does not establish faster user-visible completion. Attribute overlapping work through the critical path rather than summing all stage durations.

The evaluator owns timing boundaries and counters. Record the clock domain; use monotonic elapsed time within one process and explicit clock alignment or causally correlated spans across processes. Pin fixtures, data shape, cache state, dependencies, runtime, and measurement environment. Isolate harness overhead with fixed responses or mock tools where appropriate, then separately validate live end-to-end behavior; that isolation does not measure model quality or provider latency.

Keep a versioned performance record:

```text
journey and completion boundary
measurement owner and clock domain
workload, fixture, and cache-state identity
implementation, benchmark, and environment versions
proxy-to-latency validation evidence
trial aggregation and noise tolerance
quality, safety, and cost constraints
active threshold and prior baseline
rollout cohort and acceptance or revert evidence
```

### Proxy acceptance

A cheap counter is an optimization target only after two checks: it is repeatable for the declared workload, and improving it produces a meaningful latency improvement on representative cases. Instruction, allocation, or call counts are candidates, not universal guarantees. Reject unstable proxies and counters that improve while work moves outside the measured boundary or user latency fails to improve. Repeat calibration when workloads or measurement environments change.

Wall-clock gates remain valid when repeated trials and declared uncertainty support them. For synthetic clocks or frame stepping, distinguish the simulated budget from observed physical-device performance. Use instrumentation overhead and correctness checks to ensure a lower count was not bought by dropping required work.

### Ratchet and rollout acceptance

A performance ratchet is a versioned upper bound for a lower-is-better metric. Run the benchmark on affected changes and reject regressions beyond its declared tolerance. Tighten the bound only from repeatable, accepted evidence; the best of many noisy samples is not a new baseline. Candidate optimizations must not weaken the benchmark, fixtures, threshold, or quality floor to pass. Evaluation changes require separate review and preserve the previous record; an incompatible environment or workload change needs a newly calibrated baseline rather than a silent reset.

After an authorized rollout, compare the same journey under representative traffic, cache states, and concurrency. Accept the win only when user latency improves without violating the existing quality, safety, or cost constraints. A lab-only gain stays provisional; a failed field check follows the deployment's existing revert or disable procedure. Record the measured outcome before tightening the regression threshold, and retain the check in the [regression suite](#regression-loop).

Stop or redirect experiments when measured benefit no longer justifies complexity or the declared budget. Keep scope, permissions, stopping rules, and any concurrent-worker lifecycle with their existing [goal](planning-and-goals.md), [tool](tools-and-permissions.md), and [workflow](workflow-orchestration.md) owners.

## Regression loop

Every incident, review finding, or repeated manual correction should become a regression eval:

1. Preserve the failing task, trace, fixtures, and state snapshot.
2. Reduce it to the smallest replayable case.
3. Define the expected trace behavior and final status.
4. Patch the harness, tool schema, permission policy, context builder, or instruction source.
5. Add the case to the recurring suite.
6. Track pass/fail by model and harness version.

Select change-time coverage from a core set of common tasks, every safety case, affected capabilities, and their neighboring boundary cases. A tool change includes its callers; a shared prompt change runs the full suite. Run the full suite before release and periodically to catch regressions outside the selected set, using several trials and declared pass thresholds for nondeterministic cases.

Re-scoring stored outcomes tests graders and preserves diagnostics; it does not test a changed model, prompt, skill, or runtime. Those changes need fresh executions of the selected cases. Record the case and scorer behind each known failure so a different failure on the same case remains visible.

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
