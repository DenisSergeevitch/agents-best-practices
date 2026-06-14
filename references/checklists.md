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
