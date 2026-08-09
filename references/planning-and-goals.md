# Planning and Goals

## Planning mode

Planning mode is a runtime mode, not just a paragraph in a prompt.

Planning mode allows:

- reading;
- searching;
- asking clarifying questions;
- comparing approaches;
- drafting a plan artifact;
- estimating risks and validation steps.

Planning mode blocks:

- writes;
- sends;
- deletes;
- payments;
- permission changes;
- deployments;
- external commitments;
- other irreversible side effects.

Use planning mode for non-trivial, ambiguous, multi-step, high-impact, or high-risk tasks.

## When to enter planning mode

Enter planning mode when:

- more than one valid strategy exists;
- the work touches multiple systems or stakeholders;
- side effects are hard to undo;
- user preferences materially affect the outcome;
- the domain is regulated or high stakes;
- tool execution is expensive;
- validation criteria are unclear;
- the task will likely exceed one context window.

Do not enter planning mode for simple read-only questions or obvious single-step actions.

## Plan artifact

Store the plan outside the prompt as a durable artifact.

Plan format:

```markdown
# Plan: [objective]

## Objective
...

## Scope
Included:
- ...

Excluded:
- ...

## Assumptions
- ...

## Risks
- ...

## Steps
1. ...
2. ...
3. ...

## Tools required
- ...

## Approval points
- ...

## Validation
- ...

## Rollback or recovery
- ...

## Done condition
- ...
```

## Plan approval

Before executing risky steps, request approval with:

```text
summary of plan
exact actions requiring approval
risk class
expected outcome
rollback or recovery path
scope of approval
expiration or budget
```

Approval should be tied to the specific plan version. If the plan changes materially, request approval again.

## Execution after planning

After approval:

1. Reattach the approved plan.
2. Create a short todo list.
3. Execute one bounded step at a time.
4. Validate after each meaningful change.
5. Record progress.
6. Pause if risk increases or assumptions fail.

## Workflow orchestration

Workflow orchestration is a structured execution mode for large plans that need decomposition, parallel read-only work, independent verification, or resumable packet state.

It sits between planning mode and goal-like loops:

```text
planning mode: decide what should happen
workflow orchestration: run a decomposed plan through packets and verification
goal-like loop: continue toward a durable objective across steps or sessions
```

Use workflow orchestration when:

- one linear loop would overload context;
- the task can be split into independent packets;
- multiple areas must be covered systematically;
- findings need independent verifier passes;
- the workflow needs pause, resume, replay, or audit;
- total cost, time, or worker count must be budgeted explicitly.

Do not use workflow orchestration only because a task is important. Use it when decomposition and verification create measurable value.

Workflow execution should follow this sequence:

```text
1. Create a workflow artifact with objective, scope, packets, verification, integration, budgets, and approval points.
2. Validate the artifact against permissions, risk policy, source-of-truth availability, and budget.
3. Ask for approval if the workflow includes risky, expensive, external, destructive, or privileged actions.
4. Execute bounded packets with narrow worker contexts.
5. Verify important findings independently.
6. Integrate results, conflicts, failed packets, coverage gaps, and evidence.
7. Store workflow state and final output outside the prompt.
```

Material workflow changes require a new approval check when they expand scope, raise risk, add tools, increase budget, or change the final commit behavior.

## Goal-like loop

A goal is a durable objective with a measurable done condition. It is different from a plan:

```text
plan: how to approach the work
goal: what state should eventually be true
```

Use a goal-like loop when the agent should continue making progress across many steps, tool calls, or sessions.

Goal state:

```yaml
objective: "..."
status: active | paused | completed | blocked | cancelled
scope: "..."
done_condition: "..."
budget:
  max_steps: 30
  max_cost: "..."
  max_wall_time: "..."
checkpoints:
  - "..."
validation:
  - "..."
forbidden_actions:
  - "..."
approval_required_for:
  - "..."
progress_log_ref: "..."
```

## Good and bad goals

Bad:

```text
Improve support operations.
```

Good:

```text
Analyze the last 200 support escalations, classify the top five repeatable causes, cite evidence for each, propose one operational fix per cause, and stop when the report has passed the source-check and PII-redaction checklist.
```

A good goal has:

- one objective;
- bounded scope;
- source materials;
- allowed tools;
- forbidden actions;
- budget;
- checkpoints;
- validation method;
- stopping condition.

## Checkpoints

For long-running work, add checkpoints:

```text
checkpoint 1: context gathered
checkpoint 2: plan approved
checkpoint 3: first safe artifact produced
checkpoint 4: validation passed
checkpoint 5: final review complete
```

At each checkpoint, record:

- what was done;
- evidence;
- remaining work;
- risks;
- next action.

## Timed re-entry for long-running goals

A durable goal record does not execute itself. In an advanced resident harness, a host-owned heartbeat may periodically reconsider active work, while a schedule requests re-entry at a specific time or interval. Both should enqueue a normal permissioned turn rather than invoke tools directly.

Store a stable wakeup ID, goal and session references, due time, recurrence and time zone, lease, attempt count, misfire policy, and last outcome. Claim wakeups atomically, coalesce duplicates, make handlers idempotent, and re-read durable goal state before acting so a stale timer cannot revive completed, paused, cancelled, or superseded work. Define explicit skip, catch-up, retry, and expiry behavior for downtime. See [self-refining recursive harnesses](self-refining-recursive-harnesses.md) for the resident lifecycle that makes timed re-entry reliable.

## Stopping conditions

Stop when:

- done condition is met;
- budget is reached;
- validation fails repeatedly;
- required approval is missing;
- tool access is unavailable;
- the user changes objective;
- safety policy blocks continuation;
- the agent cannot reduce uncertainty without risky action.

## Planning questions

Ask the user only when needed. Good questions are specific:

- “Should the agent draft only, or may it send after approval?”
- “Which source of truth should win if CRM and billing data conflict?”
- “Is the goal to reduce cost, latency, risk, or user effort?”
- “What is the maximum budget or runtime for this loop?”

Avoid broad questions like “What should I do?” when the agent can safely inspect or propose.

## Plan-validate-execute pattern

For fragile or high-risk operations:

```text
1. Gather source of truth.
2. Create a structured plan.
3. Validate the plan against source data.
4. Ask for approval if needed.
5. Execute the approved plan.
6. Validate the result.
7. Produce a final audit summary.
```

This pattern applies to data migrations, customer communications, financial adjustments, legal document changes, operational runbooks, procurement workflows, and medical literature review workflows.
