# Agentic Loop

## Canonical loop

The provider-neutral loop is:

```text
while not done:
  build context
  call model with visible tools
  receive final answer or tool requests
  validate every tool request
  check permission and approval policy
  execute or deny each tool request
  append structured tool results
  compact or retrieve context if needed
  stop on completion or budget
```

The model never executes a tool directly. It emits a structured request. The harness executes or rejects it.

## Loop invariants

Enforce these invariants in code:

1. Every tool call receives exactly one corresponding result.
2. Tool arguments are parsed and validated before execution.
3. A permission decision happens before every side effect.
4. Tool results are bounded, structured, and traceable.
5. The loop has hard step, time, token, cost, and tool-call budgets.
6. The final answer is based on observations, not assumed tool success.
7. Errors, denials, cancellations, and timeouts become structured observations.

## Simple pseudocode

```python
def run_agent(task, session):
    session.add_user_message(task)

    for step in range(session.max_steps):
        context = context_builder.build(session)

        if budget.exceeded(session):
            return stop("budget_exceeded", session)

        if compactor.should_compact(context, session):
            session = compactor.compact(session)
            context = context_builder.build(session)

        output = model.generate(
            context=context,
            tools=tool_registry.visible_tools(session),
        )
        session.record_model_output(output)

        if output.final_answer:
            return finalize(output.final_answer, session)

        if not output.tool_calls:
            return stop("no_final_answer_or_tool_call", session)

        for call in scheduler.order(output.tool_calls):
            result = handle_tool_call(call, session)
            session.add_tool_result(call.id, result)

    return stop("step_limit_reached", session)
```

Tool-call handler:

```python
def handle_tool_call(call, session):
    tool = tool_registry.get(call.name)
    if tool is None:
        return error_result("unknown_tool", call.name)

    try:
        args = tool.validate(call.arguments)
    except ValidationError as exc:
        return error_result("invalid_arguments", str(exc))

    decision = permission_engine.evaluate(tool, args, session)

    if decision.type == "deny":
        return denied_result(decision.reason)

    if decision.type == "approval_required":
        return pause_for_approval(call, decision, session)

    if decision.type == "sandbox":
        return sandbox.execute(tool, args)

    return tool.execute(args)
```

## Manual loop versus hosted loop

Some APIs require the application to run the entire loop manually. Others can perform parts of the loop server-side for hosted tools. The architecture should stay the same conceptually:

```text
manual client loop
- application sends tools
- model requests tool call
- application executes tool
- application sends tool result
- repeat

hosted or provider-assisted loop
- provider may execute hosted tools
- application still controls business tools, permissions, approvals, state, and traces
```

Even when the provider supports hosted tools, keep business-critical authorization and audit in the harness.

## Step budgets

Use explicit budgets:

```text
max_model_turns
max_tool_calls
max_parallel_tool_calls
max_wall_time_seconds
max_input_tokens
max_output_tokens
max_total_cost
max_tool_result_chars
max_retries_per_model_call
max_retries_per_tool_call
```

When a budget is reached, stop with a clear status:

```json
{
  "status": "stopped",
  "reason": "step_limit_reached",
  "completed": false,
  "next_safe_action": "Ask the user whether to continue with a larger budget."
}
```

## Retry policy

Retry only safe failures.

Usually safe to retry:

- transient model API errors;
- network timeouts for read-only calls;
- idempotent retrieval;
- validation after the model fixes malformed arguments.

Do not automatically retry:

- payments;
- external sends;
- destructive actions;
- permission changes;
- operations with unclear idempotency.

For high-risk operations, use idempotency keys and approval records.

## Parallelization

Parallelize only independent, read-only, concurrency-safe tool calls.

Safe candidates:

- search;
- read;
- retrieve metadata;
- classify independent records;
- summarize independent documents.

Serialize:

- writes;
- sends;
- deletes;
- financial actions;
- permission changes;
- shell/process execution;
- multi-step external workflow commits.

This is committed parallelism: the complete calls and their independence are already known. An advanced code-mode harness may instead predict and prelaunch eligible calls while a program is still being generated. That requires exact claim-or-run identity, isolated disposable state, separate waste budgets, and cancellation accounting; use [speculative tool execution](speculative-tool-execution.md) rather than weakening the rules above.

## Human-in-the-loop loop

Sensitive actions should pause the loop:

```text
model requests action
  -> harness validates
  -> harness detects approval requirement
  -> harness emits approval request
  -> user or policy approves/rejects
  -> harness resumes with approval_result
```

Approval must be scoped to the exact action. Do not treat vague consent as blanket authorization.

## Goal-like loop

A goal loop is a long-running version of the standard loop. It needs additional state:

```text
objective
done condition
budget
checkpoints
current plan
progress log
validation method
stop rules
```

The loop should periodically ask:

1. Is the objective still valid?
2. What evidence proves progress?
3. Are we within budget?
4. Is the done condition met?
5. Is human approval needed before the next step?
6. Should compaction or handoff happen now?

Goal loops should not be used for vague backlogs or unrelated tasks.

## Termination rules

Stop when any of these are true:

- final answer produced;
- done condition satisfied;
- user approval is required;
- blocker requires user input;
- budget reached;
- repeated failure threshold reached;
- safety policy denies the task;
- tool or connector unavailable and no safe fallback exists.

## Provider-neutral implementation notes

- With OpenAI Responses-style APIs, represent model outputs as typed items and use previous response or conversation state if appropriate.
- With Chat Completions-style or OpenAI-compatible APIs, maintain message history manually and append tool result messages with matching call IDs.
- With Anthropic APIs, handle structured tool-use blocks and return corresponding tool-result blocks.
- With any provider, keep application-side validation, permissioning, and audit logs outside the model.
