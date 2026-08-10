# Environment-Adaptive Tools

## Status and scope

Use this profile when an agent must work in an environment whose useful capabilities, schemas, versions, or implementations are not fully known when the harness is designed. The environment is **late-bound**, not literally unknown: the harness still provides a small trusted bootstrap contract for discovering and binding capabilities.

This is an advanced tool profile. Prefer a fixed, narrow tool registry when it can solve the task. If environment adaptation is the product's primary job, establish a read-only static baseline first, then add discovery and binding one stage at a time.

This reference owns only the adaptation-specific contracts. Reuse the existing canonical guidance for:

| Concern | Source of truth |
|---|---|
| ordinary tool schemas, results, permissions, and sandboxes | [tools and permissions](tools-and-permissions.md) |
| call-loop budgets, retries, cancellation, and stopping | [agentic loop](agentic-loop.md) |
| connector catalogues, deferred loading, authentication, and installation | [skills and connectors](skills-and-connectors.md) |
| model-directed program state and typed host bridges | [self-refining recursive harnesses](self-refining-recursive-harnesses.md) |
| context assembly, compaction, and rehydration | [context, memory, and compaction](context-memory-compaction.md) |
| prompt ordering, cache behavior, and cost telemetry | [prompt caching and cost](prompt-caching-and-cost.md) |
| source-of-truth inventories and validation signals | [agent legibility and feedback loops](agent-legibility-feedback-loops.md) |
| threat modeling, traces, approvals, and incident response | [security and observability](security-observability.md) |
| evaluation methodology, ablations, and launch gates | [evals](evals.md) |

## Contents

