# Agent Harness Checklists

## MVP agent blueprint checklist

- [ ] Domain, primary user, and job-to-be-done are stated.
- [ ] MVP scope, assumptions, non-goals, and deferred capabilities are explicit.
- [ ] Autonomy level is the lowest level that still creates value.
- [ ] Core model-tool-observation loop is specified.
- [ ] Step, tool-call, time, token, and cost budgets are specified.
- [ ] Minimal typed tool registry is defined.
- [ ] Permission matrix covers read, draft, write, external, financial, destructive, and privileged actions.
- [ ] Risky actions use draft/commit separation.
- [ ] Planning mode blocks mutation until approval.
- [ ] Goal-like loop has objective, checkpoints, budget, validation, and stop rules.
- [ ] Context builder separates stable/cacheable content from volatile state.
- [ ] Memory, plans, approvals, todos, and artifacts are stored outside the prompt.
- [ ] Auto-compaction summary format and rehydration rules are defined.
- [ ] Skills are progressively disclosed and permission-bounded.
- [ ] MCP/external connectors are namespaced, scoped, and logged.
- [ ] Prompt caching and cost telemetry are included.
- [ ] Traces and evals are defined before launch.
- [ ] First rollout is limited, monitored, or shadow-mode.

## Coding-agent MVP checklist

Use the checklist in [coding-agents.md](coding-agents.md) for repository-facing coding agents. Keep this file as the general harness checklist index.

## Design checklist

- [ ] Domain and user persona defined.
- [ ] Autonomy level selected.
- [ ] Risk classes identified.
- [ ] Success and done conditions defined.
- [ ] Source-of-truth systems identified.
- [ ] Instruction hierarchy defined.
- [ ] Tool registry scoped to minimum viable tools.
- [ ] Permission matrix written.
- [ ] Draft/commit split defined for risky actions.
- [ ] Context builder designed.
- [ ] Memory and durable state plan defined.
- [ ] Compaction trigger and summary format defined.
- [ ] Planning mode criteria defined.
- [ ] Workflow orchestration criteria, packet shape, and verification strategy defined where needed.
- [ ] Goal loop criteria and budgets defined.
- [ ] Skills and connector strategy defined.
- [ ] Observability and eval plan defined.

## Tool checklist

For each tool:

- [ ] Name is specific and domain meaningful.
- [ ] Purpose says when to use and when not to use.
- [ ] Input schema is strict.
- [ ] Output schema is structured.
- [ ] Arguments are locally validated.
- [ ] Risk class assigned.
- [ ] Side effects declared.
- [ ] Permission policy assigned.
- [ ] Timeout set.
- [ ] Result size limit set.
- [ ] Retry policy set.
- [ ] Audit policy set.
- [ ] Errors return structured observations.
- [ ] Sensitive data is redacted.

## Permission checklist

- [ ] Read-only tools can run automatically only inside scope.
- [ ] Draft tools are separated from commit tools.
- [ ] External sends require approval.
- [ ] Financial actions require approval and strong auth.
- [ ] Destructive actions are denied or approval-gated with recovery plan.
- [ ] Identity/access changes require approval and strong auth.
- [ ] Shell/process execution is sandboxed.
- [ ] Connector tools are namespaced and scoped.
- [ ] Approval records are persisted.
- [ ] The model cannot approve its own actions.

## Environment-adaptive tools checklist

- [ ] A small stable bootstrap contract exists; the design does not assume literally zero environment knowledge.
- [ ] Discovery exposes only capabilities visible in the current user, tenant, task, and policy scope.
- [ ] Environment generation and catalogue version are recorded.
- [ ] Capability IDs are stable and namespaced; mutable names alone are not binding keys.
- [ ] Descriptors record origin, digest, implementation version, trust, and freshness.
- [ ] Declared, retrieved, reflected, and inferred schema evidence remain distinguishable.
- [ ] Inferred schemas cannot enable risky writes or establish side-effect safety.
- [ ] Probes are host-approved, bounded, read-only, dry-run, or isolated against disposable fixtures.
- [ ] Probe output, documentation, descriptions, examples, and errors are treated as untrusted data.
- [ ] Bindings include exact revision or digest, principal, tenant, resource scope, policy version, and expiry.
- [ ] A binding is neither a credential nor an approval; policy is rechecked for every invocation.
- [ ] Catalogue, schema, implementation, auth, policy, scope, and lease changes invalidate affected bindings.
- [ ] Timeouts and disconnects with possible side effects are reconciled before retry or rebinding.
- [ ] Programmatic composition can access external effects only through bound typed host capabilities.
- [ ] Generated helpers remain session-local and cannot register, persist, install, or expand authority automatically.
- [ ] Missing dependencies do not trigger automatic package, connector, or executable-skill installation.
- [ ] Compaction and recovery preserve evidence and binding status but re-resolve authority through the host.
- [ ] Evals cover held-out capabilities, poisoned descriptors, unsafe probes, drift, revocation, stale restore, substitution, and install confusion.

## Speculative tool execution checklist

