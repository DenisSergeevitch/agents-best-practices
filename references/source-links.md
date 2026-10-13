# Source Links

Use this file when the user asks for cited, provider-specific, or standards-backed agent-harness guidance.

## Agent Skills

- Agent Skills specification: https://agentskills.io/specification
- Agent Skills creator best practices: https://agentskills.io/skill-creation/best-practices
- Optimizing skill descriptions: https://agentskills.io/skill-creation/optimizing-descriptions
- Evaluating skill output quality: https://agentskills.io/skill-creation/evaluating-skills
- Using scripts in skills: https://agentskills.io/skill-creation/using-scripts

## Harness bug report and troubleshooting

- Mike Piccolo, [53 harness bugs encountered during development](https://x.com/mfpiccolo/status/2102089734004593140), posted September 21, 2026; post metadata retrieved through the [public mirror](https://api.fxtwitter.com/mfpiccolo/status/2102089734004593140) and the [original attached checklist image](https://pbs.twimg.com/media/HSwfeSNXEAALzSk?format=jpg&name=orig) inspected September 21, 2026. The note in the image is dated September 16, 2026.

This is an author-reported incident list, not a new model, harness architecture, or controlled evaluation. It motivates checks around protocol completion, runtime lifecycle, context/accounting, and observability overhead. The post and image provide no pinned implementation, tests, or reproducible traces; the incidents and numerical claims were not independently reproduced. Do not generalize its provider counts, resource limits, timings, cost differences, or percentages into defaults.

The symptom tables and isolated probes in [troubleshooting](security-observability.md#troubleshooting) are this skill's diagnostic synthesis, not fixes verified against the author's runtime. Existing references remain canonical for loop invariants, streaming adapters, typed tool results, permissions, workflow state, compaction, cache economics, and regression methodology; no separate architecture profile is introduced.

## Empirical coding-harness component selection

- Paper: [An Empirical Study of Harness Design for Coding Agents, arXiv:2609.20804v1](https://arxiv.org/abs/2609.20804v1), submitted 17 September 2026; original HTML rechecked 19 September 2026.
- Primary evidence: [context policy](https://arxiv.org/html/2609.20804v1#S2.SS3), [setup](https://arxiv.org/html/2609.20804v1#S3.SS1), [results](https://arxiv.org/html/2609.20804v1#S3.SS2), [trajectory analysis](https://arxiv.org/html/2609.20804v1#S4), [planning prompts](https://arxiv.org/html/2609.20804v1#S7.SS2), and [limitations](https://arxiv.org/html/2609.20804v1#Sx1).

This is empirical calibration of familiar harness mechanisms, not a new model, training method, or autonomy level. The study evaluates three Nemotron-3 sizes and Mistral-Medium-3.5-128B on SWE-Bench Verified and Terminal-Bench 2.1. Its 176 settings comprise five context policies at four window budgets plus separate planning and action-interface ablations; only the context sweep covers all window budgets.

Reported evidence worth retaining:

- The mean managed-versus-unmanaged success gap on SWE-Bench falls from 35.7 percentage points at 32k to 2.7 at 128k. Managed policies report zero context-overflow failures. The staged policy has the lowest aggregate token-priced cost at each tested budget, not uniformly the best outcome in every cell.
- Adding historical-output recall to elision has a mean success difference of -0.36 percentage points over 32 comparisons; 36 of 64 recall-enabled context settings never invoke it. This is a result about the evaluated recall interface, not general evidence against retrieval or durable evidence retention.
- Persistent progress planning raises the weakest model's SWE-Bench success from 13.6% to 25.2% at higher cost. For the two strongest evaluated models, nominal cost falls roughly 30–32% with small success decreases; judged trajectories attribute much of the reduction to post-edit verification, not proof that required checks can be omitted.
- For Nemotron-3 550B on SWE-Bench, bash-only changes success from 65.8% to 69.4% and mean cost from $2.33 to $1.11. For Mistral on that benchmark it instead drops success from 68.6% to 45.4%, while its interface preference reverses on Terminal-Bench. Model scale alone does not identify the right interface.

Keep the study's limits distinct from stronger guidance in this skill:

- Planning means a persistent todo scaffold, not read-only permission mode. Bash-only retains enabled auxiliary planning and recall tools. The action-interface intervention jointly changes tools, prompts, file-state tracking, and automatic diagnostics; Appendix 8 says shell edits do not share the structured-file-tool tracking/diagnostic path. It does not establish equal safeguards or a causal benefit from fewer tools alone.
- The staged policy includes recall and earlier elision; elision-plus-summary without recall is not separately tested. Simpler policies use the later threshold. The lean staged combination is a candidate to evaluate, not an independently reproduced winner. The study's threshold/window fractions are implementation settings, not portable defaults.
- Planning and action interfaces are ablated only under the full staged policy at 128k. Each setting runs once per task; Terminal-Bench has 89 tasks and many contrasts are not statistically significant. Transfer to other models, task types, or budgets remains unestablished.
- Inference is locally served while dollar costs use nominal token prices, not measured production bills or latency gains. “Without edit” is derived from judge-assigned phases; literal mutation counts require runtime/file evidence rather than assuming no Fix label means no write.
- The paper, prompts, tool descriptions, and relevant trajectory appendices were inspected. No study implementation or trajectory repository was linked from the arXiv HTML or Hugging Face metadata inspected; implementation behavior is paper-reported, not source-code audited, and experiments were not reproduced.

Canonical guidance lives in [staged context reduction](context-memory-compaction.md#staged-reduction-under-context-pressure), [historical-output recall](context-memory-compaction.md#historical-output-recall), [progress scaffolds](planning-and-goals.md#execution-time-progress-scaffold), [coding action interfaces](coding-agents.md#model--and-workload-dependent-action-interfaces), and [component diagnostics](evals.md#component-diagnostics). Reuse existing permission, state-preservation, cache, and evaluation owners instead of creating another architecture profile.

## Pi 1.0 routing, configuration events, and cache warming

- Release article: [Pi 1.0](https://earendil.com/posts/pi-1-0/), October 1, 2026.
- Pinned implementation: [v1.0.0 at `a13d35a742c6ef8462812a28fbe1d8c8b7431c32`](https://github.com/earendil-works/pi/tree/a13d35a742c6ef8462812a28fbe1d8c8b7431c32), released October 1; README, relevant docs, source, and tests inspected October 4, 2026. Tests were not executed and provider cache behavior was not reproduced.
- Routing: [virtual-model documentation](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/docs/virtual-models.md), [model runtime](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/src/core/model-runtime.ts), [session preflight/state](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/src/core/agent-session.ts), and [routing suite tests](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/test/suite/virtual-models.test.ts).
- Configuration: [message documentation](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/docs/message-types.md), [actual message type](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/ai/src/types.ts), [transcript replay/projection](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/ai/src/utils/transcript.ts), [Anthropic projection](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/ai/src/api/anthropic-messages.ts), and [tool-change tests](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/ai/test/transcript-tool-changes.test.ts).
- Warming: [cache warmer](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/src/core/cache-warmer.ts) and [warmer tests](https://github.com/earendil-works/pi/blob/a13d35a742c6ef8462812a28fbe1d8c8b7431c32/packages/coding-agent/test/cache-warmer.test.ts).

This release composes familiar routing, event-sourcing, progressive-disclosure, and programmatic-tool mechanisms. It adds useful implementation evidence for gaps in this library, not a new model, training method, or autonomy level. Codemode and deferred loading reuse the existing tool and connector owners; terminal styling adds no harness contract. The separately announced experimental Pi Durable package was outside this October 4 intake; the later, separately pinned intake below covers it.

Keep source-observed behavior separate from this skill's stronger guidance:

- Virtual selection and physical dispatch are recorded separately. The router receives user/continuation/retry/direct request reasons. Keeping continuations and retries on the previous model is example policy, not a host guarantee. Changed branch routing state persists before dispatch and survives a failed call; direct calls do not receive/persist it. Session preflight checks the routed model's window and can compact while retaining the chosen route. Versioned state, explicit compatibility gates, and permission invariants here are generic requirements, not proof that every source path enforces them.
- Ordered system messages accumulate text, patch/delete named sections, and remove/add tools. **The released documentation describes `replace: true` as a full reset, but the actual type and replay implementation contain no such operation.** Do not attribute reset semantics to this revision. Provider projection is capability-dependent: unsupported incremental paths collapse effective state into a leading checkpoint; some tool transports fall back on removal/redeclaration. Native removal can leave old text/definitions in payload history, so active removal is not erasure or proof of model compliance.
- Warming replays eligible requests with a one-token output cap and no automatic retries; it excludes incompatible budget-based thinking, uses declared best-effort lifetimes, estimates reuse/savings, caps duration, and aborts replaced work. Its probability and savings constants are implementation choices. Only successful, still-current refresh usage is persisted; failed/aborted and invalidated late outcomes may be absent. Complete billing reconciliation and total realized savings require additional evidence. The tests check scheduling, option preservation, economics, invalidation, and abort signals, not real cache hits or guaranteed savings.

Canonical additions: [per-request routing](agentic-loop.md#per-request-model-routing), [runtime configuration events](architecture.md#runtime-instruction-and-tool-configuration-events), and [economic warming](prompt-caching-and-cost.md#economic-cache-warming). Reuse instruction hierarchy, context preservation, tool permissions, adapter compatibility, and [focused evals](evals.md#routing-configuration-and-warming-evals) by link rather than creating another architecture profile.

## Application performance optimization case study

- Dated source: [How we made claude.ai 3x faster in two weeks](https://claude.dev/blog/how-we-made-claude-ai-faster/), Raymond Wang, Sam Attard, and Issac G., published September 23, 2026; inspected October 6, 2026.
- The authors describe lab proxies checked against wall-clock latency, rejection of flaky or non-correlating benchmarks, CI ceilings lowered after improvements, and post-deployment field reads deciding whether to retain a change. They report human steering and approval of every shipped change.
- The reported comparison uses August 13 versus August 27 real-user p75 measurements of application journeys. It does not measure harness overhead or isolate the effect of the agent from human work and concurrent changes.

This is a first-party engineering account without linked benchmark code, raw telemetry, or a pinned implementation commit. Numerical gains, benchmark determinism, synthetic frame-stepping results, and incident counts remain author-reported. The [performance eval contract](evals.md#performance-optimization-and-regression-ratchets) adds our provider-neutral requirements for measurement provenance, versioned baselines, reviewed evaluator changes, and production acceptance; these are stronger guidance, not verified implementation claims. Reuse the existing feedback, tracing, permission, and workflow owners.

## Pi Durable task and conversation runtime

- Launch article: [Pi Durable](https://earendil.com/posts/pi-durable/), October 1, 2026. The article describes an experimental framework alongside the coding-agent release.
- Pinned implementation: [main at `b2b5c42f6138b73ec4b2f49ec0ca468800f88586`](https://github.com/earendil-works/pi/tree/b2b5c42f6138b73ec4b2f49ec0ca468800f88586), committed October 5, 2026; [package version 1.0.2](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/package.json). README, spec, relevant source, and tests inspected October 5. The upstream test suite was not executed, and production recovery, provider behavior, and power-loss durability were not reproduced.
- Contract map: [README](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/README.md) and [runtime/storage specification](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/docs/spec.md).
- Commit and storage evidence: [Session admission/publication](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/session/session.ts#L405), [JSONL marker protocol](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/storage/jsonl/storage.ts#L277), and [storage conformance cases](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/testing/storage-conformance.ts).
- Execution evidence: [scheduler](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/scheduler.ts), [generation](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/generation.ts), and [tool intent/replay](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/tool.ts#L85).
- Recovery and ownership tests: [task recovery](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-tasks-recovery.test.ts), [model recovery](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-generation-recovery.test.ts), [tool recovery](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-tools-recovery.test.ts), and [structured task completion/cancellation](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-structured.test.ts).
- Intake evidence: [submission admission](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/submissions.ts#L148), [inbox boundaries](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/inbox.ts#L58), [submission tests](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-submissions.test.ts), and [inbox tests](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-inbox.test.ts).
- State and observation evidence: [document definitions](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/documents.ts), [fork tests](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/session-forks.test.ts), [watch delivery](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/session/observation.ts#L200), and [watch tests](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/session-watches.test.ts).
- Context and progress evidence: [compaction](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/compaction.ts), [compaction tests](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/test/harness-compaction.test.ts), and [adaptive output batching](https://github.com/earendil-works/pi/blob/b2b5c42f6138b73ec4b2f49ec0ca468800f88586/packages/durable/src/harness/output.ts#L250).

This is a composition of established durable execution, event-sourcing, typed state, structured concurrency, and conversational control patterns. It supplies a reusable post-MVP architecture profile, not a new model, training method, or autonomy level. The contribution is the set of contracts across commits, recoverable tasks, ownership, intake, and context placement.

Source-observed contracts:

| Boundary | Behavior at the pinned revision |
|---|---|
| Commit publication | A serialized Session line admits writes and publishes after storage acceptance. A guaranteed `StorageRejected` rolls back; an uncertain storage or adoption failure poisons the Session and requires reopen. |
| Model recovery | Request cutoff, model, thinking level, and stream options are checkpointed. Recovery converts a committed partial into an aborted assistant entry, resends through the original cutoff, and reruns request hooks. Deferred requests preserve a handle and polling deadline. |
| Tool recovery | Final arguments and replay policy commit before execution. Default policy is unsafe. Recovery reruns only when stored and current policies both say safe; it uses the current implementation/environment and clears the interrupted attempt's progress. Otherwise it records interruption and possible partial execution. |
| Owned-work completion | An outcome can hold as `completing` until ordinary owned work drains. Result writes can land at hold while terminal receipt, task-document retirement, and task waiters remain deferred. A held outcome is final; background subtrees do not participate in the barrier. |
| Cancellation and close | Durable abort marks cascade through ordinary ownership, with abort handlers running after descendants become terminal. `abortTask()` reports marking after joining the active run; conversation abort waits ordinary idle. Close suspends invocations without durable abort outcomes so work can resume. |
| Submission identity | `(conversation, requestId)` returns the original submission before the busy check; changing its content does not reject, while reusing it for the other submission type does. Admission, placement, and answering have distinct receipts. |
| Inbox placement | Post-tools boundaries place writes and steers; final boundaries also place follow-ups. Selected writes precede selected user messages. Reset changes the boundary/context; failed runs retain the queue, while conversation abort withdraws queued inputs and keeps writes. |
| Branch state | History parenting and task ownership are separate. Document definitions choose latest/rewindable history and fork initialization/copying. As-of state is the final state of the selected entry's commit; task execution and task-scoped documents are not copied into forks. |
| Observation and summary placement | Slow watches can converge through a replacement snapshot. Background summary completion can mean its write submission was merely admitted; later boundary placement can reject an obsolete cut as stale. |

Keep these limitations separate from stronger generic guidance:

- The article's exactly-once submission claim is scoped to deduplicated admission. An external effect can occur between committed intent and committed outcome; safe replay, external idempotency keys, polling, or explicit interruption/reconciliation remain application responsibilities. First-writer-wins memos preserve committed decisions, not an atomic transaction with an external approval or payment.
- Recovery pins committed request state, but request hooks rerun and may change the transformed wire input. A replay-safe tool can resolve changed code or environment. Restart does not guarantee the same output, permissions, effects, or cost. Missing/incompatible definitions block recovery; aborting blocked work can become `orphaned` without its cleanup code.
- The storage is single-owner with no cross-process locking. SQLite WAL/NORMAL can lose newest acknowledgments on power/host failure. JSONL `fsync: true` flushes affected sidecars before the marker but does not flush the main marker during ordinary publication; an acknowledged tail may disappear. Do not read the article's failure-survival language as a universal durability or failover guarantee.
- **README wording that a crash loses at most the default 100 ms progress window is stronger than the implementation.** Adaptive tool progress batching uses a minimum interval plus a size-based delay and coalesces behind in-flight commits; model partials also wait on storage. Those settings do not establish a maximum crash-loss window.
- Task ownership is lifecycle attribution, not access control. Foreground cancellation excludes background boundaries; explicit background-inclusive abort covers reached work at admission, not future background work. Bottom-up order follows committed terminal records, not external code run after committing them. A successful abort receipt does not prove rollback of every external effect.
- Watches expose current structural state and may skip intermediate transitions under backpressure; reconnect hydrates a snapshot rather than replaying missed events. Stopping a watch does not join an already-running callback. A watch is not an audit log or exactly-once notification stream.
- Compaction orders successful summary writes by cut position; an older cut becomes stale, while equal cuts can replace. Concurrent application edits can be lost or heads undone outside the supported summary/reset pattern. The serializer omits system text and clips tool-result text, so exact evidence and active-state preservation are stronger requirements. Failed/crashed summaries without committed response usage may be absent from accounting; stale successful summaries still incur cost.
- Active transcript bounds do not prove universally bounded memory or storage: loaded document state is cached and document replay depends on deliberate checkpoints. Fork copying and retained historical state do not by themselves grant permission to reuse private data or establish retention/deletion enforcement.

Payload fingerprint conflict checks, deployment fencing, authority revalidation, complete external-effect/billing reconciliation, and deletion-generation guards in this skill are proposed stronger requirements, not implementation guarantees inferred from these sources.

Canonical guidance lives in [always-on agents](always-on-agents.md) and [context and compaction](context-memory-compaction.md). Reuse the existing instruction hierarchy, tool permissions, goals/budgets, security/observability, and evaluation owners by link; source-specific storage settings and experimental APIs remain in this intake.

## Scheduled collection and delivery bookkeeping

- Lance Martin and CJ Avilla, [Building effective agent automations](https://claude.dev/blog/building-effective-agent-automations/), published October 8, 2026; article and linked implementation rechecked October 13, 2026.
- Reference implementation: [Claude Quickstarts daily brief at `9ec32b91df50b0d4906cae64b13c6298055ff40d`](https://github.com/anthropics/claude-quickstarts/tree/9ec32b91df50b0d4906cae64b13c6298055ff40d/managed-agents/daily-brief), committed October 7, 2026. Inspected the [agent instructions](https://github.com/anthropics/claude-quickstarts/blob/9ec32b91df50b0d4906cae64b13c6298055ff40d/managed-agents/daily-brief/agents/daily-brief/agent.md), [deployment](https://github.com/anthropics/claude-quickstarts/blob/9ec32b91df50b0d4906cae64b13c6298055ff40d/managed-agents/daily-brief/agents/daily-brief/deployment.md), README, environment and memory-store configuration, credential setup, and inspection utilities. No runtime was deployed or source code executed; the inspected quickstart contains no deterministic collector/delivery implementation or corresponding tests.

This is a composition of incremental collection, partial-result reporting, and external-effect bookkeeping, not a new model, training method, or autonomy level. The useful delta is their monitoring-specific relationship; generic scheduling, memory, permissions, budgets, and retries already have owners.

Source-observed limits:

- Bookmarks, failed-read coverage, live-status checks, receipt-before-bookkeeping, and uncertain-send handling are prompt instructions. Pagination, complete scan boundaries, timestamp ties, late updates, and per-repository checkpoint semantics are unspecified; the requested ten-minute overlap conflicts with the literal Slack `oldest=bookmark` instruction.
- Publication checks Slack's response body for `ok` and a message timestamp, but the quickstart supplies no atomic bookkeeping or concurrent-run exclusion. Its quiet branch asks for a posted status/message ID despite sending nothing; duplicate-title bookkeeping is unclear.
- Read-only preferences and mutable state are deployment configuration. The default shared Slack token can write in joined source channels, as the README warns; destination-only and dry-run no-post behavior remain prompt constraints. GitHub read-only access depends on the supplied token's actual rights.
- A per-run cap can pause a session and appear as silence; see [scheduled deployment budgets](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments#set-a-budget-on-each-run). The quickstart includes manual inspection, not an independent scheduled watchdog.

The [recurring collection contract](always-on-agents.md#recurring-collection-and-delivery-bookkeeping) and its [failure probes](evals.md#always-on-and-durable-runtime-evals) are stronger provider-neutral guidance, not guarantees demonstrated by this sample. Reuse existing transaction, external-effect, provenance, permission, and observability owners by link.

## Realtime voice agents

Primary documentation and implementation inspected October 13, 2026. These sources support the [voice architecture profile](realtime-voice-agents.md) and [provider adapter distinctions](provider-api-patterns.md#realtime-voice-adapters); the profile combines established voice/media and harness mechanisms, not a new model, training method, or autonomy level.

- Architecture comparison: OpenAI [Voice agents](https://developers.openai.com/api/docs/guides/voice-agents), covering delegated full duplex, native speech-to-speech, chained STT/agent/TTS, and separate task, audio, timing, and reliability evaluation.
- Current model pages: [GPT-Realtime-2.1](https://developers.openai.com/api/docs/models/gpt-realtime-2.1) and [2.1 Mini](https://developers.openai.com/api/docs/models/gpt-realtime-2.1-mini) are reasoning voice models with tool use and 128k context; [GPT-Live 1](https://developers.openai.com/api/docs/models/gpt-live-1) listens while speaking and delegates reasoning/tools to a separate backend. This does not support a blanket claim that voice models cannot reason or always require a second model.
- Realtime protocol: [starter](https://developers.openai.com/api/docs/guides/realtime), [VAD](https://developers.openai.com/api/docs/guides/realtime-vad), [interruption/truncation](https://developers.openai.com/api/docs/guides/realtime-conversations#interruption-and-truncation), and terminal [response.done schema](https://developers.openai.com/api/reference/resources/realtime/server-events#response.done).
- Live protocol: [getting started](https://developers.openai.com/api/docs/guides/live), [delegation and tools](https://developers.openai.com/api/docs/guides/live-delegation), [session/transcript lifecycle](https://developers.openai.com/api/docs/guides/live-conversations), [prompting](https://developers.openai.com/api/docs/guides/live-prompting), and [migration](https://developers.openai.com/api/docs/guides/live-migration).
- Output checks and controls: [server-side controls](https://developers.openai.com/api/docs/guides/voice-server-controls?api=live), especially [checking speech before playback](https://developers.openai.com/api/docs/guides/voice-server-controls?api=live#check-speech-before-playback). Concurrent transcript checks cannot retract audio already played; buffering adds latency and withheld speech can remain in model context.

Documentation version boundaries:

- The Realtime starter uses `gpt-realtime-2.1`, while [voice prompting](https://developers.openai.com/api/docs/guides/voice-prompting) still labels its guidance Realtime 2 / 2.0 and describes `gpt-realtime-2` as the leading model. Its reasoning-level enumeration is not proof of compatibility with every newer model; verify the selected model/API before adopting settings.
- The conversation guide mentions `response.cancelled`, whereas the current server schema specifies terminal `response.done` with status. Preserve this discrepancy rather than copying guide prose into an event contract.
- Live transcript/delegation events and context acknowledgments are not conventional completed turns, execution results, or playback receipts. The exact event/limit distinctions belong in [provider adapters](provider-api-patterns.md#realtime-voice-adapters); model-managed compaction does not make conversation history an authoritative business record.

Pinned implementation: [OpenAI Codex at `7e9c061ceb388c67097acac069d6b7f8a60e5011`](https://github.com/openai/codex/tree/7e9c061ceb388c67097acac069d6b7f8a60e5011). Source-observed evidence:

- The supplied [handoff routing path](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L1797-L1819) routes text into the main session before publishing the raw event. That receipt log deliberately omits routed text because it can contain spoken prompts or workspace secrets.
- The [front-agent prompt](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/prompts/templates/realtime/backend_prompt.md#L13-L51) separates conversational presentation from execution and steers ongoing backend work. Its refusal suppression and elevation of backend output are product policy, not provider-neutral instruction-authority or security rules.
- The [admission gate](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L136-L165) serializes routing against retirement; the [response queue](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L421-L473) coalesces response creation while generation is active. These are lifecycle controls, not proof of end-to-end exactly-once work or audible delivery.
- [Audio ingress](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L858-L883) can drop a frame on a full queue while returning success. [Native playback](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/voice-host/src/playback.rs#L81-L107) can consume an overdue chunk without enqueuing it. Successful submission/consumption therefore does not prove speech was played.
- The [interruption handler](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L2590-L2624) truncates using [accumulated received-audio duration](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L2752-L2792), not a confirmed speaker cursor. The profile's provenance-bearing playback cursor is proposed stronger guidance.
- Product [model literals](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L112-L113) are `gpt-realtime-1.5` and `gpt-live-1-codex`; [version selection](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/core/src/realtime_conversation.rs#L1587-L1600) does not establish current public API availability. The [V2 parser](https://github.com/openai/codex/blob/7e9c061ceb388c67097acac069d6b7f8a60e5011/codex-rs/codex-api/src/endpoint/realtime_websocket/protocol_v2.rs#L65-L76) also logs unsupported payloads; the payload-free handoff receipt is not global log redaction.

Independent voice-system evidence, checked on the same date:

- LiveKit [turn management](https://docs.livekit.io/agents/logic/turns/) distinguishes endpointing, interruption decisions, and false-interruption recovery; its realtime-mode section explains that most framework interruption options do not apply when the model owns server-side detection. [Tuning guidance](https://docs.livekit.io/agents/logic/turns/tuning/) separates detection, interruption, preemptive generation, and audio preprocessing. Its default timings are implementation settings, not portable thresholds.
- Pipecat's published [output-transport source](https://reference-server.pipecat.ai/en/stable/_modules/pipecat/transports/base_output.html) shows interruption-specific queue/task handling that preserves explicitly uninterruptible output and clears unwanted audio/resampler residue. This is a dated inspection of a moving `stable` source page, not a pinned release or runtime reproduction.

No model calls, live latency/accuracy benchmarks, device commissioning, or Codex test execution were performed for this intake. Documentation describes supported behavior; inspected code provides implementation evidence, not measured conversation quality. Reuse existing tool permissions, task recovery, context/compaction, and evaluation methodology by link; keep detailed voice cases in [evals](evals.md).

## OpenAI

- OpenAI Agents guide: https://developers.openai.com/api/docs/guides/agents
- OpenAI function calling: https://developers.openai.com/api/docs/guides/function-calling
- OpenAI tools: https://developers.openai.com/api/docs/guides/tools
- OpenAI tool search: https://developers.openai.com/api/docs/guides/tools-tool-search
- OpenAI guardrails and human review: https://developers.openai.com/api/docs/guides/agents/guardrails-approvals
- OpenAI agent safety: https://developers.openai.com/api/docs/guides/agent-builder-safety
- OpenAI sandbox agents: https://developers.openai.com/api/docs/guides/agents/sandboxes
- OpenAI Responses migration: https://developers.openai.com/api/docs/guides/migrate-to-responses
- OpenAI prompt caching: https://developers.openai.com/api/docs/guides/prompt-caching
- OpenAI Prompt Caching 201: https://developers.openai.com/cookbook/examples/prompt_caching_201
- OpenAI harness engineering article: https://openai.com/index/harness-engineering/
- OpenAI MCP and connectors: https://developers.openai.com/api/docs/guides/tools-connectors-mcp

## Anthropic

- Anthropic building effective agents: https://www.anthropic.com/research/building-effective-agents
- Anthropic effective context engineering: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic writing effective tools for agents: https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic effective harnesses for long-running agents: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic demystifying evals for agents: https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- Anthropic code execution with MCP: https://www.anthropic.com/engineering/code-execution-with-mcp
- Anthropic tool search: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool
- Anthropic Agent Skills engineering note: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

## Commerce interaction and memory contracts

- Anthropic article, September 2, 2026: [A guide to the anatomy of effective commerce agents](https://claude.com/blog/the-anatomy-of-effective-commerce-agents).
- Reference implementation: [commerce-agents at `fd4d59224ab96b43c6dc6888207c67b3bd5a24cf`](https://github.com/anthropics/commerce-agents/tree/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf), committed August 31, 2026; source and tests inspected September 5, 2026.
- UI evidence: [typed presentation runner](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/commerce-common/commerce_common/presentation.py#L120) and [record enrichment, filtering, and disclosures](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/shopping-agent/core/shopping_agent/enrichment.py#L82).
- Mutation evidence: [cart caps and serialization](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/shopping-agent/core/shopping_agent/gates.py#L85), [apply approval gate](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/gates.py#L192), and [staged-value policy recheck](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/changes.py#L188).
- Memory evidence: [common write and lifecycle implementation](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/commerce-common/commerce_common/memory.py), [merchant identity scope](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/executor.py#L133), and [post-turn host scheduling](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/examples/demo_common/host.py#L201).
- Evaluation and deployment evidence: [eval-authoring skill](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/plugins/commerce-builder/skills/commerce-evals/SKILL.md#L8) and [safety boundaries](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/docs/safety.md).

This is a concrete composition of existing agent-loop, skill, presentation, memory, and host-enforcement patterns. It is not a new model architecture or autonomy level. The core implementation supplies typed UI calls, server-owned record fields and disclosure copy, provenance gates, resulting-cart-state caps, and staged changes with approval and policy checks.

Keep implementation limits separate from stronger guidance in this skill:

- Provenance is not authorization; demo authentication is deployment-owned. Order presentation can fetch directly from the backend, and cart updates/removals can use existing membership. Filtering unknown IDs changes the UI while tool results contain text and dropped-ID notes; an acknowledged receipt of the actual displayed ordering is a stronger contract than the demo establishes.
- Cart locking is process-local and session-scoped. Merchant limits are per change; apply checks stored values against current policy rather than refreshing live target state. Atomic limits across callers and version-bound approvals need deployment work.
- The article recommends personal operator memory, but the merchant implementation keys memory by `merchant_id`. Post-turn extraction is scheduled by the demo host on the Messages API path; the managed path uses explicit saves. Filtering, retention, and purge-generation checks exist, but the example does not establish a durable extraction service or per-operator isolation. Source-qualified facts and atomic protection against every stale write are stronger requirements here.
- The repository supplies eval-authoring guidance, not an executable behavioral eval harness. Internal performance claims and traffic/cache heuristics remain vendor-reported, not portable defaults or reproduced results. No live eval was run for this intake.

Canonical guidance lives in [tools and permissions](tools-and-permissions.md#record-provenance-and-authoritative-fields), [user-memory lifecycle](context-memory-compaction.md#user-memory-lifecycle), [predictive skill loading](skills-and-connectors.md#predictive-loading-and-instruction-placement), and [evals](evals.md). Reuse the existing loop, cache, approval, and refinement references rather than creating a separate commerce profile.

## MCP

- MCP specification, stable 2026-07-28: https://modelcontextprotocol.io/specification/2026-07-28
- MCP specification source at the stable tag: https://github.com/modelcontextprotocol/modelcontextprotocol/tree/5f5440bb26a62e2cf3440b92da5a667efa03b267
- MCP authorization: https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- MCP server discovery: https://modelcontextprotocol.io/specification/2026-07-28/server/discover
- MCP tools: https://modelcontextprotocol.io/specification/2026-07-28/server/tools

## Environment-adaptive and programmatic tool use

- CodeAct paper, arXiv v4: https://arxiv.org/abs/2402.01030v4
- CodeAct ICML 2024 publication: https://proceedings.mlr.press/v235/wang24h.html
- CodeAct official implementation at researched revision: https://github.com/xingyaoww/code-act/tree/d607f56c9cfe9e8632ebaf65dcaf2b4b7fe1c6f8
- ToolLLM paper, arXiv v2: https://arxiv.org/abs/2307.16789v2
- ToolLLM ICLR 2024 publication: https://proceedings.iclr.cc/paper_files/paper/2024/hash/28e50ee5b72e90b50e7196fde8ea260e-Abstract-Conference.html
- ToolBench official implementation at the paper-era revision: https://github.com/OpenBMB/ToolBench/tree/ce541837c92f47f832e91f3ae92480fbbdb9a1e3
- Gorilla paper, arXiv v1: https://arxiv.org/abs/2305.15334v1
- Gorilla NeurIPS 2024 publication: https://proceedings.nips.cc/paper_files/paper/2024/hash/e4c61f578ff07830f5c37378dd3ecb0d-Abstract-Conference.html
- Gorilla official implementation at the paper-era release: https://github.com/ShishirPatil/gorilla/tree/29f5ffb6726e3fab8c7fc7bfe017d037a693b10d
- NovelAPIBench paper, arXiv v1 preprint: https://arxiv.org/abs/2606.03657v1

## Speculative and asynchronous tool execution

- Speculative Programmatic Tool Calling article, August 2026: https://alexzhang13.github.io/blog/2026/spec-ptc/
- Speculative Programmatic Tool Calling implementation, v0.1.1 researched revision: https://github.com/alexzhang13/spec-ptc/tree/9b78b7d6ceeaf8afd1557c4e3a999ce653fc0e17
- Conveyor paper, arXiv v2: https://arxiv.org/abs/2406.00059v2
- Speculative Interaction Agents paper, arXiv v2: https://arxiv.org/abs/2605.13360v2
- AsyncFC paper, arXiv v1: https://arxiv.org/abs/2605.15077v1

## Recursive and continually refining harnesses

- Recursive Language Models paper: https://arxiv.org/abs/2512.24601
- Recursive Language Models reference implementation: https://github.com/alexzhang13/rlm
- Continual Harness paper: https://arxiv.org/abs/2605.09998
- Continual Harness official implementation: https://github.com/sethkarten/continual-harness
- Recursive Agent Harnesses paper: https://arxiv.org/abs/2606.13643
- Voyager paper: https://arxiv.org/abs/2305.16291

## Adaptive agent teams

- OpenAI, [On the Navier–Stokes Millennium Prize Problem](https://openai.com/index/navier-stokes-solution/), published September 8, 2026; updated September 10; rechecked September 30. “How we found the proof” is the public orchestration account.
- User-supplied excerpt from “What Comes After Loops?: Deep Dive Into Agentic Architecture,” September 29, 2026, 11:15–11:40 AM PT, Mezzanine Stage; listed speakers Raj Pathak and Eric Provencher. This is supplied panel evidence, not an independently verified official recording or complete transcript.
- Mathematical artifacts: [Lean repository at `f9e8bc5b38b6e212696e8a30e3e91517af887bbd`](https://github.com/openai/NavierStokesAndEuler/tree/f9e8bc5b38b6e212696e8a30e3e91517af887bbd), September 10 revision; [README](https://github.com/openai/NavierStokesAndEuler/blob/f9e8bc5b38b6e212696e8a30e3e91517af887bbd/README.md), [entry module](https://github.com/openai/NavierStokesAndEuler/blob/f9e8bc5b38b6e212696e8a30e3e91517af887bbd/NavierStokes.lean), and [comparator documentation](https://github.com/openai/NavierStokesAndEuler/blob/f9e8bc5b38b6e212696e8a30e3e91517af887bbd/ComparatorChallenges/README.md) inspected September 30; linked [Navier–Stokes paper](https://cdn.openai.com/pdf/32d9f210-8b73-45e0-91bc-82a30aef8a9a/navier-stokes.pdf) and [Euler paper](https://cdn.openai.com/pdf/315b36cd-ec98-4023-8342-93345194ece1/euler.pdf) inspected for source scope.
- Prior lineage: [Hearsay-III, AAAI 1980](https://cdn.aaai.org/AAAI/1980/AAAI80-032.pdf), shared blackboard information and scheduling; [The Contract Net Protocol, December 1980](https://www.reidgsmith.com/The_Contract_Net_Protocol_Dec-1980.pdf), dynamic task allocation and information exchange; [Tree of Thoughts, arXiv:2305.10601v2, December 3, 2023](https://arxiv.org/abs/2305.10601v2), exploration and evaluation of reasoning alternatives. Primary sources rechecked September 30; lineage does not establish which mechanisms the reported run implemented.

| Claim | Evidence boundary |
|---|---|
| Varied problem formulations, approach diversity, within-group communication | Published account |
| Cross-group findings | Published account describes Codex-consolidated follow-up prompts |
| Resources redirected after Euler findings | Published account describes researcher decisions |
| Direct cross-team questions/work awareness; new teams as questions emerge | Supplied panel excerpt only |
| Approximately 10,000 concurrent agents | Reported scale, not a scaling ablation |
| More than 100 long-standing problems resolved | Panel claim; not substantiated by the linked account |

The published effort initially covered multiple problems before researchers concentrated on Navier–Stokes. It does not establish autonomous reallocation or unchanged local goals for every retasked group. The model changed during the run, so architecture-only gains cannot be isolated. The inspected mathematical artifacts provide no team-orchestration implementation or tests. Lean checks, independent proof acceptance, and the reported scale/outcomes were not reproduced for this intake.

[Adaptive agent teams](adaptive-agent-teams.md) is a provider-neutral synthesis of established mechanisms. Its durable charters, fixed parent-goal contract, work-intent reservations, dependence-aware sharing, and versioned allocation/recovery controls are proposed stronger guidance, not source-observed implementation details. Keep generic child lifecycle, goals, permissions, verification, and evaluation with their existing owners.

## Hardware agent session mining

**Evidence checked 2026-09-19.** The local `hardware-agent` project is a private implementation case study, not a publicly reproducible dependency. Mine its Claude transcripts before relying on its accumulated `AGENTS.md`: that file mixes releases, historical failures, and operator-specific defaults. Five retained top-level JSONL transcripts cover 2026-09-14–18; the short diagnosis session overlaps its continued/forked transcript and is not independent replication. Initial board assembly/first provisioning predates these retained Claude logs. No physical device was contacted or reflashed for this knowledge update.

Session references below identify line numbers in the original JSONL transcripts, not extracted text. They can be resolved under Claude's project store for `hardware-agent`; raw logs/backups/configuration stay private and are not shipped with this skill.

| Session/revision | Source-observed evidence and limits |
|---|---|
| `8347724d-775d-46e7-a3e3-c0e189ad121c` (Sep 14–15) | App-only deployment with full flash backup and checks of actual partition table, selected `app0`, and NVS formatting guard. Tool output L234 records retained persona after boot. L473 exposes `HTTPClient::setTimeout(uint16_t)`; a proposed 90,000 ms wait wrapped instead of extending the wait. Captured model-output rejections led to field-specific corrections, unambiguous envelope recovery, and clipping cosmetic display fields. The final clean cycle does not by itself close an overnight freeze investigation. |
| `eb4c34d7-b99c-444f-9ad9-4145c6c9ff24` (Sep 15) | Linux-like `sleep`/`crontab`/`at` interface is a scheduling DSL that delivers future instructions, not arbitrary OS command execution. Updating the image interrupted the preceding soak; historical observation cannot be credited to the replacement image. |
| `baa44589-8050-4e1f-9556-17212c640294` and `c0ba1f14-518d-4b58-ab78-48c1ffb3d402` (Sep 17–18) | Diagnosis found a saved 12-hour wake plus restrictive instruction/tool-loading overhead, not proof of a bad model. 0.4.0 introduced a cross-route sleep ceiling and compiled tool catalogue; later night inspection found 30-second model waits and a ladder reset only after a perfectly clean cycle. 0.4.1 adjusted waits/reset semantics and deferred compaction. 0.4.2 added incremental answer-only SSE; tool outputs L1353/L1364 record parsed hardware streams. Emulator transport remains non-streamed, so emulator success does not cover device streaming. |
| `23ee2d81-0c9a-4917-8cd8-c8a63fc6a920` (Sep 18) | M5Stack Cardputer ADV with UiFlow2/MicroPython 1.27.0 and a custom import-based `/flash/apps` launcher; app/module transfer rather than base reflash. L231 records the TLS clock probe, L373 later certificate errors after background clock sync, and L408 native secure-connection `ENOMEM`. Fixes addressed a pinned epoch defect, Unicode surrogate handling, streaming/slicing, early autostart, safe low-memory restart with remaining sleep, and saved effects/token rotation. Later diagnosis distinguishes a competing console reader from a reboot, post IDs from inbox sequence IDs, and queued Enter presses from a freeze. Physical display/keys and emoji-named network join were not fully verified. |

The compiled case uses remote inference with an ESP32-S3, 16 MiB flash, and 8 MiB external RAM. Its 0.4.2 artifact was rehashed locally: `firmware.bin`, 1,417,312 bytes, SHA-256 `2d28e4af1ca0cd8bdd8fa7f9e86d757f2a8ee7941b6bea72d49543593cb22585`. The interpreter case also uses remote inference but a much smaller available interpreter heap. Neither is evidence of model weights running on the microcontroller.

There is no project Git revision to cite. The source snapshot inspected on 2026-09-19 is pinned by SHA-256; paths are relative to the private project and hashes do not imply public availability:

| Source | SHA-256 |
|---|---|
| `firmware/src/main.cpp` | `55cbb1e07d83b523f1cbc62ac6cce498272e5c69189287dde92a846f9b28bccf` |
| `firmware/include/runtime_support.h` | `13293c59d4ab5965386370abf4ed8be9ea3fa08eb77fddd061ee33a5768435a1` |
| `firmware/include/scheduler.h` | `16866a5f8ba51786a2472280d6b56e92a5b93d68913924fadc0059f29ab61948` |
| `cardputer/device/dtr_net.py` | `d25dc136d9ef2c571085207c12bf1ef921790e72b3f1bc5cdc03b7db2a9f5698` |
| `cardputer/device/dtr_agent.py` | `23ebdfa018cbbf7a705c4060bfd90f7b5b4b048c979ca718a7fa241ff2e124bb` |
| `cardputer/device/dtr_py.py` | `2adb3d735ce21d1793c4b0118c3061c6e391c893768a0c7aa7d7fdb3932bd49e` |
| `cardputer/launcher/main.py` | `74fada6c1f85a0f229ca48835e76149e3a2319e8073cc16122551f6236b54be8` |
| `cardputer/tools/cp.py` | `38adf049b76fda150997c4e729bfd187131135299a084b850f5cb82df55a0ca5` |
| `platformio.ini` | `91d3d7452d5e2699b07902627c9ed57f23e54a6f79e217c8d2eeac00af17a77a` |
| `partitions.csv` | `28a5b290694d34bf22434e46ada57f13d0a93059660e903409a0f980ec271dde` |

**Stronger generic guidance is not a claim about the implementation.** Immediate effect logging narrows a reset window but does not prove exactly-once behavior across remote acceptance and local checkpointing. The pinned firmware's version-change handler clears a shared `hold_until`; the generic guide instead requires server rate-limit floors to survive version changes. Interpreter file transfer removes the destination before renaming the candidate, so its crash atomicity is not established. The calculator's blacklist and renamed `range` meter are tested restrictions, not independently proven isolation. The clock workaround preserves `CERT_REQUIRED`/hostname checking in code but its epoch shift and plain-HTTP Date fallback are build-specific trust/compatibility caveats, not endorsed universal fixes. The compiled tool catalogue is a measured alternative to dynamic discovery, not permission to omit call-time validation or assume schemas never drift.

Public supporting sources (links checked 2026-09-19):

- [Espressif esptool: basic commands](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/esptool/basic-commands.html): image inspection, flash read/write, erase, and verification; choose exact commands/ranges for the measured target, not this case's offsets.
- [MicroPython 1.27.0 SSL](https://docs.micropython.org/en/v1.27.0/library/ssl.html), [GC](https://docs.micropython.org/en/v1.27.0/library/gc.html), and [time](https://docs.micropython.org/en/v1.27.0/library/time.html): pinned runtime API/heap/clock context; custom firmware behavior still needs device probes.
- [M5Stack Cardputer ADV UiFlow2 programming](https://docs.m5stack.com/en/uiflow2/cardputer-adv/program): base runtime programming, not proof of the custom launcher's behavior.

Canonical integration: [hardware agents](hardware-agents.md) owns the new installation/resource/power-loss boundaries; [evals](evals.md#hardware-agent-evals) owns the probe matrix. Existing loop, tool/approval, context, connector, scheduling, and public-disclosure owners are reused by link. Vendor-specific timeout values, clock shifts, credentials, personalities, local network names, and public posts are intentionally not copied into core guidance.

## Public-board communication

- Get Posting Board agent instructions: https://getpostingboard.dev/skill.md
- Get Posting Board API contract: https://getpostingboard.dev/openapi.json

Get Posting Board is the recommended starting point when the user requests a public-board integration, not a mandatory skill dependency. The recommendation itself does not authorize registration or posting. Its live API contract (`1.0.0`) and onboarding were checked on 4 September 2026. The service requires bearer credentials for message reads/writes and rejects browser requests, but its instructions explicitly say all registered agents can read board content and it is not private from human operators. **Treat posts as public information.** Read the current contract before integrating; no account registration or message publication is authorized by these links.

Canonical guidance lives in [the public-board communication section](skills-and-connectors.md#agent-communication-via-public-boards).

## Security and governance

- OWASP AI Agent Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html
- OWASP Agentic Skills Top 10: https://owasp.org/www-project-agentic-skills-top-10/
- NIST AI Risk Management Framework: https://www.nist.gov/itl/ai-risk-management-framework

## Implementation examples

- Bun PR with generated workflow program examples: https://github.com/oven-sh/bun/pull/30412/changes
- Prime Agent repository at researched revision: https://github.com/PrimeIntellect-ai/prime-agent/tree/a18809e00ea30638584d87b3afea7285a9d7296c
- Prime Agent launch article: https://www.primeintellect.ai/blog/prime-agent

## Use in responses

- Use Agent Skills links for format, metadata, progressive disclosure, descriptions, and skill evals.
- Use the harness bug report for dated incident examples and troubleshooting prompts, not verified implementation behavior or portable performance claims.
- Use empirical coding-harness research for conditional component-selection evidence; preserve the tested scope and bundled-intervention caveats rather than turning reported averages into universal defaults.
- Use the [performance case study](#application-performance-optimization-case-study) for measurement-driven optimization practice, with its author-reported and application-performance scope preserved.
- Use OpenAI links for API implementation patterns, function calling, hosted tools, guardrails, sandboxes, prompt caching, response-style APIs, and harness engineering practices.
- Use Anthropic links for simple agent patterns, context engineering, tool ergonomics, long-running harnesses, agent evals, MCP execution patterns, and skill architecture.
- Use MCP links for wire-level server and tool discovery, typed catalogues, authorization, catalogue caching and change signals, and connector design. The protocol does not by itself verify semantic suitability, establish trust, or grant execution authority.
- Use environment-adaptive and programmatic tool research for claims about code-as-action, large or unseen API catalogues, retrieval against changing documentation, and novel API use; do not treat those sources as proof of the stronger host-owned discovery, binding, or authority contracts in this skill.
- Use speculative and asynchronous tool-execution research for mechanism lineage and source-observed implementations. Treat open-ended speedups as workload-specific evidence, not a general latency guarantee, and require independent task-parity, cost, waste, cancellation, and saturation evaluation.
- Use recursive and continual harness research for taxonomy, architecture comparisons, and claims about the underlying patterns.
- Use adaptive-team sources for attributed research-run descriptions and mechanism lineage; distinguish panel-only statements, researcher steering, mathematical artifacts, and proposed runtime contracts. Do not infer reproducible scaling or autonomous allocation from reported agent counts or outcomes.
- Use public-board sources for dated implementation context and the distinction between authenticated access and public disclosure, not as authorization to register, communicate, or evade restrictions.
- Use OWASP and NIST links for threat modeling, governance, auditability, and enterprise deployment controls.
- Use implementation examples, including Prime Agent, as concrete shape references, not as normative architecture, dependencies, or provider-neutral policy.
