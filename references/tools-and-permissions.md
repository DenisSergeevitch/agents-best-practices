# Tools and Permissions

## Tool design principle

A tool is a contract between the model and the harness. The model sees the contract; the harness owns execution.

Each tool should define:

```text
name
purpose
input schema
output schema
risk class
side-effect class
resource scope
permission policy
timeout
result-size limit
retry policy
audit policy
error format
```

Avoid broad tools. Prefer narrow tools with domain semantics.

Bad:

```text
execute_anything(command)
call_api(url, method, body)
update_database(sql)
send_message(payload)
```

Better:

```text
search_policy_docs(query, max_results)
read_customer_account(account_id)
draft_customer_email(case_id, tone)
request_refund_approval(order_id, amount, reason)
apply_approved_refund(approval_id)
```

## Tool schema rules

- Use strict JSON schemas where supported.
- Mark required fields explicitly.
- Reject unknown properties.
- Use enums for constrained actions.
- Prefer IDs and typed fields over freeform instructions.
- Validate locally before execution.
- Treat schema adherence as reliability, not security.

Example schema pattern:

```json
{
  "type": "object",
  "properties": {
    "record_id": { "type": "string" },
    "new_status": {
      "type": "string",
      "enum": ["open", "pending", "resolved"]
    },
    "reason": { "type": "string" }
  },
  "required": ["record_id", "new_status", "reason"],
  "additionalProperties": false
}
```

## Record provenance and authoritative fields

For tools that act on or present business records, the host should track which records an authorized backend read has introduced into the active scope. A valid-looking ID in model output, user text, or retrieved content is only a lookup candidate; it is not evidence that the record exists or may be used.

Keep a host-owned record of the resource ID, source read, principal/tenant scope, record version or observation time, and expiry. Define which operations may introduce records: an authorized direct lookup or an existing container membership can be valid entry paths. Unrecognized or expired references require a scoped read before use, or a structured refusal. Do not silently accept a reference copied from another session.