- [ ] A measured serial and ordinary committed-parallel baseline exists.
- [ ] Speculation is feature-flagged, post-MVP, and justified by critical-path latency.
- [ ] Eligible tools are explicitly classified by the host; names and descriptions do not establish purity.
- [ ] Privacy, cost, rate, logging, cache, and shared-resource effects are included in eligibility.
- [ ] Writes, sends, payments, destructive actions, permission changes, and approval-gated calls cannot execute speculatively.
- [ ] Every physical dispatch resolves an exact binding, validates arguments, and passes permission policy first.
- [ ] Shadow state is disposable and cannot access credentials, approvals, ambient authority, or authoritative mutable objects.
- [ ] Uncertain syntax, arguments, dependencies, branches, or loops degrade to no speculation.
- [ ] Claim identity includes implementation, configuration, scope, policy/environment generation, snapshot, arguments, and occurrence.
- [ ] Deterministic reuse and stochastic occurrence handling are separate.
- [ ] A miss follows the normal committed path only when another physical attempt is replay-safe.
- [ ] The model receives one logical result per committed call while all physical attempts remain traceable.
- [ ] Speculation has hard concurrency, dispatch, time, token, cost, byte, rate, and waste budgets.
- [ ] Committed work has queue and capacity priority over speculative work.
- [ ] Eviction, cancellation request, cancellation confirmation, and completion after eviction are distinct states.
- [ ] Invalid code, abandoned turns, disconnects, and user cancellation clean up unclaimed candidates.
- [ ] Restarts do not restore serialized futures without executor reconciliation and policy revalidation.
- [ ] Evals cover parity, p50/p95 latency, hit/miss, waste, cost, cancellation, saturation, and ineligible effects.
- [ ] A monitored kill switch disables speculation when quality, latency, queue, waste, or throughput gates regress.

## Context checklist

- [ ] Trusted instructions separated from untrusted data.
- [ ] Scoped instructions loaded only when relevant.
- [ ] Retrieved content labeled by source and trust level.
- [ ] Exact facts preserved when needed.
- [ ] Large outputs summarized or stored externally.
- [ ] Active plan and goal reattached after compaction.
- [ ] Approval state reattached after compaction.
- [ ] Loaded skills and connector state tracked.
- [ ] Secrets are not placed in context.

## Planning checklist

- [ ] Planning mode exists for high-risk or ambiguous tasks.
- [ ] Mutation tools are blocked during planning.
- [ ] Plan artifact is stored outside prompt.
- [ ] Plan contains objective, scope, risks, steps, validation, rollback, and done condition.
- [ ] Approval tied to exact plan version.
- [ ] Execution uses todo/checkpoints after approval.

## Goal checklist

- [ ] Goal has one objective.
- [ ] Done condition is measurable.
- [ ] Budget is explicit.
- [ ] Validation method exists.
- [ ] Forbidden actions are listed.
- [ ] Approval-required actions are listed.
- [ ] Progress log is durable.
- [ ] Stop rules are explicit.

## Workflow orchestration checklist

- [ ] Workflow is justified by decomposition, broad coverage, parallel read-only work, verification needs, or resume requirements.
- [ ] Single-worker loop was considered first.
- [ ] Workflow artifact states objective, scope, success criteria, packet definitions, verification strategy, integration rules, and budgets.
- [ ] Generated orchestration program, if used, declares metadata, args, phases, schemas, prompt builders, scheduler logic, assertions, and final result shape.
- [ ] Approval binds to the exact workflow artifact version.
- [ ] Each packet has one purpose, explicit inputs, narrow tool permissions, output schema, timeout, budget, and evidence requirement.
- [ ] Worker contexts receive only packet-relevant context and tools.
- [ ] Risky side effects remain approval-gated and are not delegated to workers.
- [ ] Parallel execution is limited to independent, concurrency-safe work.
- [ ] Verifier contexts are independent enough to challenge findings.
- [ ] Assertions or gates catch null results, severe findings, missing quorum, scope drift, and exhausted budget.
- [ ] Integration rules cover deduplication, conflict resolution, confidence, coverage gaps, and failed packets.
- [ ] Workflow state is durable: plan, approvals, packet status, worker outputs, verifier outputs, integration notes, errors, and budget usage.
- [ ] Reproducibility state is captured: workflow version, model/runtime settings, tool calls, result references, source revision or data snapshot, and approval records.
- [ ] Final output distinguishes verified findings, rejected findings, unresolved questions, partial coverage, and next safe actions.

## Skills checklist

- [ ] Skill name matches directory name.
- [ ] Skill name is lowercase with hyphens only.
- [ ] `SKILL.md` has required frontmatter.
- [ ] Description says when to use the skill.
- [ ] Main instructions are concise.
- [ ] Detailed material is in focused Markdown references.
- [ ] References are loaded only when needed.
- [ ] Gotchas and validation steps are included.
- [ ] Skill activation eval exists.
- [ ] Output quality eval exists.
- [ ] Skill does not silently expand permissions.

## Self-refining recursive harness checklist

