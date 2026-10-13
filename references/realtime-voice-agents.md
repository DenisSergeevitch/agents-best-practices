# Realtime Voice Agents

## Scope and canonical ownership

Use this profile for agents that receive speech and produce spoken answers while input, output, or tool work can overlap. It owns media state, turn decisions, playback repair, and the boundary between conversation and execution. This is a composition of established mechanisms, not a new model, training method, or autonomy level. [Source evidence](source-links.md#realtime-voice-agents) distinguishes implemented behavior from the stronger provider-neutral contracts proposed here.

Voice can be a domain-specific MVP. Start with one useful scenario and a fixed narrow tool surface; push-to-talk is a valid baseline. Independent workers, durable orchestration, and speculative execution remain optional post-MVP additions unless the chosen architecture explicitly requires them. Do not infer universally weaker reasoning from audio capability; measure the selected model and task.

| Shared concern | Canonical owner |
|---|---|
| loop, stopping, retries, and aggregate budgets | [agentic loop](agentic-loop.md) |
| schemas, permissions, approval records, and apply-time state checks | [tools and permissions](tools-and-permissions.md), [approval records](security-observability.md#approval-records) |
| task ownership, cancellation scope, durable admission, and recovery | [owned work](always-on-agents.md#durable-tasks-and-owned-work), [accepted inputs](always-on-agents.md#accepted-inputs-and-control-lanes) |
| work packets or retained children, when selected | [workflows](workflow-orchestration.md), [child protocol](self-refining-recursive-harnesses.md#recursive-child-session-protocol) |
| instruction hierarchy, working state, and compaction | [instructions](system-prompts-instructions.md), [context and memory](context-memory-compaction.md) |
| credentials, untrusted content, recording retention, redaction, and trace storage | [security and observability](security-observability.md) |
| cache strategy and complete-task economics | [prompt caching and cost](prompt-caching-and-cost.md) |
| provider event syntax and model-specific capabilities | [voice adapters](provider-api-patterns.md#realtime-voice-adapters) |
| detailed fault cases, measurements, and launch evidence | [voice evals](evals.md#realtime-voice-agent-evals) |

## Architecture choices

Keep three choices independent:

| Axis | Choices and decision boundary |
|---|---|
| Media topology | Native speech-to-speech, or speech recognition → text agent → speech synthesis. Choose by task correctness, intermediate-text control, acoustic requirements, and measured audible latency. |
| Interaction | Explicit push-to-talk, automatic turns, or continuous full duplex. Listening while speaking does not imply that all overlapping speech is an interruption or an authorized command. |
| Execution placement | Tools and reasoning in the voice session, or an execution bridge to an existing backend. Duplex behavior alone does not require delegation. |

Streaming transcription or translation alone is not a task-executing voice agent. A text transcript also loses acoustic information: a backend deciding something about a sound may need the original authorized audio, rather than an assumed equivalent transcript.

A small voice layer can reuse a mature text harness. Keep its conversational prompt focused on speaking style, clarification, yielding, and the exact work/delegation boundary. Put detailed business procedures in the executor's existing instructions. Do not read internal plans or unrestricted tool payloads aloud. Acknowledgments and silence should serve the conversation; a control event need not trigger speech.

## Host state and evidence

The trusted host owns transport/device configuration, effective response policy, input admission, queues, playback state, execution authority, and correlation. The model proposes speech, interpretation, and work; it cannot establish that audio played or an external action committed.

Use typed records with the following voice-specific information. References to tasks and approvals point to the existing canonical records rather than creating a second task or permission system.

| Record | Required distinguishing fields |
|---|---|
| Session | Session epoch, effective configuration version, media format, transport, response-policy owner, lifecycle state |
| Input evidence | Provider item ID or host sequence, audio/transcript source, observed time range and clock domain, provisional/final status when available, current intent revision |
| Speech output | Response and segment IDs, associated intent/task revision, generated/queued/played position, playback-cursor provenance, output fence, terminal generation outcome |
| Execution bridge, if used | Existing operation/task ID, task revision, delegation/call ID, context watermark, admitted versus terminal outcome, result freshness |

If an API provides continuous transcript fragments without item IDs or completed-turn markers, generate host correlation and retain source offsets. Do not invent a provider completion event or treat a fragment gap as proof that the user finished. Recognition results and direct audio interpretation may disagree; clarify critical names, numbers, dates, and corrections when the evidence is insufficient for the proposed action. Background speech and transcripts retain their source/trust labels under the existing instruction and permission rules.

Separate **accepted context, model generation, transport delivery, client playback, and completed work**. An acknowledgment at one boundary proves only that boundary. Renderer receipts are the best available playback evidence, not proof of human understanding; label estimates when a transport cannot expose an accurate cursor. Do not derive heard text from generated bytes or assume precise audio/transcript alignment.

## Turn and response admission

Declare one effective owner for each decision: speech activity detection, input commitment, response creation, and interruption. These may be different components, but automatic and manual triggers must converge on one response-admission policy. Avoid enabling a framework detector whose settings are ignored while a provider owns detection.

For turn-based interaction, distinguish speech start/stop from intent commitment. Silence-based endpointing, semantic endpointing, and explicit release have different delay/cutoff tradeoffs. Tune on the target languages, microphones, and noise rather than copying a fixed silence threshold. Backchannels, hesitation, coughs, and another speaker require an explicit false-interruption/yield policy.

For continuous duplex interaction, admit work from sufficient current intent and the selected protocol's evidence; do not require a final transcript marker the protocol lacks. This does not authorize consequential execution from an ambiguous partial sentence. Reuse the existing approval and apply-time checks.

Response admission must correlate triggers with the current session epoch and intent revision. Serialize or coalesce competing requests when the provider permits only one foreground response; independent or out-of-band responses need distinct identities and explicit playback destinations. A completion from an obsolete response must not release new speech or overwrite current intent. Generation termination, input commitment, and output playback completion are separate transitions.

## Interruption and conversational repair

Treat an interruption as four explicit decisions, with separate outcomes:

| Boundary | Required transition |
|---|---|
| Playback | Stop local/relay output, clear or fence queued stale segments, and capture the best available played position. Ensure later chunks for that output cannot refill the queue. |
| Generation | Cancel, pause, or yield the foreground response according to the chosen protocol; observe its terminal outcome independently of playback silence. |
| Conversational history | Reconcile the unheard remainder through provider-supported truncation or explicit context repair. If exact alignment is unavailable, retain the uncertainty rather than fabricate a precise heard transcript. |
| Backend work | Explicitly continue, steer, or cancel the existing task using its cancellation policy. Yielding speech does not imply cancellation or rollback of an action. |

Apply this transition whether interruption originates in the model, host detector, push-to-talk control, or an explicit user stop. False interruption recovery may resume still-current speech only if its content, task revision, and output fence remain valid. An urgent correction must not wait behind a slow tool result or a full audio queue.

If the transport manages output buffering/truncation, verify its documented scope and observe the relevant events. If the client manages playback, the client owns stop and repair. A server-side buffer position is not independent evidence of physical client audibility. Neither canceling generation alone nor deleting history alone stops downstream queued audio.

## Execution bridge and corrections

A delegated voice request should carry a bounded, source-attributed context packet and the existing task identity. Include the current request/correction, intent revision, relevant conversation watermark, and references to authoritative state; use the [work-packet owner](workflow-orchestration.md) when actual worker orchestration is selected. A delegation notification may contain only an identity/offset, requiring the host to assemble the packet from its records.

Keep audio/control processing responsive while work runs. Admission means work was accepted, not completed. Backend updates should contain concise verified facts, current status, or a next step; each update declares whether it is context-only or eligible for speech. Context-only does not mean secret: any fact sent to the voice model may influence later output. Keep credentials and private backend reasoning outside the spoken context using the existing security boundary.

When the user corrects a request, advance the intent/task revision and invalidate obsolete confirmation eligibility before accepting old output. Bind both staged actions and speech eligibility to the current revision, using the existing [apply-time checks](tools-and-permissions.md#resulting-state-limits-and-apply-time-checks). Late results may remain evidence of work already performed; they cannot silently become the answer to the revised request. An already committed or uncertain effect needs reconciliation under the [retry owner](agentic-loop.md#retry-policy), not a claim that a voice interruption undid it.

For example, a user changes a requested appointment day while a lookup runs. The old lookup can settle, but its result cannot authorize booking the original day or support a spoken confirmation for the revised request. A previously committed booking must be checked and, if authorized, changed through the normal task workflow.

## Media budgets and output checks

Bound audio queues in **duration and age**, not only bytes or item counts. Declare sample format/rate/channels, cadence, resampling location, queue headroom, and overflow behavior. Observe missing input and dropped output explicitly; returning success after discarding a frame must not count as lossless delivery. Avoid unbounded buffering that preserves every sample while making conversation unusably late.

Keep control and correction handling outside blocking media/tool paths. Budget endpointing delay, useful response latency, interruption-to-silence, stall/silence timeouts, and pending backend work alongside the existing aggregate task budget. Decide which boundaries can drop media, which require backpressure, and which failures stop or downgrade the session. Acknowledgment latency is not useful-answer latency.

Model-managed audio history can compact or expire independently of transcripts. Preserve the current request, authoritative facts, permissions, and pending work through the existing [context owner](context-memory-compaction.md); do not treat a transcript's token count as the session's full audio-context usage. Use the existing cost owner for all attempts and modalities rather than a voice-only accounting system.

If the product requires checking speech before it is heard, the host must buffer it and correlate segment, transcript, check result, and playback eligibility. Missing, failed, or timed-out checks cannot release that segment. An interruption invalidates the relevant queued output and any late approval. A check running alongside immediately streamed speech cannot guarantee prevention; choose and measure that latency tradeoff explicitly. This barrier concerns speech release and does not replace tool authorization.

## Recovery and closure

Distinguish media transport, control-channel, provider conversation, and backend task lifetimes. Reconnecting a sideband does not prove the media survived, and opening a new voice session does not resume or cancel a task by itself. Fence obsolete audio and callbacks by session epoch; reconcile existing tasks/results before retrying an action or replaying a pending send. General durable recovery stays with [owned work and recovery](always-on-agents.md#durable-tasks-and-owned-work).

Retirement must prevent queued or already parsed old-session requests from admitting new work. Serialize retirement with admission and ensure late closure cannot retire a successor. Preserve required final transcript/effect evidence before reporting a settled close. Recording a transcript tail must not implicitly submit it as a new user command or trigger an extra action.

Declare graceful-close deadlines and forced-close outcomes. Keep required observers open while pending terminal events and usage drain; record incomplete finalization if the connection fails first. A voice-session close receipt does not prove background work has settled. The current task and approved effects remain authoritative outside reconstructable audio history.

## Observability and acceptance

Extend the existing [trace contract](security-observability.md#observability) with session/response/segment identities, intent/task revision, audio time and clock provenance, input commitment, response trigger, context/result submission, generated/delivered/played positions, interruption/fence events, queue age/loss, and terminal task outcomes. Store payloads and recordings only under that owner's retention and access policy; metadata-only correlation often suffices.

Use the [voice evaluation cases and measurements](evals.md#realtime-voice-agent-evals) and [mechanical voice checklist](checklists.md#realtime-voice-agent-checklist). Acceptance requires task/result correctness and audible interaction evidence independently. A plausible transcript, prompt change, lower generation latency, or passing source test alone does not establish usable spoken behavior.

## Conservative build sequence

1. Select media topology, interaction behavior, and execution placement for one scenario; establish a fixed-model, narrow-tool baseline with host permissions.
2. Implement correlation, turn ownership, bounded media queues, and interruption/history repair on the actual transport. Measure playback and resulting task state separately.
3. Tune automatic endpointing or duplex behavior using representative recordings and listening review. Keep a declared manual/failure fallback where appropriate.
4. Add an execution bridge when needed; verify correction, late-result, silence, and completion semantics before broadening its tools. Reuse existing task records and budgets.
5. Add durable recovery or speculative work only for measured needs or an explicitly selected architecture, using their canonical owners. Require matched voice evals before changing models, transport, or authority.

## Anti-patterns and unsupported claims

- Treating voice capability as a universal reasoning limit, or duplex as proof of delegated execution.
- Promoting a transcript fragment, audible acknowledgment, or context-injection receipt into completed/authorized work.
- Assuming received/generated/consumed audio was played, or that a terminal response event means the speaker is silent.
- Stopping generation while stale audio remains queued, or treating speech interruption as backend rollback.
- Applying both framework and provider turn rules without one effective admission owner.
- Speaking stale results after a correction or allowing a late guardrail approval to release canceled output.
- Replaying old speech or consequential work blindly after reconnect, or executing a saved transcript tail on close.
