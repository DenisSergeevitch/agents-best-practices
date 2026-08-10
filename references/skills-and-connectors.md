# Skills, MCP, and External Connectors

## Skill purpose

A skill is reusable procedural knowledge packaged for progressive loading. It helps an agent handle a class of tasks without putting every workflow instruction in the main prompt.

Use skills for:

- repeatable workflows;
- domain-specific procedures;
- organizational conventions;
- output templates;
- validation checklists;
- gotchas the model would otherwise miss;
- reusable reference material.

Do not use skills for one-off task instructions.

## Agent Skills structure

A portable Agent Skill is a directory with at least:

```text
skill-name/
  SKILL.md
```

The required `SKILL.md` has YAML frontmatter and Markdown instructions. The `name` must match the parent directory, use lowercase letters/numbers/hyphens, and stay within the specification limits.

Optional reference material should be split into Markdown files such as:

```text
references/process.md
references/checklist.md
references/templates.md
```

For a Markdown-only skill, do not add scripts, binaries, images, or data files.

## Skill metadata

A strong description is essential because agents often load only `name` and `description` at startup.

Description guidance:

- Start with “Use this skill when...”
- Describe user intent, not implementation internals.
- Mention adjacent terms users may use.
- Include boundaries so the skill does not trigger too broadly.
- Keep it concise and below the specification limit.

Example:

```yaml
---
name: renewal-risk-analysis
description: Use this skill when analyzing renewal risk, account health, churn likelihood, expansion blockers, or customer retention actions using usage, support, contract, and sentiment data.
---
```

## Progressive disclosure

Use progressive disclosure:

```text
1. Startup: expose only skill name and description.
2. Activation: load SKILL.md core instructions.
3. On demand: load focused reference files.
```

Do not pack all possible details into `SKILL.md`. Keep the entry point short and point to reference files with clear triggers.

Good reference trigger:

```text
Read references/approval-policy.md when the workflow includes external sends, payments, permission changes, or destructive actions.
```

Bad reference trigger:

```text
See references/ for more information.
```

## Skill content pattern

Use this structure:

```markdown
# [Skill Name]

## When to use
...

## Inputs to identify
...

## Procedure
1. ...
2. ...
3. ...

## Tools to prefer
...

## Tools to avoid
...

## Validation
...

## Output template
...

## Gotchas
...
```

## Skill governance

Skills can change behavior and tool use. Treat them as supply-chain artifacts.

Governance:

- source verification;
- publisher identity;
- version pinning;
- review before installation;
- permission manifest;
- static scan where relevant;
- runtime sandboxing for executable assets;
- inventory and audit logs;
- removal and incident response process.

For Markdown-only skills, still review prompt-injection risk, overbroad instructions, hidden policy conflicts, and excessive tool permissions.

## Descriptor versus executable package

Do not confuse a mutable skill descriptor with an installed executable skill package:

```text
descriptor: activation metadata and procedural guidance loaded into context
executable package: reviewed code, dependencies, callable entry points, schemas, and permission manifest
```

Changing a descriptor can alter model behavior, but it does not install or verify executable capability. Installing or updating a package requires an explicit supply-chain transaction: resolve and pin dependencies, review the source and manifest, provision an isolated runtime, test the declared calls, record provenance, and support disablement or rollback. A refinement mechanism may propose descriptor edits; it must not silently turn those edits into code installation or package mutation.

This distinction matters for [self-refining recursive harnesses](self-refining-recursive-harnesses.md), where mutable supplemental state may refer to callable skills. It does not change this repository's Markdown-only skill policy.

## Skill evaluation

Evaluate both activation and output quality.

Activation eval:

```text
should-trigger queries
should-not-trigger near misses
multiple phrasings
casual language and typos
multi-step user tasks where the skill is relevant but not obvious
```

Output eval:

```text
task success
policy compliance
tool choice
unnecessary tool calls
use of validation steps
citation/evidence quality
format adherence
failure handling
```

Keep train and validation sets separate when optimizing the description.

## MCP and external connectors

MCP is a standard way to connect an AI application to external data, tools, and workflow prompts. More generally, treat any connector protocol as an external capability layer.

Connector features usually map to:

```text
resources: data/context the model or user can read
prompts: reusable templates or workflows
tools: executable functions or actions
```

## Connector attachment strategy

Do not attach all connector tools up front. Use staged exposure:

```text
1. List available connector servers or domains.
2. Search or load only relevant tool summaries.
3. Load full schemas only for likely tools.
4. Execute only after validation and permission checks.
5. Return compact results or references.
```

For large connector ecosystems, provide a `search_tools` or `list_capabilities` mechanism.

## Connector safety

Connector tools should be:

- namespaced by server or source;
- scoped by user and tenant;
- described concisely;
- treated as untrusted unless from a trusted source;
- permissioned by risk class;
- logged on every call;
- disabled when unused;
- version-pinned where possible.

Tool annotations and descriptions from external servers can be wrong or malicious. The harness must not blindly trust them.

## Authentication versus authorization

Authentication proves a connector can be accessed. Authorization decides what this agent may do now.

Use:

```text
per-user credentials
least-privilege scopes
short-lived tokens
resource-level checks
approval gates for risky operations
revocation
call logging
```

Do not give the model raw tokens. Let the connector manager use tokens internally and return redacted observations.

## Tool search and deferred loading

Deferred loading prevents context overload.

Pattern:

```text
visible tool: search_connector_tools(query, detail_level)
result: tool names, short descriptions, risk classes
next: load_tool_schema(tool_name) for selected tools
then: call selected tool after permission check
```

Detail levels:

```text
name_only
name_and_description
full_schema
examples
```

This pattern progressively reveals tools that are already registered and governed. When the environment itself is late-bound and candidates need provenance checks, safe probes, exact runtime bindings, or drift invalidation, use [environment-adaptive tools](environment-adaptive-tools.md). Discovery and binding do not install a connector or executable package; installation remains a separate reviewed transaction.

## Code execution with connectors

When many tools or large data are involved, consider using a sandboxed execution environment to interact with connector APIs programmatically. Benefits:

- load only needed tool definitions;
- filter or aggregate large data before model context;
- keep intermediate sensitive data outside the model;
- persist temporary state;
- reduce repeated tool-call loops.

Use this only with sandboxing, resource limits, logging, and strict credential boundaries.

## Skill and connector anti-patterns

Avoid:

- a skill that silently grants broad permissions;
- connector tools exposed without namespacing;
- loading hundreds of tool schemas into the prompt;
- using external connector descriptions as trusted policy;
- installing unreviewed skills from unknown sources;
- letting a connector perform sampling or sub-agent behavior without user approval;
- returning huge connector payloads directly to the model;
- allowing connector credentials to leak into context.

## Source links

- Agent Skills specification: https://agentskills.io/specification
- Agent Skills creator best practices: https://agentskills.io/skill-creation/best-practices
- Agent Skills description optimization: https://agentskills.io/skill-creation/optimizing-descriptions
- Agent Skills evaluation guide: https://agentskills.io/skill-creation/evaluating-skills
- MCP specification: https://modelcontextprotocol.io/specification/2026-07-28
- MCP authorization: https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- OpenAI tools: https://developers.openai.com/api/docs/guides/tools
- Anthropic code execution with MCP: https://www.anthropic.com/engineering/code-execution-with-mcp
