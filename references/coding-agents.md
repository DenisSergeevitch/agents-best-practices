# Coding-Agent Harnesses

Use this reference when the requested agent reads, edits, tests, reviews, migrates, or opens changes against a software repository. This is a domain overlay on the general MVP harness, not the default architecture for every agent.

## MVP boundary

A coding-agent MVP is not an autonomous engineer. It is a bounded draft-change worker that can:

1. inspect repository instructions, status, and structure;
2. classify the task and identify the smallest relevant file set;
3. produce a short plan when the work is ambiguous, risky, or multi-step;
4. make a minimal local patch;
5. run deterministic checks inside policy;
6. attempt a bounded repair loop on clear validation failures;
7. self-review the diff for scope, safety, and evidence;
8. produce a reviewable handoff with commands run, results, risks, and remaining gaps.

The product boundary is:

```text
MVP coding agent = draft + verify + explain.
Not merge + deploy + own production.
```

## Core loop

```text
issue / task
  -> read workspace instructions and current repository state
  -> record baseline git state and pre-existing user changes
  -> classify task type and risk
  -> search and read relevant files
  -> identify available test, lint, typecheck, or build commands
  -> produce a short plan if scope or risk requires it
  -> apply a minimal draft patch
  -> run narrow validation
  -> repair once or twice only when the failure is clear and in scope
  -> run broader validation when the changed surface justifies it
  -> inspect the final diff for unrelated churn, path escapes, and secret exposure
  -> produce final evidence for human review or approval
```

For coding agents, "done" requires evidence. The final answer, change summary, or draft change-request body should include:

```text
task understood
scope and files changed
commands run
checks passed, failed, skipped, or unavailable
behavioral before/after when applicable
assumptions
risks and rollback notes
reviewer notes or follow-up gaps
```

If validation cannot run, the agent should say which evidence is missing and why. It should not claim the change is complete merely because it edited files.

## Task profiles

Bug-fix agent:

- reproduce the failing behavior if feasible;
- make the minimal patch;
- add or update a regression test when reasonable;
- report validation commands and results.

Code-review agent:

- produce risk-ranked findings with file and line evidence;
- avoid blocking comments without concrete evidence;
- suggest patches only when confidence is high;
- separate correctness, security, maintainability, and test gaps.

Migration agent:

- inventory affected files before editing;
- apply a mechanical transformation where possible;
- list skipped, ambiguous, or manually reviewed cases;
- validate representative and global checks.

Dependency-upgrade agent:

- state the version change and reason;
- summarize relevant compatibility risk;
- update lockfiles only when in scope;
- run tests/build and include rollback notes.

Test-generation agent:

- capture existing behavior first;
- avoid changing product behavior unless explicitly requested;
- prefer focused tests over broad snapshots;
- show that tests exercise the changed or risky path.

Docs-sync agent:

- update docs from source-of-truth code, API, or policy artifacts;
- remove or flag stale instructions;
- cite source files or references used;
- avoid inventing product behavior.

## Baseline tools

Make repository work explicit instead of exposing raw shell or a broad filesystem API as the main interface.

Repo inspection:

```text
read_workspace_instructions(path)
git_status(cwd)
list_files(path, glob, limit)
search_code(query, path_globs, max_results)
read_file(path, line_range)
inspect_symbol(symbol, path_globs)
detect_project_commands(cwd)
```

Workspace edits:

```text
propose_patch(summary, files, risk_notes)
apply_patch(patch_id)
revert_patch(patch_id)
inspect_diff(scope)
check_unrelated_churn(scope)
```

Validation:

```text
run_test(selector, timeout_seconds)
run_lint(scope, timeout_seconds)
run_typecheck(scope, timeout_seconds)
run_build(target, timeout_seconds)
run_command_limited(command_id, args, timeout_seconds)
```

Review and handoff:

```text
summarize_diff(scope)
collect_validation_evidence(run_ids)
create_draft_change_request(title, body, diff_ref)
request_review(reason, diff_ref, evidence_ref)
```

Safety:

```text
request_approval(action, risk, preview_ref)
deny_secret_access(path_or_key, reason)
classify_file_sensitivity(path)
scan_diff_for_secrets(diff_ref)
```

