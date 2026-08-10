# Security and Observability

Use this reference for threat modeling, guardrails, approval records, trace design, launch safety gates, and incident response. Use [evals.md](evals.md) for evaluation strategy, adversarial test suites, trace grading, regression evals, and eval-driven launch criteria.

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
