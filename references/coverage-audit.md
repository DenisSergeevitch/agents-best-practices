# Coverage Audit

This file maps the required agent-harness knowledge areas to the Markdown files in this skill. Use it to confirm that no major design topic was omitted before applying the skill to a real agent architecture.

## Required coverage map

| Topic | Covered in | Notes |
|---|---|---|
| General-purpose agent harness | `SKILL.md`, `architecture.md` | Treats coding as one domain among many. |
| Realtime voice agents | [voice profile](realtime-voice-agents.md), [voice evals](evals.md#realtime-voice-agent-evals), `provider-api-patterns.md`, `checklists.md` | Owns independent media/interaction/execution choices, input and playback evidence, turn admission, interruption repair, correction freshness, media budgets, and voice-specific recovery boundaries; reuses permissions, task lifecycle, compaction, and general eval methodology. Voice can be an MVP without broader orchestration. |
| Coding-agent harness overlay | `coding-agents.md`, `mvp-agent-blueprint.md`, `tools-and-permissions.md`, `security-observability.md`, `evals.md`, `checklists.md` | Optional domain overlay for repository-reading, patching, validating, reviewing, migration, dependency, test, and docs-sync agents; includes measured post-MVP action-interface selection without relaxing host controls. |
| Agent-legible environment and feedback loops | `agent-legibility-feedback-loops.md`, `architecture.md` | Covers source-of-truth knowledge bases, validation signals, mechanical invariants, throughput, and entropy cleanup. |
| Agentic loop | `agentic-loop.md` | Includes canonical loop, invariants, budgets, retries, provider-neutral variants, termination, and optional per-request routing with selected/dispatched identity, target compatibility, and branch-state semantics. |
| Runtime instruction and tool configuration | [architecture.md](architecture.md#runtime-instruction-and-tool-configuration-events), `system-prompts-instructions.md`, `evals.md`, `checklists.md` | Covers authorized ordered deltas, effective versions, declaration/execution boundaries, replay/checkpoints, provider projection, revocation, and removal versus historical erasure. |
| Goal-like loop | `agentic-loop.md`, `planning-and-goals.md` | Includes objective, done condition, budget, checkpoints, progress log, validation, and stop rules. |
| Always-on agents and durable runtime | [always-on-agents.md](always-on-agents.md#taxonomy-and-boundaries), `architecture.md`, `planning-and-goals.md` | Separates availability, durability, inference activity, and authority; owns general resident leases, scheduled wakeups, and restart recovery without requiring recursive execution or refinement. |
| Durable submissions, tasks, and application documents | [commit boundary](always-on-agents.md#records-and-atomic-publication), [input/control lanes](always-on-agents.md#accepted-inputs-and-control-lanes), [owned work](always-on-agents.md#durable-tasks-and-owned-work), [documents/forks](always-on-agents.md#application-documents-and-forks) | Covers accepted-input deduplication versus completed work, settled local publication, owned foreground/background task boundaries, and document scope/lifetime versus historical/current/fresh fork policy; links external-effect retry and permission owners. |
| Observer reconnection and projection | [observation and reconnection](always-on-agents.md#observation-and-reconnection), `security-observability.md` | Distinguishes snapshot/live convergence and bounded overflow recovery from durable audit replay; observation cursors do not prove committed history or exactly-once delivery. |
| Recurring collection and delivery bookkeeping | [collection contract](always-on-agents.md#recurring-collection-and-delivery-bookkeeping), [failure probes](evals.md#always-on-and-durable-runtime-evals), `checklists.md` | Covers per-source collection checkpoints, explicit incomplete-source coverage, and receipt-backed delivery progress while reusing local commit, effect reconciliation, and permission owners. |
| Planning mode | `planning-and-goals.md` | Covers read-only planning, plan artifact, approval, execution after approval, and plan-validate-execute; distinguishes execution-time progress scaffolds and their stopping effects from permission mode. |
| Workflow orchestration | `workflow-orchestration.md`, `architecture.md`, `planning-and-goals.md`, `checklists.md` | Covers planner-generated workflows, work packets, worker and verifier contexts, integration, durable workflow state, budgets, approvals, and anti-patterns. |
| Adaptive agent teams | [adaptive-agent-teams.md](adaptive-agent-teams.md), [team evals](evals.md#adaptive-agent-team-evals), `checklists.md` | Covers approach portfolios, discoverable work intentions, overlap resolution and declared replication, communication dependence, evidence-linked allocation changes, and stale-result attribution; reuses goal, workflow, and child-lifecycle owners. |
| Programmable context and recursive execution | `self-refining-recursive-harnesses.md`, `context-memory-compaction.md`, `workflow-orchestration.md`, `planning-and-goals.md` | Distinguishes strict prompt-as-variable processing, code-first context access, raw-model recursion, and full-harness recursion; covers aggregate tree budgets and retained-child contracts. |
| Hardware agents and board deployment | `hardware-agents.md`, `evals.md`, `checklists.md`, `source-links.md` | Covers inference location, board/runtime inventory, launcher versus compiled installation, partition/boot selection, bounded transport and native TLS memory, power-loss state, clock/wake contracts, retained identity, rollback, and physical-versus-host evidence. |
| Continual harness refinement | `self-refining-recursive-harnesses.md`, `agent-legibility-feedback-loops.md`, `security-observability.md`, `evals.md`, `checklists.md` | Covers typed supplemental state, immutable policy boundaries, structured proposals, observed validation, rollback, quarantine, and local-to-global promotion. |
| Executable skills and learned artifacts | `self-refining-recursive-harnesses.md`, `skills-and-connectors.md`, `tools-and-permissions.md`, `security-observability.md`, `evals.md` | Separates descriptive skill knowledge from executable artifacts and covers provenance, sandboxing, review, promotion, and regression tests. |
| Retained child-session lifecycle | `self-refining-recursive-harnesses.md`, `workflow-orchestration.md`, `security-observability.md` | Owns child admission handles, terminal records, message delivery, cancellation propagation, aggregate budgets, and child recovery; general resident ownership remains in `always-on-agents.md`. |
| Auto context and compaction | `context-memory-compaction.md` | Covers context tiers, scoped instruction loading, retrieval, staged elision-before-summary, input headroom, protocol preservation, optional historical-output recall, handoff summaries, rehydration, and safe publication of fixed-prefix background compaction. |
| User-memory lifecycle | `context-memory-compaction.md`, `evals.md` | Covers eligible fact sources, person/tenant scope, optional bounded background extraction, corrections, retention, deletion races, and layered reads. |
| Prompt caching and cost control | `prompt-caching-and-cost.md`, `context-memory-compaction.md`, `provider-api-patterns.md`, `evals.md` | Covers stable-prefix design, deterministic serialization, provider cache fields, TTL/retention notes, compaction/cache tradeoffs, monitoring, and optional warming eligibility, economics, cancellation, and full outcome accounting. |
| Skills attachment | `skills-and-connectors.md`, `SKILL.md` | Covers Agent Skills structure, progressive disclosure, measured predictive loading, trigger descriptions, governance, and evals. This skill's [freshness and self-update contract](../SKILL.md#freshness-and-self-update) defines canonical-source checks, revision-consistent refresh, local-change preservation, and unverified-freshness disclosure. |
| MCP and external connectors | `skills-and-connectors.md` | Covers resources/prompts/tools, staged loading, namespacing, authorization, deferred tool loading, and code-execution patterns. |
| Public-board agent communication | [skills-and-connectors.md](skills-and-connectors.md#agent-communication-via-public-boards), `checklists.md` | Makes PUBLIC INFORMATION visible to the model and user; covers destination/content approval, outbound query/profile disclosure, private-context exclusion, untrusted peers, and recovery without treating API access as confidentiality. |
| Environment-adaptive tools | `environment-adaptive-tools.md`, `tools-and-permissions.md`, `skills-and-connectors.md`, `evals.md` | Covers stable bootstrap interfaces, capability provenance, schema verification, bounded probes, exact runtime bindings, programmatic composition, drift invalidation, and focused evals without turning discovery into authority. |
| Speculative tool execution | `speculative-tool-execution.md`, `agentic-loop.md`, `tools-and-permissions.md`, `evals.md`, `checklists.md` | Covers experimental prelaunch from partial programs, exact claim-or-run identity, disposable shadow state, multiplicity, waste budgets, cancellation, safe fallback, and parity evaluation. |
| System prompts and instructions | `system-prompts-instructions.md` | Covers authority hierarchy, runtime reminders, injection boundaries, and prompt templates. |
| Tool design | `tools-and-permissions.md`, `environment-adaptive-tools.md` | Covers fixed and late-bound tool contracts, schemas, risk taxonomy, structured outputs, result limits, errors, sandboxing, secrets, visibility, verification, binding, and invalidation. |
| Record provenance and interactive presentation | [tools-and-permissions.md](tools-and-permissions.md#presentation-tools-and-rendered-state), `evals.md` | Covers authorized record introduction, authoritative fields, ordered render receipts, provisional UI, and follow-up references after filtering or reordering. |
| Resulting-state business limits | [tools-and-permissions.md](tools-and-permissions.md#resulting-state-limits-and-apply-time-checks), `evals.md` | Covers cumulative limits at the shared resource scope, atomic mutations, stale stages, and current-policy/live-state checks at apply. |
| Permissions and approvals | `tools-and-permissions.md`, `security-observability.md` | Covers permission matrix, draft/commit split, approval records, and policy enforcement. |
| Provider API differences | `provider-api-patterns.md` | Covers OpenAI Responses-style APIs, Chat Completions-style/OpenAI-compatible APIs, Anthropic APIs, hosted tools, adapters, streaming, and state. |
| Security | `security-observability.md` | Covers threat model, guardrails, prompt injection, approvals, launch gates, and incidents. |
| Observability | `security-observability.md` | Covers traces, events, token/cost/latency, and replay. |
| Runtime troubleshooting | [security-observability.md](security-observability.md#troubleshooting), `SKILL.md`, `checklists.md` | Routes symptoms to evidence and isolated probes for streams, completion, recovery races, registrations, approvals, deadlocks, context/cost accounting, payload limits, and telemetry overhead; reuses existing mechanism and eval owners. |
| Evals | `evals.md`, `coding-agents.md`, `checklists.md` | Covers runtime-state fixtures, paired and cross-capability cases, outcome and safety trace grading, failure-aware component diagnostics across context budgets, physical mutation versus judged-phase evidence, model/configuration sweeps, task economics, and regression launch gates. |
| Performance optimization | [evals.md](evals.md#performance-optimization-and-regression-ratchets), `checklists.md`, `source-links.md` | Covers scoped timing provenance, proxy acceptance, versioned regression ratchets, evaluator-change review, and field-confirmed gains while reusing existing latency, trace, and feedback contracts. |
| Implementation checklist | `checklists.md` | Includes design, tool, permission, context, planning, goal, skill, connector, eval, and rollout checklists. |

## Required language and scope checks

- The skill is provider-neutral.
- The skill refers to OpenAI, Anthropic, and OpenAI-compatible APIs only where provider-specific API patterns matter.
- The skill does not depend on coding-agent-specific assumptions; the coding-agent MVP profile is an optional domain overlay.
- The skill contains only Markdown files.
- The skill includes prompt-cache architecture and cache-hit monitoring.
- Per-request routing and paid cache warming remain post-MVP optimizations; runtime configuration changes cannot grant authority or erase required history implicitly.
- The skill includes agent-legibility, knowledge-base, feedback-loop, and entropy-management practices.
- The skill includes workflow orchestration as a generic harness pattern without depending on a vendor-specific runtime.
- The skill treats recursive execution and continual refinement as advanced, post-MVP profiles that require measured justification.
- Always-on and durable runtimes remain post-MVP lifecycle profiles, independent of recursion/refinement and autonomy level; local persistence or single-owner execution does not establish high availability or exactly-once external effects.
- Adaptive agent teams remain a post-MVP architecture composition with measured simpler baselines; agent count, consensus, and published outcomes do not establish independent acceptance or expanded authority.
- Mutable harness state cannot expand base authority, permissions, credentials, budgets, or evaluation policy.
- Runtime capability discovery, probing, schema inference, binding, and generated helpers cannot create or expand authority.
- Partial model output cannot authorize speculative execution; every physical dispatch and later claim remain host-validated and policy-bound.
- Public-board communication is opt-in external publication, not private memory; the model-visible public-audience label survives compaction and does not substitute for runtime approval.
- The skill uses progressive disclosure: `SKILL.md` is the entry point; detailed guidance is in focused reference files.
- Hardware installation requires the measured target/boot path and preserved-state boundary; host or emulator checks do not prove physical TLS, UI, or unattended uptime.

## Minimum file set

```text
agents-best-practices/
  SKILL.md
  references/
    architecture.md
    always-on-agents.md
    agent-legibility-feedback-loops.md
    coding-agents.md
    agentic-loop.md
    tools-and-permissions.md
    hardware-agents.md
    environment-adaptive-tools.md
    speculative-tool-execution.md
    workflow-orchestration.md
    adaptive-agent-teams.md
    self-refining-recursive-harnesses.md
    context-memory-compaction.md
    prompt-caching-and-cost.md
    planning-and-goals.md
    skills-and-connectors.md
    system-prompts-instructions.md
    provider-api-patterns.md
    security-observability.md
    evals.md
    checklists.md
    coverage-audit.md
```


## MVP agent blueprint generation

Covered in `SKILL.md`, `references/mvp-agent-blueprint.md`, `references/architecture.md`, and `references/checklists.md`. The skill now explicitly instructs assistants to produce a domain-specific MVP harness blueprint when the user asks to make or build an agent. The blueprint includes agentic loop, tool registry, permissions, context and memory, auto-compaction, planning mode, goal-like loop criteria, skills, MCP/external connectors, prompt caching, cost-aware context, observability, dedicated eval strategy, and launch path.
