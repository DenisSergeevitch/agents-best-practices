# Agent Legibility and Feedback Loops

## Core principle

The harness should make the work legible to the agent. What the agent cannot inspect, retrieve, validate, or act on through approved tools is operationally absent from the agent's world.

This applies beyond coding:

```text
support agent: ticket history, customer state, escalation policy, response examples
finance agent: ledger state, approval policy, reconciliation rules, audit trail
legal agent: contract repository, clause library, jurisdiction rules, redline history
research agent: source corpus, extraction rubric, citation rules, review checklist
ops agent: incidents, runbooks, metrics, logs, service topology, rollback policy
sales agent: account plans, CRM state, product constraints, call notes, approval rules
```

Do not rely on tacit knowledge in chat threads, meetings, private documents, or people's heads. Encode durable knowledge into retrievable, versioned artifacts.

## Humans steer, agents execute

The strongest harnesses move humans up a level of abstraction. Humans should set priorities, acceptance criteria, boundaries, taste, and escalation policy. Agents should do bounded execution, gather evidence, validate outcomes, and surface judgment calls.

When the agent fails, do not only rewrite the prompt. Ask which missing component caused the failure:

```text
missing instruction
missing source of truth
missing tool
missing validator
missing permission rule
missing sandbox signal
missing eval
missing recovery path
```

Then encode the fix into the harness, documentation, tools, tests, or evaluations so the improvement compounds.

## Knowledge base as system of record

Use a short top-level instruction file as a map, not as an encyclopedia. The main instruction should point to deeper, structured references that are loaded only when needed.

Recommended layout for any domain:

```text
agent-instructions.md          # short map and rules
architecture.md                # domain model and major boundaries
policies/                      # authority, compliance, escalation, safety
runbooks/                      # operational procedures
plans/active/                  # current plans and execution logs
plans/completed/               # completed plans and decisions
references/                    # external or generated reference material
generated/                     # generated schemas, API inventories, catalogues
quality/                       # scorecards, known gaps, audit status
evals/                         # task fixtures and regression cases
```

Each document should have enough metadata to remain useful:

```text
owner
last_reviewed
scope
source_of_truth
verification_status
related_docs
known_staleness_risks
```

For late-bound environments, keep capability inventories explicit about origin, environment and catalogue version, descriptor digest, validation evidence, binding status, and expiry. Use [environment-adaptive tools](environment-adaptive-tools.md) for the lifecycle; this file owns only making those artifacts discoverable and fresh.

Version plans, decisions, quality reports, and generated references where possible. A durable local artifact is easier for an agent to discover and reuse than a prior chat discussion.

## Agent-legible environment

A mature harness exposes the environment through approved tools:

```text
read source-of-truth records
search policies and documentation
query logs, metrics, traces, or audit events
inspect current workflow state
capture screenshots or structured UI state when relevant
run validation checks
produce evidence artifacts
compare before/after state
```

For each domain, define the signals that prove progress. Examples:

```text
support: customer reply drafted, policy citations present, PII redacted
finance: ledger balances reconcile, approval attached, audit event written
legal: clause changes mapped to source request, risk flags reviewed
ops: incident mitigated, metric recovered, postmortem draft created
research: sources screened, extraction table complete, citations verified
sales: account brief prepared, risks ranked, next steps approved
```

The agent should be able to validate its work using these signals without relying on a human to manually copy data into the prompt.

## Mechanical invariants beat prompt advice

Documentation alone does not keep an agentic system coherent. Convert recurring guidance into mechanical checks.

Examples:

```text
schema validators
policy checkers
lint rules
structural tests
approval matrix checks
PII and secret scanners
source-citation validators
freshness checks
workflow-state validators
cost and latency budgets
regression evals
```

A useful pattern is to give validators remediation messages that are safe to show to the model:

```text
Violation: External customer email has no approval record.
Fix: Call request_approval with action_type="external_send" and include the email preview.
```

Centralize boundaries and correctness rules. Allow local autonomy only inside those boundaries.

## Feedback loops

Treat every run as a feedback opportunity.

Standard loop:

```text
1. Validate current state.
2. Gather source-of-truth context.
3. Produce a plan or action proposal.
4. Execute only within permission policy.
5. Validate result against objective.
6. Capture proof or evidence.
7. Record progress, failures, and decisions.
8. Feed recurring issues into docs, tools, policies, or evals.
```

For high-throughput systems, human attention becomes the scarce resource. Automate low-risk review and reserve humans for judgment, policy exceptions, high-risk commits, and unresolved ambiguity.

In an advanced post-MVP harness, repeated human corrections and trace or eval failures may feed a separate automated refiner. Keep that loop gated: it proposes a versioned supplemental-state change, the host checks scope and immutable policy, held-out evaluation measures the outcome, and failed changes are quarantined or rolled back. A model's self-reported improvement is never sufficient proof. See [self-refining recursive harnesses](self-refining-recursive-harnesses.md); the standard execution feedback loop above remains the default.

## Throughput and merge philosophy, generalized

When agents can produce many candidate artifacts, the operating model should change from manual inspection of every detail to risk-tiered validation.

Use:

```text
low risk: automated validation and sampling
medium risk: automated validation plus targeted human review
high risk: explicit human approval before commit
regulated/destructive: approval, audit, rollback, and post-action verification
```

This principle applies outside software. For example, a support agent may auto-draft many replies but require approval before sending certain categories; a finance agent may auto-prepare adjustments but require approval before posting; a research agent may auto-screen papers but require reviewer confirmation before final conclusions.

## Entropy and garbage collection

Agents replicate existing patterns, including bad ones. Without cleanup, stale rules, mediocre examples, and weak abstractions compound.

Run recurring garbage-collection workflows:

```text
scan for stale documentation
identify repeated tool failures
find low-quality examples that agents imitate
remove unused tools and skills
update quality scorecards
merge duplicate instructions
retire obsolete workflows
convert repeated review comments into checks
refresh source-of-truth indexes
```

Maintain a small set of golden principles. Make them enforceable wherever possible.

## Design rule

A strong harness is not only a prompt and tools. It is an agent-legible operating environment with feedback loops, validators, source-of-truth documents, and recurring cleanup.