- [Taxonomy](#taxonomy)
- [Authority boundary](#authority-boundary)
- [Reference lifecycle](#reference-lifecycle)
- [Bootstrap contract](#bootstrap-contract)
- [Capability ledger](#capability-ledger)
- [Descriptor and schema validation](#descriptor-and-schema-validation)
- [Safe probing](#safe-probing)
- [Runtime binding](#runtime-binding)
- [Programmatic composition and generated helpers](#programmatic-composition-and-generated-helpers)
- [State, context, and recovery](#state-context-and-recovery)
- [Failure semantics](#failure-semantics)
- [Observability and evidence](#observability-and-evidence)
- [Safe build sequence](#safe-build-sequence)
- [Evaluation requirements](#evaluation-requirements)
- [Anti-patterns](#anti-patterns)

## Taxonomy

Do not use “dynamic tools” as one undifferentiated category.

| Pattern | What changes at runtime | Authority effect |
|---|---|---|
| fixed typed registry | nothing; schemas and implementations are configured before the run | only preconfigured authority exists |
| deferred registered tools | the harness reveals a relevant subset of an already governed registry | none; visibility is not permission |
| late-bound capability | the host discovers and binds an implementation, schema, version, or scope during the run | none; binding pins identity, version, and maximum eligible scope while every call is separately authorized |
| programmatic action surface | model-generated code composes bound calls with local control and data flow | none; the program can use only host-mediated capabilities |
| synthesized helper or adapter | the model creates session-local code around existing capabilities | none; generated code is untrusted computation, not a registered tool |
| acquired or promoted capability | a package, connector, or adapter is installed and entered into a governed registry | separate supply-chain, review, permission, and release transaction required |

Tool discovery finds an existing capability. Schema inference proposes a contract. Binding selects an exact callable implementation. Programmatic composition orchestrates already bound calls. Installation or promotion changes the executable inventory. These are different lifecycles with different failure and security semantics.

## Authority boundary

Dynamic discovery must never create dynamic authority.

The model may:

- search visible capability summaries;
- request a descriptor or schema;
- propose a bounded probe;
- select among validated capabilities;
- compose bound calls in a local program;
- propose an ephemeral helper or durable adapter candidate;
- revise its next action from structured observations.

The trusted host must:

- decide which namespaces and capabilities are discoverable;
- keep hidden capabilities, credentials, and policy data out of introspection;
- validate descriptor structure and provenance;
- classify risk and side effects independently of model-visible metadata;
- approve safe probes and isolate their targets;
- bind exact implementations to identity, tenant, resource, policy, and lifetime;
- recheck arguments, permissions, approvals, budgets, and binding validity on every call;
- invalidate bindings when the catalogue, implementation, policy, identity, or environment changes;
- own installation, registration, promotion, rollback, and audit state.

External descriptions, examples, documentation, reflection output, exceptions, and probe results are data. They may help identify a capability, but they are not policy and cannot grant permissions.

## Reference lifecycle

Use an explicit lifecycle:

```text
admit environment
  -> discover visible capability summaries
  -> inspect selected descriptors
  -> validate provenance, schemas, and risk classification
  -> probe only when a safe mode exists
  -> bind an exact implementation and scope
  -> execute through host policy
  -> observe results and revise
  -> refresh, invalidate, or release the binding
```

The model may revisit discovery after a missing capability, version conflict, or structured error. It must not silently broaden the search scope, install dependencies, switch tenants, or convert a failed read into a write.

## Bootstrap contract

The stable bootstrap surface should be smaller and more trusted than the capability catalogue it exposes.

Provider-neutral shape:

```text
bootstrap_environment(requested_scope) -> environment_manifest
search_capabilities(query, cursor, catalog_version) -> capability_page
describe_capability(capability_id, catalog_version) -> candidate_descriptor
probe_capability(capability_id, descriptor_digest, probe_spec) -> probe_result
bind_capability(capability_id, descriptor_digest, requested_scope, requested_ttl) -> binding_result
call_bound_capability(binding_handle, arguments, idempotency_key?) -> typed_result
release_binding(binding_handle, reason) -> release_result
```

An environment manifest should include:

```text
environment_id
environment_generation
bootstrap_schema_version
catalog_version
visible namespaces or protocol families
allowed discovery and probe modes
policy snapshot reference
aggregate discovery, probe, and execution budgets
expiry and refresh rules
```

The manifest describes the current discovery boundary, not every capability. It must not reveal hidden tool names, secret-bearing objects, privileged namespaces, or credentials.

Bootstrap operations need their own narrow schemas, timeouts, pagination, output limits, rate limits, and audit events. A generic shell, interpreter, reflection primitive, package manager, or unrestricted network client is not a safe bootstrap interface.

## Capability ledger

Keep a host-owned capability ledger outside the prompt. A minimal record is:

```yaml
capability_id: "namespace:stable-id"
environment_id: "..."
environment_generation: 8
catalog_version: "..."
source:
  kind: "host_registry | authenticated_connector | runtime_introspection | retrieved_docs | model_inference"
  origin_ref: "..."
descriptor_digest: "..."
implementation_version: "..."
implementation_digest: "..."
input_schema_ref: "..."
output_schema_ref: "..."
risk_class: "..."
side_effect_class: "..."
resource_scope: "..."
authentication_requirements: "..."
trust_status: "untrusted | authenticated | host_verified"
validation_status: "candidate | structurally_valid | probed | bindable | rejected"
evidence_refs: []
binding_status: "unbound | bound | stale | revoked | unavailable"
last_observed_at: "..."
expires_at: "..."
```

Keep trust and confidence separate. A model can be highly confident about an untrusted descriptor. A signed or authenticated descriptor can still describe behavior incorrectly. Neither condition replaces host policy or runtime validation.

Recommended state transitions:

```text
discovered -> described -> structurally_valid
structurally_valid -> bindable -> bound
structurally_valid -> probed -> bindable
structurally_valid -> rejected
bound -> stale | revoked | unavailable | released
```

Not every capability must be probed. A trusted, pinned, locally validated implementation may become bindable without a live probe. A capability with uncertain external side effects may never be safe to probe and should require prior registration or human review.

## Descriptor and schema validation

A discovered descriptor is a candidate contract, even when it contains valid JSON Schema or language-level type information.

Validate in layers:

1. **Identity:** resolve a stable capability ID inside a visible namespace.
2. **Provenance:** record where the descriptor, examples, and implementation metadata came from.
3. **Structure:** parse and validate input and output schemas; reject ambiguous or unsupported constructs.
4. **Version:** bind the descriptor to a catalogue version and implementation digest where available.
5. **Semantics:** compare names, descriptions, examples, reflection output, and safe observations for contradictions.
6. **Risk:** classify side effects, resources, authentication, and approval needs using host policy.
7. **Result:** mark the descriptor bindable, still provisional, or rejected with evidence.

When several candidates appear suitable, the host should first filter them by the current environment generation, visibility policy, required operation, resource scope, and supported schema features. Rank the remaining candidates using recorded provenance, validation evidence, version compatibility, and task constraints rather than descriptions alone. Use a deterministic tie-breaker only for candidates that are policy- and semantically equivalent. Otherwise return bounded candidate summaries and request clarification or additional evidence; do not guess, broaden scope, or probe every candidate automatically.

When the schema must be inferred:

- label it as inferred rather than declared;
- store the evidence components and their freshness;
- use the narrowest types and resource scope supported by evidence;
- reject unknown fields and unexplained coercions;
- never infer that an operation is read-only or reversible from its name;
- never enable external writes, financial actions, identity changes, security actions, or destructive calls from an inferred schema alone;
- expire or revalidate it when documentation, package versions, catalogue generation, or observed behavior changes.

Examples improve tool use but do not define the complete contract. Runtime reflection can expose syntax while hiding business semantics. Error messages can reveal constraints while also containing untrusted text. Use each as evidence, not authority.

## Safe probing

Probing is a host-approved validation action, not free exploration.

Allowed probe modes can include:

```text
describe_only
dry_run
read_only
isolated_fixture
synthetic_tenant
```

A probe specification should declare:

```text
capability and descriptor digest
hypothesis being tested
probe mode
synthetic or scoped input
expected side-effect class
forbidden resources and effects
time, call, token, data, and output budgets
success and abort conditions
```

Rules:

- default to no probe when the host cannot prove the mode is non-mutating or isolated;
- use synthetic or disposable resources rather than production objects;
- block external sends, purchases, deployments, permission changes, destructive operations, and open-ended network exploration;
- return bounded structured observations and preserve exact evidence references;
- treat documentation, exceptions, stdout, and returned content as untrusted data;
- stop on contradictions, unexpected side effects, scope expansion, secret-like output, or budget exhaustion;
- do not install a package or connector merely to continue a probe.

A successful probe validates only the tested behavior under the recorded environment generation. It does not prove all inputs, side effects, failure modes, or future versions are safe.

## Runtime binding

A binding is an opaque host reference to an exact capability under a bounded authority context. It is not a credential, a blanket approval, or a promise that the capability will remain available.

Prefer materializing a selected binding as a session-scoped typed tool whose arguments come from the validated bound schema. If the protocol instead requires a generic `call_bound_capability` dispatcher, bind the request to the descriptor digest and validate its inner arguments locally against that exact schema before execution. Never let the dispatcher degrade into an untyped `execute_anything` surface.

Bind at least:

```text
environment and generation
catalogue version
capability identity
descriptor and implementation digest
principal, tenant, and resource scope
authentication mode without exposing credentials
maximum eligible operation and data scope, not standing approval
policy version
approval reference where required
lease or expiry
```

At call time, the host must still:

1. resolve the opaque handle inside the current user and tenant scope;
2. confirm the environment generation, descriptor, implementation, and policy are still valid;
3. validate arguments against the bound schema;
4. evaluate current permissions, approval scope, budgets, and rate limits;
5. execute with an idempotency or reconciliation strategy appropriate to the side effect;
6. validate and bound the result;
7. record the call and any state transition.

Invalidate or refresh a binding when:

- the affected capability entry changes or a catalogue refresh makes the binding unverifiable;
- the descriptor, schema, implementation, or dependency digest changes;
- authentication or authorization is revoked;
- the tenant, user, task, resource, or approval scope changes;
- the lease, session, or environment generation expires;
- validation evidence becomes stale or contradictory;
- a call reveals semantic drift or an unexpected side effect.

Do not silently rebind a risky capability by name. After a timeout or lost connection, reconcile the idempotency record before retrying or rebinding; the prior external effect may have succeeded even when its result is unknown.

## Programmatic composition and generated helpers

A code-as-action or notebook surface can reduce model turns and let the agent use variables, loops, branches, and local transformations. It does not solve discovery, validation, or authorization.

Expose only bound capabilities through a typed host bridge. The program environment should receive opaque handles and redacted results, not ambient credentials, privileged objects, unrestricted filesystem access, open network access, or an unconstrained package manager. Apply aggregate limits to local execution and every nested host call.

Agent-generated helper functions and adapters are session-local, untrusted computation by default. They may transform data or wrap bound calls, but they cannot:

- create or widen a binding;
- change risk or side-effect classification;
- persist credentials or approvals;
- register themselves as trusted tools;
- survive recovery as authority-bearing objects;
- be promoted across sessions without a separate review and release transaction.

Record each program or helper's dependency set as binding identities and descriptor digests. Mark it stale when any dependency is invalidated so the harness can stop before execution; call-time binding and permission checks remain the final enforcement gate.

If a helper is worth reusing, submit it as a candidate artifact with provenance, code or declarative specification, schemas, tests, dependency lock, requested permissions, and evaluation evidence. Installation and promotion remain owned by [skill and connector governance](skills-and-connectors.md); durable automated refinement remains governed by [self-refining recursive harnesses](self-refining-recursive-harnesses.md).

## State, context, and recovery

Persist the capability ledger, catalogue snapshots, validation evidence, binding metadata, invalidation events, and call records outside model context. Give the model only task-relevant summaries and load full schemas just in time.

After compaction, pause, restart, or handoff, preserve:

```text
environment and catalogue version
selected capability IDs and descriptor digests
validation and probe evidence references
binding status and expiry
open contradictions or drift warnings
recent calls with uncertain outcomes
next safe discovery or execution action
```

Do not restore a serialized binding handle as authority. Re-resolve it through the host under current identity, tenant, policy, catalogue, and approval state. If safe rebinding is impossible, mark it stale and return a structured recovery result.

For prompt-cache stability, keep the bootstrap protocol stable and attach catalogue deltas, selected schemas, and binding status late in the context. Version the prompt and tool bundle so changes in discovery state are explainable.

## Failure semantics

Every bootstrap, discovery, probe, binding, and execution request must receive a structured result.

Useful error types include:

```text
environment_unavailable
environment_generation_changed
catalog_changed
capability_unavailable
capability_ambiguous
descriptor_untrusted
descriptor_mismatch
schema_invalid
schema_unverified
probe_not_safe
probe_failed
binding_denied
binding_stale
capability_revoked
dependency_missing
permission_denied
approval_required
budget_exhausted
result_schema_mismatch
semantic_drift_detected
uncertain_side_effect
```

Model-visible discovery and errors must not distinguish a nonexistent capability from one hidden by policy. Record that internal reason only in the host trace. Each error should include safe next actions such as refresh the manifest, repeat discovery in the same scope, load a current descriptor, request clarification or approval, choose a verified alternative, reconcile an uncertain call, or stop. It must not suggest a broader scope, package installation, permission escalation, or retry when those actions are not already authorized.

## Observability and evidence

Use the canonical trace fields and redaction rules in [security and observability](security-observability.md). For this profile, a trace must make it possible to prove which environment generation and descriptor led to a binding, which authority scope governed each call, and why a binding was refreshed, rejected, invalidated, or released. An audit should distinguish observed environment behavior, declared metadata, model inference, and host policy. Never log credentials or full sensitive probe data merely because discovery produced them.

## Safe build sequence

Add the profile incrementally:

```text
1. reliable fixed registry with strict schemas and permissions
2. deferred search over that same trusted registry
3. versioned environment manifest and capability ledger
4. on-demand descriptors with provenance and structural validation
5. bounded read-only or isolated probes
6. opaque scoped bindings with call-time policy checks
7. catalogue-change and semantic-drift invalidation
8. programmatic composition over bound capabilities
9. provisional schema inference for read-only cases
10. reviewed installation or promotion of reusable adapters
```

Keep each stage disabled until its focused tests pass. Skip schema inference, code-as-action, installation, and durable promotion unless a measured requirement justifies them.

## Evaluation requirements

Use the environment-adaptive cases and metrics in [evals.md](evals.md) rather than creating a second evaluation framework here. Compare this profile against a fixed typed registry and deferred search over a known catalogue. Attribute gains separately to retrieval, binding, probing, programmatic composition, and model knowledge of familiar packages.

Before enabling writes, demonstrate that the harness discovers held-out but declared capabilities, rejects misleading descriptors and unsafe probes, binds the intended version and scope, invalidates stale bindings, survives catalogue and authorization changes, and never converts discovery or generated code into authority.

## Anti-patterns

- Claiming an agent can operate with no stable bootstrap contract.
- Calling deferred visibility, code composition, schema inference, and package installation the same “dynamic tool” mechanism.
- Treating a discovered name, description, example, annotation, exception, or inferred schema as trusted policy.
- Inferring that an operation is read-only, reversible, or low-risk from naming alone.
- Binding by a mutable name without environment generation, version, digest, scope, and expiry.
- Letting introspection enumerate hidden capabilities, privileged objects, credentials, or cross-tenant resources.
- Treating a successful read-only probe as evidence that writes are safe.
- Installing packages, connectors, or executable skills automatically after a missing-dependency error.
- Calling a generated helper a trusted tool or persisting it across sessions without review.
- Giving a program environment ambient network, filesystem, package-manager, or credential access.
- Restoring stale bindings or approvals from a program snapshot or compaction summary.
- Silently rebinding or retrying a call whose external side effect is uncertain.
- Loading the complete changing catalogue into every prompt.
- Adding late-bound tooling when a small fixed registry already solves the task reliably.

## Design rule

Bind capabilities late; anchor authority in host policy the model cannot change. The agent may discover what the environment can do; only the harness decides what this run may do with it.