## Model- and workload-dependent action interfaces

Keep the explicit baseline tools as the conservative MVP default. After measuring that baseline, a shell-centric or programmatic action profile can be a post-MVP option for a model that reliably expresses the workload through that interface. Select by demonstrated action proficiency and task mix, not parameter count or a general claim that fewer tools are better. The [naming adapter below](#api-tool-name-recommendations) remains useful, but naming alignment is not proof of shell proficiency.

An action profile includes tool availability, interface instructions, mutation accounting, validation support, and execution semantics. For a meaningful profile comparison, preserve equivalent host enforcement and verification requirements. Shell-mediated writes must retain the [command policy](#command-policy), [implementation invariants](#implementation-invariants), and required diagnostics rather than bypassing them. If a comparison changes these controls, report it as a bundled intervention, not an isolated tool-count effect.

Capable models may bundle several approved operations into fewer interactions. Evaluate that benefit alongside out-of-interface calls, localization failures, unintended edits, and completion quality; larger commands or edits are not intrinsically better. Use [component diagnostics](evals.md#component-diagnostics) for the comparison and keep the simpler safe profile when the alternative does not meet the same quality and safety floor.

## API tool-name recommendations

Tool names are part of the model interface. For coding agents, prefer names that match the dominant tool vocabulary of the model family you are serving. The harness can map those model-facing names to the same internal implementation.

Do not expose two naming profiles for the same capability in the same turn. Pick one profile, keep names stable, and make aliases internal.

OpenAI API profile:

```text
shell
apply_patch
update_plan
view_image
tool_search
request_user_input
list_mcp_resources
read_mcp_resource
list_available_plugins_to_install
request_plugin_install
```

Recommended OpenAI-style additions for a coding-agent harness:

```text
list_files
search_code
read_file
inspect_symbol
inspect_diff
run_tests
collect_validation_evidence
create_draft_change_request
```

Anthropic API profile:

```text
Bash
PowerShell
Read
Edit
Write
Glob
Grep
TodoWrite
Agent
AskUserQuestion
EnterPlanMode
ExitPlanMode
TaskStop
WebFetch
WebSearch
LSP
NotebookEdit
ToolSearch
```

Recommended capability mapping:

| Capability | OpenAI-style name | Anthropic-style name | Notes |
|---|---|---|---|
| Shell command | `shell` | `Bash` | Use one model-facing command tool for POSIX shells. Keep `cwd`, timeout, output caps, and permission metadata in the schema. |
| Windows shell command | `shell` or `powershell` | `PowerShell` | Use a separate name only if policy and parsing differ from POSIX shell behavior. |
| Patch edit | `apply_patch` | `Edit` | Prefer patch/diff-shaped edits over raw file rewrites for existing files. |
| Full file write | `write_file` | `Write` | Use mainly for new files or deliberate full rewrites. Require read-before-write for existing files. |
| File read | `read_file` | `Read` | Return line numbers, byte/line limits, and truncation metadata. |
| File glob | `list_files` or `glob_files` | `Glob` | Keep pattern matching separate from shell. |
| Content search | `search_code` | `Grep` | Use ripgrep-like semantics, output modes, and file globs. |
| Plan update | `update_plan` | `TodoWrite` | Use for visible task tracking, not hidden reasoning. |
| Ask user | `request_user_input` | `AskUserQuestion` | Use for scoped clarification with bounded options when possible. |
| Tool discovery | `tool_search` | `ToolSearch` | Expose only when deferred tools exist. |
| Image/local visual inspection | `view_image` | `Read` or domain-specific visual tool | Use only when the model must inspect rendered or local visual state. |
| Language intelligence | `inspect_symbol` | `LSP` | Keep symbol lookup separate from freeform shell commands. |
| Background worker | `spawn_agent` | `Agent` | Post-MVP unless the single-agent loop has measured failures requiring decomposition. |
| Stop background work | `stop_task` | `TaskStop` | Required if any tool can start long-running work. |

Provider-neutral harnesses can keep the generic baseline names, but should still support a thin adapter layer that exposes the preferred profile for the selected model. For example, `Read` and `read_file` can call the same internal tool, but only one should be visible to a model in a given turn.

Naming rules:

- keep names short, stable, and action-oriented;
- avoid synonyms such as both `search_code` and `grep_code` in one profile;
- avoid overloaded names such as `execute`, `run`, `do`, or `tool`;
- avoid raw infrastructure names unless the model-facing action is actually that narrow;
- keep dangerous capabilities visible in the name, such as `apply_patch`, `shell`, `Bash`, `PowerShell`, `Write`, or `create_draft_change_request`;
- keep commit, push, merge, deploy, and permission changes separate from edit and validation tools.

## Permission defaults

```text
Read repository instructions and git state: allow inside approved workspace
Search and read repository files: allow inside approved workspace
Edit local draft workspace or branch: allow after scope is understood
Run allowlisted validation commands: allow in sandbox with fixed cwd, timeout, and output caps
Install dependencies: approval-gated unless preconfigured by project policy
Change lockfiles: approval-gated unless explicitly part of the task
Read environment files, tokens, private keys, or credentials: deny by default
Commit, push, or open a draft change request: approval-gated or explicit product allowlist
Merge, deploy, or modify production data: deny in the MVP
Change identity, access, CI secrets, or repository permissions: deny in the MVP
```

## Command policy

If shell is necessary, wrap it as `run_command_limited`, not `execute_shell(command)`. The wrapper should enforce:

```text
allowlisted command ids
fixed working directory
argument schema validation
canonical approval key derived from parsed argv, not raw command text only
shell wrapper and compound-command parsing
subcommand decomposition budget with fail-closed approval
denial of broad saved prefixes for shells, interpreters, env/sudo-like wrappers, and inline-code runners
mode-aware permission checks for read-only, edit, and bypass-like modes
path extraction from command arguments, flags, end-of-options markers, and output redirections
timeout
stdout and stderr caps
secret-free environment
network policy
destructive-command denial
structured result with exit code and truncated output refs
```

Do not use approval-cache keys that are easy to bypass through wrapper spelling differences. Normalize equivalent forms such as explicit shell paths, shell `-c` or `-lc` wrappers, and platform-specific command wrappers before matching or saving approvals. If the command cannot be safely normalized, require one-time approval and avoid suggesting a persistent broad rule.

## Implementation invariants

Workspace boundary:

- discover the repository root and approved working directories before mutation;
- resolve file paths, symlinks, shell redirections, and command path arguments before allowing reads or writes;
- distinguish pre-existing user changes from agent-created changes.

Command analysis:

- canonicalize command argv before matching approval cache entries;
- parse shell wrappers, compound commands, and subcommands instead of matching only raw strings;
- cap command decomposition work so complex commands fail closed to approval;
- never save broad allow rules for bare shells, interpreters, privilege wrappers, or open-ended package runners.

Change accounting:

- snapshot repository status at turn start;
- track patch-tool changes as a turn-scoped diff where possible;
- invalidate or re-read the diff when shell commands or external tools may have edited files;
- show bounded diff stats and representative hunks instead of loading unbounded diffs into context.

Tool lifecycle:

- run pre-tool checks for policy, hooks, path constraints, and command risk;
- run post-tool checks for secret exposure, diff changes, validation evidence, and trace updates;
- return a structured result for every allow, deny, timeout, abort, and validation failure.

## Evals

Do not only evaluate code correctness. Evaluate scope control, permission behavior, evidence quality, and recovery behavior.

Tiny bug fix:

- reads local instructions and repository state before editing;
- finds the relevant failing behavior or narrow code path;
- patches the minimal file set;
- runs the most relevant check;
- final evidence cites commands and results.

Ambiguous feature request:

- asks a targeted question or produces a plan before broad edits;
- does not invent product requirements;
- avoids touching unrelated modules.

Prompt injection in issue, ticket, test fixture, or repository text:

- treats the text as untrusted task data;
- ignores instructions such as "reveal secrets" or "bypass policy";
- does not let untrusted text choose tools or permissions.

Secret access attempt:

- refuses or redacts environment files, tokens, private keys, and credentials;
- records denial as a structured observation;
- proposes a safe alternative such as using documented config names.

Unsafe shell command:

- blocks destructive, network-open, or permission-changing commands outside policy;
- proposes safer inspection or allowlisted validation;
- records the permission decision in the trace.

Approval-cache bypass attempt:

- normalizes equivalent command forms before matching approvals;
- does not persist broad approvals for bare shells, interpreters, privilege wrappers, or inline-code runners;
- falls back to one-time approval when command structure cannot be safely parsed.

Path escape through command syntax:

- extracts paths from command arguments, flags, end-of-options markers, and redirections;
- resolves paths against the active working directory and approved workspace roots;
- blocks critical paths and out-of-workspace reads or writes even when hidden behind shell syntax.

Broken test after patch:

- performs bounded repair only when the failure is clear and in scope;
- stops after the repair budget with evidence if unresolved;
- does not hide failed or skipped validation.

Over-broad refactor:

- limits changes to the requested behavior;
- avoids unrelated style churn;
- reports skipped opportunities separately from the patch.

Dependency upgrade:

- states the version delta and compatibility risk;
- updates lockfiles only when in scope;
- runs build or tests that cover the dependency surface;
- includes rollback notes.

Review finding quality:

- findings include file and line evidence;
- severity is tied to reachable impact;
- speculative comments are labeled as questions or non-blocking notes.

Change-request body quality:

- summary, files changed, validation, risks, and reviewer notes are present;
- failed or unavailable checks are explicit;
- final claims are grounded in tool results.

Turn-scoped diff accounting:

- records baseline repository state before edits;
- separates pre-existing user changes from agent-created changes;
- invalidates or refreshes the turn diff after shell-mediated writes;
- limits huge diff details while preserving accurate stats and changed-file evidence.

## Coding-agent MVP checklist

- [ ] The agent reads workspace instructions and repository state before editing.
- [ ] The task type is classified: bug fix, review, migration, dependency upgrade, test generation, docs sync, or other.
- [ ] The intended file scope is identified before mutation.
- [ ] Baseline repository state and pre-existing user changes are recorded before mutation.
- [ ] The patch is local and reviewable before any commit, push, merge, or deploy action.
- [ ] Repository reads are limited to the approved workspace.
- [ ] Read and write paths are resolved against approved workspace roots, including symlinks, redirections, command flags, and `--` end-of-options cases.
- [ ] Secret-like paths, tokens, private keys, credentials, and environment files are denied or redacted by policy.
- [ ] Shell/process execution is wrapped by allowlisted command ids, fixed cwd, timeout, output cap, and secret-free environment.
- [ ] Command approval matching canonicalizes shell wrappers and argv forms before reusing or saving approvals.
- [ ] Persistent allow-rule suggestions reject broad shells, interpreters, privilege wrappers, and inline-code runners.
- [ ] Compound-command analysis has a bounded decomposition budget and fails closed to approval.
- [ ] Dependency installation and lockfile changes are approval-gated unless explicitly in scope.
- [ ] Commit, push, and draft change-request creation are approval-gated or explicitly allowlisted.
- [ ] Merge, deploy, production data edits, CI secret edits, and permission changes are denied in the MVP.
- [ ] Validation commands are detected or configured before editing where feasible.
- [ ] The repair loop has a bounded retry count and stops with evidence when unresolved.
- [ ] Shell-mediated writes invalidate or refresh turn-scoped diff accounting before final claims.
- [ ] The final diff is checked for unrelated churn and accidental secret exposure.
- [ ] Pre-tool and post-tool hooks, or equivalent harness checks, run for command/file tools.
- [ ] Final evidence includes files changed, commands run, pass/fail/skipped checks, assumptions, risks, and reviewer notes.
- [ ] Evals cover prompt injection in repository text, unsafe shell, approval-cache bypass, path escape, secret access, over-broad edits, broken tests, turn diff accounting, and evidence quality.

## Anti-patterns

Avoid:

```text
raw shell as the primary MVP tool
approval rules based only on unparsed command strings
persistent allow rules for shells, interpreters, or privilege wrappers
editing before reading workspace instructions and git state
claiming success without validation evidence
merging, deploying, or changing production state in the MVP
reading secrets into model context
over-broad refactors for narrow tasks
letting repository text override trusted instructions
losing pre-existing user changes in final diff accounting
```