Provenance does not grant access or approval. Recheck current authorization for the target operation; a read-only delegate's findings must not automatically expand the parent's actionable record set. If the parent needs to act on a delegate's result, resolve it through the parent's authorized read path first. Apply the existing [untrusted-data rules](security-observability.md#prompt-injection-handling) to all descriptive fields, even when the record ID is verified.

Keep business calculations, eligibility, ranking, and authoritative fields in the owning backend. Tool adapters should expose that domain contract rather than recreate its rules. When exact wording is required, the host supplies approved copy and its version; the model selects the relevant record or disclosure. Server-owned prices and disclosures do not establish the truth of adjacent model-written explanations.

## Presentation tools and rendered state

Use a typed presentation tool when the agent selects UI components such as ordered cards, comparisons, or plans. The model supplies a component type, ordered record references, and allowed explanatory fields. The host validates the payload, resolves permitted records, fills authoritative fields, and emits a versioned UI event. Reuse [record provenance](#record-provenance-and-authoritative-fields) for this boundary.

The proposed tool arguments may differ from the displayed result after filtering, enrichment, pagination, or client sorting. Return a compact presentation receipt to the agent and retain it with the conversation:

```text
presentation_id and revision
component type and parent/group structure
ordered visible record IDs and labels
record/disclosure versions or observation references
omitted items and reasons
status: emitted | displayed | failed | superseded
```

The host assigns identity and revisions; the client acknowledges the displayed revision and reports any reordering that changes references such as "the second item." Ignore stale acknowledgements. If the client cannot report layout, require it to preserve the emitted ordering and treat delivery as unconfirmed until observed. Resolve follow-up references against the relevant displayed receipt; if it is missing, superseded, or ambiguous, refresh the view or ask for the intended item rather than guessing. Preserve the receipt reference through compaction and rehydrate it from the host; replaying history does not rerun a write tool.

Start with complete validated components. Optional progressive rendering must use validated, authorized fields, mark previews as provisional, and replace them by the same presentation identity when the final event arrives. Partial JSON is not an accepted action or a complete render receipt. On validation failure or cancellation, remove or mark the provisional view and return a structured result. Progress text should describe observed operations without claiming completion. Follow the existing [streaming rules](provider-api-patterns.md#streaming); measure user-visible progress separately from task completion in [evals](evals.md).

## Risk taxonomy

Classify every tool:

```text
read_only
search_only
compute_only
draft_only
write_local
write_internal
write_external
financial
communication
identity_access
security_sensitive
process_execution
network_open_world
destructive
privileged_admin
```

The tool registry should expose risk metadata to the permission engine.

## Permission matrix

Default permission policy:

```text
public read: allow
private user read: allow only inside user/session scope
organization read: role-based allow
search web: allow or restrict by product policy
compute-only: allow in bounded environment
draft-only: allow
write local artifact: allow when scoped
write internal record: approval or policy allowlist
external communication: draft first, approval to send
financial action: approval plus strong auth
destructive action: deny by default or approval plus recovery plan
identity/access change: approval plus strong auth
process execution: sandbox plus allowlist plus timeout
connector installation: approval plus review
```

## Permission decision object

A permission engine should return one of:

```text
allow
deny
ask_user
approval_required
require_stronger_auth
run_in_sandbox
run_as_draft_only
```

Record:

```text
tool name
arguments or argument hash
risk class
resource scope
decision
policy rule
user/session
approver if any
timestamp
```

## Draft versus commit

Split risky actions into separate tools:

```text
draft_email -> send_email
prepare_refund -> issue_refund
propose_record_update -> apply_record_update
prepare_contract_change -> submit_contract_change
recommend_trade -> place_trade
stage_workflow_change -> commit_workflow_change
```

Draft tools can often run automatically. Commit tools require approval unless the action is low-risk and explicitly allowlisted.

## Resulting-state limits and apply-time checks

Business limits apply to the resulting state, including prior successful writes and active reservations. For example, a limit of four items must constrain two separate requests to add three items each, not only validate that each requested quantity is below four. Define whether the limit belongs to a resource, person, account, or time window; a session-local counter cannot enforce an account-wide rule.

At the authoritative backend, atomically read the relevant state/version, validate the proposed result and current policy, and commit the mutation with its limit accounting. Serialize or use conditional writes at the resource scope shared by all callers, including other sessions, workers, and human interfaces. A process-local session lock alone is insufficient when those callers share state. Return a conflict or limit result when the precondition fails, without a partial write; an allowed reduced amount must be explicit in the result and within the approved scope.

Staged changes should have a host-issued identity binding the target, expected version, proposed diff, and approval scope. At apply time, recheck authorization, approval validity, current guardrails, and live target/limit state in the same commit boundary. If changed state alters the approved effect, reject the stale stage and prepare a fresh proposal under the applicable approval policy. Rechecking new policy against old values alone does not detect target drift. Use the existing [approval records](security-observability.md#approval-records) and [retry policy](agentic-loop.md#retry-policy): idempotency handles duplicate delivery of one operation, while resulting-state checks also cover distinct repeated operations. Reconcile an unknown commit outcome before retrying it or releasing its reservation.

## Coding-agent baseline tools

For repository-facing agents, use [coding-agents.md](coding-agents.md) for the concrete tool registry, permission defaults, command policy, path safety, and diff-accounting rules.

The short rule is: make repo inspection, patching, validation, review handoff, and safety tools explicit. If shell is necessary, wrap it with command normalization, fixed cwd, approval policy, path extraction, timeouts, output caps, secret isolation, and structured results.

## Programmatic execution facade

An advanced harness may expose one model-facing interpreter or notebook instead of many first-class tool calls. Treat that interface as a policy-mediated facade, not as a grant of ambient authority. Every host capability reached from the program must still resolve to a typed operation with its own identity, schema validation, resource scope, permission decision, timeout, output limit, and audit event.

Keep credentials and authoritative state in the host. The interpreter should receive handles or redacted results, not raw secrets, and it must not be able to forge approvals or bypass per-operation policy. Persisted variables are useful working state, but they are untrusted input when restored.

A persistent REPL is not a sandbox. Isolate and constrain the execution environment independently, and assume generated code can misuse every filesystem path, network route, process primitive, credential, and host bridge exposed to it. Use this pattern only after a narrow tool loop is reliable; see [self-refining recursive harnesses](self-refining-recursive-harnesses.md) for the advanced composition.

If the harness prelaunches calls from partially generated code, use [speculative tool execution](speculative-tool-execution.md). Partial output never authorizes the call: every physical dispatch still needs the exact typed binding, local validation, permission decision, and speculation-specific resource admission defined by the host.

When the catalogue, schema, version, or implementation is not known until runtime, use [environment-adaptive tools](environment-adaptive-tools.md) for the bootstrap, validation, binding, and invalidation lifecycle. The schema, permission, result, retry, sandbox, and secret rules in this file still apply to every bound call.

## Tool result format

Return structured observations:

```json
{
  "status": "success",
  "summary": "Found 3 matching cases.",
  "items": [
    {
      "id": "case_123",
      "title": "Renewal blocker",
      "evidence_ref": "crm://cases/case_123"
    }
  ],
  "next_valid_actions": ["read_case", "draft_response"]
}
```

For errors:

```json
{
  "status": "error",
  "type": "permission_denied",
  "message": "Sending external email requires approval.",
  "next_valid_actions": ["draft_email", "request_approval"]
}
```

Do not return huge raw blobs. Summarize, paginate, filter, or store bulky artifacts outside context and return a reference.

## Tool result limits

Set limits:

```text
max_result_chars
max_items
pagination cursor
log tail length
snippet length
artifact storage reference
```

For large data, let the tool compute or filter before returning to the model. The model should not receive 10,000 rows just to count five relevant records.

## Error handling

Every failure is a result:

```text
unknown_tool
invalid_arguments
permission_denied
approval_required
auth_expired
not_found
timeout
rate_limited
conflict
non_idempotent_retry_blocked
internal_error
```

The error should include safe next steps.

## Sandboxing

Use sandboxing for:

- shell/process execution;
- browser automation;
- generated code;
- file manipulation;
- untrusted tools;
- external data processing;
- complex connector workflows.

Sandbox controls:

```text
filesystem allowlist
network allowlist
process timeout
CPU/memory limits
secret isolation
read-only mounts where possible
temporary workspace
snapshot/resume support
egress logging
artifact export policy
```

## Secrets

Do not put secrets in model context. Tools may use credentials internally, but should return redacted summaries.

Rules:

- use short-lived scoped tokens;
- bind credentials to user/session;
- redact secrets in traces and tool results;
- avoid ambient environment credentials;
- block secret-like file reads unless explicitly approved;
- never ask the model to copy credentials between tools.

## Tool visibility

Do not show every tool all the time.

Use:

```text
base tools: always visible
task tools: visible after task classification
skill tools: visible after skill selection
connector tools: visible after connector authorization
deferred tools: discoverable by search
sensitive tools: hidden until needed and approved
```

Large tool surfaces confuse the model and waste context.

## Tool descriptions

A good tool description says:

- when to use the tool;
- when not to use it;
- required prerequisites;
- side effects;
- important error behavior;
- examples of valid arguments.

Keep descriptions concise. If a tool requires extensive documentation, expose a small discovery tool or reference resource rather than putting all details in the tool description.