- [ ] The advanced profile is justified by measured baseline failures or gains and marked post-MVP.
- [ ] When strict external context is used, large inputs stay behind typed handles with bounded inspect, search, slice, and transform operations.
- [ ] The recursive unit is explicit: raw model, full harness, or another bounded worker type.
- [ ] Admission handles and terminal results are distinct, durable, and resumable.
- [ ] Depth, concurrency, step, token, cost, time, and retained-child limits apply to the whole recursion tree.
- [ ] Child authority is no broader than parent authority; permissions are enforced by immutable runtime policy.
- [ ] Mutable state is typed, versioned, scoped, provenance-labeled, and supplemental to the base policy.
- [ ] Base instructions, permission engine, credentials, budgets, evaluator, and audit controls are outside the mutable surface.
- [ ] Refinement proposals are structured diffs; trajectory content remains trust-labeled data.
- [ ] Every apply snapshots prior state, passes schema and policy checks, and occurs at a safe boundary.
- [ ] Every applied change runs a predefined probe and records an observed outcome.
- [ ] Regressions trigger automatic rollback or quarantine; promotion requires explicit evidence.
- [ ] Changes remain session-local by default; cross-session or global promotion has a separate gate.
- [ ] Executable skill changes receive sandbox, dependency, capability, provenance, and regression checks.
- [ ] Retained, daemon-backed, and scheduled runs handle cancellation, recovery, missed ticks, backpressure, idempotency, attribution, and garbage collection.
- [ ] Adversarial evals cover persistent prompt injection, reward hacking, authority escalation, cross-session leakage, and unbounded state growth.

## MCP/external connector checklist

- [ ] Servers/connectors inventoried.
- [ ] Tools namespaced by source.
- [ ] Credentials are per-user or scoped.
- [ ] Least privilege scopes used.
- [ ] Tool descriptions truncated or reviewed.
- [ ] External descriptions treated as untrusted.
- [ ] Risk classes mapped.
- [ ] Approval required for risky calls.
- [ ] Large results filtered before model context.
- [ ] Connector calls logged.
- [ ] Auth failure and revocation handled.

## Evals checklist

Use [evals.md](evals.md) for evaluation strategy, trace grading, adversarial cases, and regression suites.

- [ ] Happy-path tasks.
- [ ] Near-miss tasks.
- [ ] Prompt injection tasks.
- [ ] Tool misuse tasks.
- [ ] Approval bypass attempts.
- [ ] Connector failure tasks.
- [ ] Context overflow and compaction tasks.
- [ ] Conflicting instruction tasks.
- [ ] High-risk action tasks.
- [ ] Cost and latency measured.
- [ ] Regression evals added for every production incident.

## Minimal provider-neutral implementation path

1. Build a manual model-tool-observation loop.
2. Add strict tool schemas and local validation.
3. Add runtime permission checks.
4. Add structured tool results and error observations.
5. Add budgets and stop conditions.
6. Add tracing.
7. Add prompt-cache-aware context ordering and cache telemetry.
8. Add planning mode for high-risk tasks.
9. Add context compaction.
10. Add skills for reusable workflows.
11. Add MCP/external connectors with scoped permissions.
12. Add goal-like loops only after the base agent passes evals.
13. Add subagents or worker pools only when decomposition improves measured results.
14. Add recurring knowledge-base and entropy cleanup workflows.

## Agent legibility checklist

- [ ] Top-level instructions are a map, not a giant manual.
- [ ] Source-of-truth documents are indexed and retrievable.
- [ ] Active and completed plans are stored as durable artifacts.
- [ ] Domain schemas, policies, and runbooks are agent-readable.
- [ ] Validation signals are accessible through approved tools.
- [ ] Logs, metrics, traces, audit events, or workflow status are queryable where relevant.
- [ ] Human feedback is converted into docs, tools, validators, or evals.
- [ ] Stale docs and obsolete tools have a cleanup process.
- [ ] Quality scorecards or known-gap trackers exist for large systems.

## Prompt caching checklist

- [ ] Stable instructions appear before volatile runtime state.
- [ ] Tool definitions and schemas are sorted deterministically.
- [ ] Dynamic values such as timestamps and request IDs are placed near the end or omitted.
- [ ] Prompt and tool bundle versions are tracked.
- [ ] Provider cached-token fields are logged.
- [ ] Cache hit rate is monitored by session and tenant or segment.
- [ ] System prompt and tool-list hashes are logged to detect fragmentation.
- [ ] Compaction boundaries are explicit.
- [ ] Summaries are not rewritten every turn.
- [ ] Long-retention cache settings are used only when reuse justifies them.

## Mechanical invariant checklist

- [ ] Repeated prompt guidance has been converted into validators where possible.
- [ ] Validator errors include model-readable remediation instructions.
- [ ] Architecture or workflow boundaries are enforced mechanically.
- [ ] Secret/PII/source-citation checks exist where relevant.
- [ ] Cost, latency, and tool-result-size budgets are enforced.
- [ ] Regression evals are added after production incidents.
