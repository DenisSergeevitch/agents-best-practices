# Coverage Audit

This file maps the required agent-harness knowledge areas to the Markdown files in this skill. Use it to confirm that no major design topic was omitted before applying the skill to a real agent architecture.

## Required coverage map

| Topic | Covered in | Notes |
|---|---|---|
| General-purpose agent harness | `SKILL.md`, `architecture.md` | Treats coding as one domain among many. |
| Coding-agent harness overlay | `coding-agents.md`, `mvp-agent-blueprint.md`, `tools-and-permissions.md`, `security-observability.md`, `evals.md`, `checklists.md` | Optional domain overlay for repository-reading, patching, validating, reviewing, migration, dependency, test, and docs-sync agents. |
| Agent-legible environment and feedback loops | `agent-legibility-feedback-loops.md`, `architecture.md` | Covers source-of-truth knowledge bases, validation signals, mechanical invariants, throughput, and entropy cleanup. |
| Agentic loop | `agentic-loop.md` | Includes canonical loop, invariants, budgets, retries, provider-neutral variants, and termination. |
| Goal-like loop | `agentic-loop.md`, `planning-and-goals.md` | Includes objective, done condition, budget, checkpoints, progress log, validation, and stop rules. |
| Planning mode | `planning-and-goals.md` | Covers read-only planning, plan artifact, approval, execution after approval, and plan-validate-execute. |
| Workflow orchestration | `workflow-orchestration.md`, `architecture.md`, `planning-and-goals.md`, `checklists.md` | Covers planner-generated workflows, work packets, worker and verifier contexts, integration, durable workflow state, budgets, approvals, and anti-patterns. |
| Programmable context and recursive execution | `self-refining-recursive-harnesses.md`, `context-memory-compaction.md`, `workflow-orchestration.md`, `planning-and-goals.md` | Distinguishes strict prompt-as-variable processing, code-first context access, raw-model recursion, and full-harness recursion; covers aggregate tree budgets and retained-child contracts. |
| Continual harness refinement | `self-refining-recursive-harnesses.md`, `agent-legibility-feedback-loops.md`, `security-observability.md`, `evals.md`, `checklists.md` | Covers typed supplemental state, immutable policy boundaries, structured proposals, observed validation, rollback, quarantine, and local-to-global promotion. |
| Executable skills and learned artifacts | `self-refining-recursive-harnesses.md`, `skills-and-connectors.md`, `tools-and-permissions.md`, `security-observability.md`, `evals.md` | Separates descriptive skill knowledge from executable artifacts and covers provenance, sandboxing, review, promotion, and regression tests. |
| Retained, daemon-backed, and scheduled lifecycle | `self-refining-recursive-harnesses.md`, `planning-and-goals.md`, `workflow-orchestration.md`, `security-observability.md` | Covers durable handles, recovery, cancellation, missed schedules, backpressure, idempotency, attribution, and garbage collection. |
| Auto context and compaction | `context-memory-compaction.md` | Covers context tiers, scoped instruction loading, retrieval, compaction triggers, handoff summaries, and rehydration. |
| Prompt caching and cost control | `prompt-caching-and-cost.md`, `context-memory-compaction.md`, `provider-api-patterns.md` | Covers stable-prefix design, deterministic serialization, provider cache fields, TTL/retention notes, compaction/cache tradeoffs, and monitoring. |
| Skills attachment | `skills-and-connectors.md`, `SKILL.md` | Covers Agent Skills structure, progressive disclosure, trigger descriptions, governance, and evals. |
| MCP and external connectors | `skills-and-connectors.md` | Covers resources/prompts/tools, staged loading, namespacing, authorization, deferred tool loading, and code-execution patterns. |
| Environment-adaptive tools | `environment-adaptive-tools.md`, `tools-and-permissions.md`, `skills-and-connectors.md`, `evals.md` | Covers stable bootstrap interfaces, capability provenance, schema verification, bounded probes, exact runtime bindings, programmatic composition, drift invalidation, and focused evals without turning discovery into authority. |
| Speculative tool execution | `speculative-tool-execution.md`, `agentic-loop.md`, `tools-and-permissions.md`, `evals.md`, `checklists.md` | Covers experimental prelaunch from partial programs, exact claim-or-run identity, disposable shadow state, multiplicity, waste budgets, cancellation, safe fallback, and parity evaluation. |
| System prompts and instructions | `system-prompts-instructions.md` | Covers authority hierarchy, runtime reminders, injection boundaries, and prompt templates. |
| Tool design | `tools-and-permissions.md`, `environment-adaptive-tools.md` | Covers fixed and late-bound tool contracts, schemas, risk taxonomy, structured outputs, result limits, errors, sandboxing, secrets, visibility, verification, binding, and invalidation. |
| Permissions and approvals | `tools-and-permissions.md`, `security-observability.md` | Covers permission matrix, draft/commit split, approval records, and policy enforcement. |
| Provider API differences | `provider-api-patterns.md` | Covers OpenAI Responses-style APIs, Chat Completions-style/OpenAI-compatible APIs, Anthropic APIs, hosted tools, adapters, streaming, and state. |
| Security | `security-observability.md` | Covers threat model, guardrails, prompt injection, approvals, launch gates, and incidents. |
| Observability | `security-observability.md` | Covers traces, events, token/cost/latency, and replay. |
| Evals | `evals.md`, `coding-agents.md`, `checklists.md` | Covers task success, tool precision, adversarial tests, trace grading, regression tests, and eval-driven launch criteria. |
| Implementation checklist | `checklists.md` | Includes design, tool, permission, context, planning, goal, skill, connector, eval, and rollout checklists. |

## Required language and scope checks

- The skill is provider-neutral.
- The skill refers to OpenAI, Anthropic, and OpenAI-compatible APIs only where provider-specific API patterns matter.
- The skill does not depend on coding-agent-specific assumptions; the coding-agent MVP profile is an optional domain overlay.
- The skill contains only Markdown files.
- The skill includes prompt-cache architecture and cache-hit monitoring.
- The skill includes agent-legibility, knowledge-base, feedback-loop, and entropy-management practices.
- The skill includes workflow orchestration as a generic harness pattern without depending on a vendor-specific runtime.
- The skill treats recursive execution and continual refinement as advanced, post-MVP profiles that require measured justification.
- Mutable harness state cannot expand base authority, permissions, credentials, budgets, or evaluation policy.
- Runtime capability discovery, probing, schema inference, binding, and generated helpers cannot create or expand authority.
- Partial model output cannot authorize speculative execution; every physical dispatch and later claim remain host-validated and policy-bound.
- The skill uses progressive disclosure: `SKILL.md` is the entry point; detailed guidance is in focused reference files.

## Minimum file set

```text
agents-best-practices/
  SKILL.md
  references/
    architecture.md
    agent-legibility-feedback-loops.md
    coding-agents.md
    agentic-loop.md
    tools-and-permissions.md
    environment-adaptive-tools.md
    speculative-tool-execution.md
    workflow-orchestration.md
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
