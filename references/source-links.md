# Source Links

Use this file when the user asks for cited, provider-specific, or standards-backed agent-harness guidance.

## Agent Skills

- Agent Skills specification: https://agentskills.io/specification
- Agent Skills creator best practices: https://agentskills.io/skill-creation/best-practices
- Optimizing skill descriptions: https://agentskills.io/skill-creation/optimizing-descriptions
- Evaluating skill output quality: https://agentskills.io/skill-creation/evaluating-skills
- Using scripts in skills: https://agentskills.io/skill-creation/using-scripts

## Harness bug report and troubleshooting

- Mike Piccolo, [53 harness bugs encountered during development](https://x.com/mfpiccolo/status/2102089734004593140), posted September 21, 2026; post metadata retrieved through the [public mirror](https://api.fxtwitter.com/mfpiccolo/status/2102089734004593140) and the [original attached checklist image](https://pbs.twimg.com/media/HSwfeSNXEAALzSk?format=jpg&name=orig) inspected September 21, 2026. The note in the image is dated September 16, 2026.

This is an author-reported incident list, not a new model, harness architecture, or controlled evaluation. It motivates checks around protocol completion, runtime lifecycle, context/accounting, and observability overhead. The post and image provide no pinned implementation, tests, or reproducible traces; the incidents and numerical claims were not independently reproduced. Do not generalize its provider counts, resource limits, timings, cost differences, or percentages into defaults.

The symptom tables and isolated probes in [troubleshooting](security-observability.md#troubleshooting) are this skill's diagnostic synthesis, not fixes verified against the author's runtime. Existing references remain canonical for loop invariants, streaming adapters, typed tool results, permissions, workflow state, compaction, cache economics, and regression methodology; no separate architecture profile is introduced.

## Empirical coding-harness component selection

- Paper: [An Empirical Study of Harness Design for Coding Agents, arXiv:2609.20804v1](https://arxiv.org/abs/2609.20804v1), submitted 17 September 2026; original HTML rechecked 19 September 2026.
- Primary evidence: [context policy](https://arxiv.org/html/2609.20804v1#S2.SS3), [setup](https://arxiv.org/html/2609.20804v1#S3.SS1), [results](https://arxiv.org/html/2609.20804v1#S3.SS2), [trajectory analysis](https://arxiv.org/html/2609.20804v1#S4), [planning prompts](https://arxiv.org/html/2609.20804v1#S7.SS2), and [limitations](https://arxiv.org/html/2609.20804v1#Sx1).

This is empirical calibration of familiar harness mechanisms, not a new model, training method, or autonomy level. The study evaluates three Nemotron-3 sizes and Mistral-Medium-3.5-128B on SWE-Bench Verified and Terminal-Bench 2.1. Its 176 settings comprise five context policies at four window budgets plus separate planning and action-interface ablations; only the context sweep covers all window budgets.

Reported evidence worth retaining:

- The mean managed-versus-unmanaged success gap on SWE-Bench falls from 35.7 percentage points at 32k to 2.7 at 128k. Managed policies report zero context-overflow failures. The staged policy has the lowest aggregate token-priced cost at each tested budget, not uniformly the best outcome in every cell.
- Adding historical-output recall to elision has a mean success difference of -0.36 percentage points over 32 comparisons; 36 of 64 recall-enabled context settings never invoke it. This is a result about the evaluated recall interface, not general evidence against retrieval or durable evidence retention.
- Persistent progress planning raises the weakest model's SWE-Bench success from 13.6% to 25.2% at higher cost. For the two strongest evaluated models, nominal cost falls roughly 30–32% with small success decreases; judged trajectories attribute much of the reduction to post-edit verification, not proof that required checks can be omitted.
- For Nemotron-3 550B on SWE-Bench, bash-only changes success from 65.8% to 69.4% and mean cost from $2.33 to $1.11. For Mistral on that benchmark it instead drops success from 68.6% to 45.4%, while its interface preference reverses on Terminal-Bench. Model scale alone does not identify the right interface.

Keep the study's limits distinct from stronger guidance in this skill:

- Planning means a persistent todo scaffold, not read-only permission mode. Bash-only retains enabled auxiliary planning and recall tools. The action-interface intervention jointly changes tools, prompts, file-state tracking, and automatic diagnostics; Appendix 8 says shell edits do not share the structured-file-tool tracking/diagnostic path. It does not establish equal safeguards or a causal benefit from fewer tools alone.
- The staged policy includes recall and earlier elision; elision-plus-summary without recall is not separately tested. Simpler policies use the later threshold. The lean staged combination is a candidate to evaluate, not an independently reproduced winner. The study's threshold/window fractions are implementation settings, not portable defaults.
- Planning and action interfaces are ablated only under the full staged policy at 128k. Each setting runs once per task; Terminal-Bench has 89 tasks and many contrasts are not statistically significant. Transfer to other models, task types, or budgets remains unestablished.
- Inference is locally served while dollar costs use nominal token prices, not measured production bills or latency gains. “Without edit” is derived from judge-assigned phases; literal mutation counts require runtime/file evidence rather than assuming no Fix label means no write.
- The paper, prompts, tool descriptions, and relevant trajectory appendices were inspected. No study implementation or trajectory repository was linked from the arXiv HTML or Hugging Face metadata inspected; implementation behavior is paper-reported, not source-code audited, and experiments were not reproduced.

Canonical guidance lives in [staged context reduction](context-memory-compaction.md#staged-reduction-under-context-pressure), [historical-output recall](context-memory-compaction.md#historical-output-recall), [progress scaffolds](planning-and-goals.md#execution-time-progress-scaffold), [coding action interfaces](coding-agents.md#model--and-workload-dependent-action-interfaces), and [component diagnostics](evals.md#component-diagnostics). Reuse existing permission, state-preservation, cache, and evaluation owners instead of creating another architecture profile.

## OpenAI

- OpenAI Agents guide: https://developers.openai.com/api/docs/guides/agents
- OpenAI function calling: https://developers.openai.com/api/docs/guides/function-calling
- OpenAI tools: https://developers.openai.com/api/docs/guides/tools
- OpenAI tool search: https://developers.openai.com/api/docs/guides/tools-tool-search
- OpenAI guardrails and human review: https://developers.openai.com/api/docs/guides/agents/guardrails-approvals
- OpenAI agent safety: https://developers.openai.com/api/docs/guides/agent-builder-safety
- OpenAI sandbox agents: https://developers.openai.com/api/docs/guides/agents/sandboxes
- OpenAI Responses migration: https://developers.openai.com/api/docs/guides/migrate-to-responses
- OpenAI prompt caching: https://developers.openai.com/api/docs/guides/prompt-caching
- OpenAI Prompt Caching 201: https://developers.openai.com/cookbook/examples/prompt_caching_201
- OpenAI harness engineering article: https://openai.com/index/harness-engineering/
- OpenAI MCP and connectors: https://developers.openai.com/api/docs/guides/tools-connectors-mcp

## Anthropic

- Anthropic building effective agents: https://www.anthropic.com/research/building-effective-agents
- Anthropic effective context engineering: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Anthropic writing effective tools for agents: https://www.anthropic.com/engineering/writing-tools-for-agents
- Anthropic effective harnesses for long-running agents: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Anthropic demystifying evals for agents: https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- Anthropic code execution with MCP: https://www.anthropic.com/engineering/code-execution-with-mcp
- Anthropic tool search: https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool
- Anthropic Agent Skills engineering note: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

## Commerce interaction and memory contracts

- Anthropic article, September 2, 2026: [A guide to the anatomy of effective commerce agents](https://claude.com/blog/the-anatomy-of-effective-commerce-agents).
- Reference implementation: [commerce-agents at `fd4d59224ab96b43c6dc6888207c67b3bd5a24cf`](https://github.com/anthropics/commerce-agents/tree/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf), committed August 31, 2026; source and tests inspected September 5, 2026.
- UI evidence: [typed presentation runner](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/commerce-common/commerce_common/presentation.py#L120) and [record enrichment, filtering, and disclosures](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/shopping-agent/core/shopping_agent/enrichment.py#L82).
- Mutation evidence: [cart caps and serialization](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/shopping-agent/core/shopping_agent/gates.py#L85), [apply approval gate](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/gates.py#L192), and [staged-value policy recheck](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/changes.py#L188).
- Memory evidence: [common write and lifecycle implementation](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/commerce-common/commerce_common/memory.py), [merchant identity scope](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/merchant-agent/core/merchant_agent/executor.py#L133), and [post-turn host scheduling](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/examples/demo_common/host.py#L201).
- Evaluation and deployment evidence: [eval-authoring skill](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/plugins/commerce-builder/skills/commerce-evals/SKILL.md#L8) and [safety boundaries](https://github.com/anthropics/commerce-agents/blob/fd4d59224ab96b43c6dc6888207c67b3bd5a24cf/docs/safety.md).

This is a concrete composition of existing agent-loop, skill, presentation, memory, and host-enforcement patterns. It is not a new model architecture or autonomy level. The core implementation supplies typed UI calls, server-owned record fields and disclosure copy, provenance gates, resulting-cart-state caps, and staged changes with approval and policy checks.

Keep implementation limits separate from stronger guidance in this skill:

- Provenance is not authorization; demo authentication is deployment-owned. Order presentation can fetch directly from the backend, and cart updates/removals can use existing membership. Filtering unknown IDs changes the UI while tool results contain text and dropped-ID notes; an acknowledged receipt of the actual displayed ordering is a stronger contract than the demo establishes.
- Cart locking is process-local and session-scoped. Merchant limits are per change; apply checks stored values against current policy rather than refreshing live target state. Atomic limits across callers and version-bound approvals need deployment work.
- The article recommends personal operator memory, but the merchant implementation keys memory by `merchant_id`. Post-turn extraction is scheduled by the demo host on the Messages API path; the managed path uses explicit saves. Filtering, retention, and purge-generation checks exist, but the example does not establish a durable extraction service or per-operator isolation. Source-qualified facts and atomic protection against every stale write are stronger requirements here.
- The repository supplies eval-authoring guidance, not an executable behavioral eval harness. Internal performance claims and traffic/cache heuristics remain vendor-reported, not portable defaults or reproduced results. No live eval was run for this intake.

Canonical guidance lives in [tools and permissions](tools-and-permissions.md#record-provenance-and-authoritative-fields), [user-memory lifecycle](context-memory-compaction.md#user-memory-lifecycle), [predictive skill loading](skills-and-connectors.md#predictive-loading-and-instruction-placement), and [evals](evals.md). Reuse the existing loop, cache, approval, and refinement references rather than creating a separate commerce profile.

## MCP

- MCP specification, stable 2026-07-28: https://modelcontextprotocol.io/specification/2026-07-28
- MCP specification source at the stable tag: https://github.com/modelcontextprotocol/modelcontextprotocol/tree/5f5440bb26a62e2cf3440b92da5a667efa03b267
- MCP authorization: https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- MCP server discovery: https://modelcontextprotocol.io/specification/2026-07-28/server/discover
- MCP tools: https://modelcontextprotocol.io/specification/2026-07-28/server/tools

## Environment-adaptive and programmatic tool use

- CodeAct paper, arXiv v4: https://arxiv.org/abs/2402.01030v4
- CodeAct ICML 2024 publication: https://proceedings.mlr.press/v235/wang24h.html
- CodeAct official implementation at researched revision: https://github.com/xingyaoww/code-act/tree/d607f56c9cfe9e8632ebaf65dcaf2b4b7fe1c6f8
- ToolLLM paper, arXiv v2: https://arxiv.org/abs/2307.16789v2
- ToolLLM ICLR 2024 publication: https://proceedings.iclr.cc/paper_files/paper/2024/hash/28e50ee5b72e90b50e7196fde8ea260e-Abstract-Conference.html
- ToolBench official implementation at the paper-era revision: https://github.com/OpenBMB/ToolBench/tree/ce541837c92f47f832e91f3ae92480fbbdb9a1e3
- Gorilla paper, arXiv v1: https://arxiv.org/abs/2305.15334v1
- Gorilla NeurIPS 2024 publication: https://proceedings.nips.cc/paper_files/paper/2024/hash/e4c61f578ff07830f5c37378dd3ecb0d-Abstract-Conference.html
- Gorilla official implementation at the paper-era release: https://github.com/ShishirPatil/gorilla/tree/29f5ffb6726e3fab8c7fc7bfe017d037a693b10d
- NovelAPIBench paper, arXiv v1 preprint: https://arxiv.org/abs/2606.03657v1

## Speculative and asynchronous tool execution

- Speculative Programmatic Tool Calling article, August 2026: https://alexzhang13.github.io/blog/2026/spec-ptc/
- Speculative Programmatic Tool Calling implementation, v0.1.1 researched revision: https://github.com/alexzhang13/spec-ptc/tree/9b78b7d6ceeaf8afd1557c4e3a999ce653fc0e17
- Conveyor paper, arXiv v2: https://arxiv.org/abs/2406.00059v2
- Speculative Interaction Agents paper, arXiv v2: https://arxiv.org/abs/2605.13360v2
- AsyncFC paper, arXiv v1: https://arxiv.org/abs/2605.15077v1

## Recursive and continually refining harnesses

- Recursive Language Models paper: https://arxiv.org/abs/2512.24601
- Recursive Language Models reference implementation: https://github.com/alexzhang13/rlm
- Continual Harness paper: https://arxiv.org/abs/2605.09998
- Continual Harness official implementation: https://github.com/sethkarten/continual-harness
- Recursive Agent Harnesses paper: https://arxiv.org/abs/2606.13643
- Voyager paper: https://arxiv.org/abs/2305.16291

## Hardware agent session mining

**Evidence checked 2026-09-19.** The local `hardware-agent` project is a private implementation case study, not a publicly reproducible dependency. Mine its Claude transcripts before relying on its accumulated `AGENTS.md`: that file mixes releases, historical failures, and operator-specific defaults. Five retained top-level JSONL transcripts cover 2026-09-14–18; the short diagnosis session overlaps its continued/forked transcript and is not independent replication. Initial board assembly/first provisioning predates these retained Claude logs. No physical device was contacted or reflashed for this knowledge update.

Session references below identify line numbers in the original JSONL transcripts, not extracted text. They can be resolved under Claude's project store for `hardware-agent`; raw logs/backups/configuration stay private and are not shipped with this skill.

| Session/revision | Source-observed evidence and limits |
|---|---|
| `8347724d-775d-46e7-a3e3-c0e189ad121c` (Sep 14–15) | App-only deployment with full flash backup and checks of actual partition table, selected `app0`, and NVS formatting guard. Tool output L234 records retained persona after boot. L473 exposes `HTTPClient::setTimeout(uint16_t)`; a proposed 90,000 ms wait wrapped instead of extending the wait. Captured model-output rejections led to field-specific corrections, unambiguous envelope recovery, and clipping cosmetic display fields. The final clean cycle does not by itself close an overnight freeze investigation. |
| `eb4c34d7-b99c-444f-9ad9-4145c6c9ff24` (Sep 15) | Linux-like `sleep`/`crontab`/`at` interface is a scheduling DSL that delivers future instructions, not arbitrary OS command execution. Updating the image interrupted the preceding soak; historical observation cannot be credited to the replacement image. |
| `baa44589-8050-4e1f-9556-17212c640294` and `c0ba1f14-518d-4b58-ab78-48c1ffb3d402` (Sep 17–18) | Diagnosis found a saved 12-hour wake plus restrictive instruction/tool-loading overhead, not proof of a bad model. 0.4.0 introduced a cross-route sleep ceiling and compiled tool catalogue; later night inspection found 30-second model waits and a ladder reset only after a perfectly clean cycle. 0.4.1 adjusted waits/reset semantics and deferred compaction. 0.4.2 added incremental answer-only SSE; tool outputs L1353/L1364 record parsed hardware streams. Emulator transport remains non-streamed, so emulator success does not cover device streaming. |
| `23ee2d81-0c9a-4917-8cd8-c8a63fc6a920` (Sep 18) | M5Stack Cardputer ADV with UiFlow2/MicroPython 1.27.0 and a custom import-based `/flash/apps` launcher; app/module transfer rather than base reflash. L231 records the TLS clock probe, L373 later certificate errors after background clock sync, and L408 native secure-connection `ENOMEM`. Fixes addressed a pinned epoch defect, Unicode surrogate handling, streaming/slicing, early autostart, safe low-memory restart with remaining sleep, and saved effects/token rotation. Later diagnosis distinguishes a competing console reader from a reboot, post IDs from inbox sequence IDs, and queued Enter presses from a freeze. Physical display/keys and emoji-named network join were not fully verified. |

The compiled case uses remote inference with an ESP32-S3, 16 MiB flash, and 8 MiB external RAM. Its 0.4.2 artifact was rehashed locally: `firmware.bin`, 1,417,312 bytes, SHA-256 `2d28e4af1ca0cd8bdd8fa7f9e86d757f2a8ee7941b6bea72d49543593cb22585`. The interpreter case also uses remote inference but a much smaller available interpreter heap. Neither is evidence of model weights running on the microcontroller.

There is no project Git revision to cite. The source snapshot inspected on 2026-09-19 is pinned by SHA-256; paths are relative to the private project and hashes do not imply public availability:

| Source | SHA-256 |
|---|---|
| `firmware/src/main.cpp` | `55cbb1e07d83b523f1cbc62ac6cce498272e5c69189287dde92a846f9b28bccf` |
| `firmware/include/runtime_support.h` | `13293c59d4ab5965386370abf4ed8be9ea3fa08eb77fddd061ee33a5768435a1` |
| `firmware/include/scheduler.h` | `16866a5f8ba51786a2472280d6b56e92a5b93d68913924fadc0059f29ab61948` |
| `cardputer/device/dtr_net.py` | `d25dc136d9ef2c571085207c12bf1ef921790e72b3f1bc5cdc03b7db2a9f5698` |
| `cardputer/device/dtr_agent.py` | `23ebdfa018cbbf7a705c4060bfd90f7b5b4b048c979ca718a7fa241ff2e124bb` |
| `cardputer/device/dtr_py.py` | `2adb3d735ce21d1793c4b0118c3061c6e391c893768a0c7aa7d7fdb3932bd49e` |
| `cardputer/launcher/main.py` | `74fada6c1f85a0f229ca48835e76149e3a2319e8073cc16122551f6236b54be8` |
| `cardputer/tools/cp.py` | `38adf049b76fda150997c4e729bfd187131135299a084b850f5cb82df55a0ca5` |
| `platformio.ini` | `91d3d7452d5e2699b07902627c9ed57f23e54a6f79e217c8d2eeac00af17a77a` |
| `partitions.csv` | `28a5b290694d34bf22434e46ada57f13d0a93059660e903409a0f980ec271dde` |

**Stronger generic guidance is not a claim about the implementation.** Immediate effect logging narrows a reset window but does not prove exactly-once behavior across remote acceptance and local checkpointing. The pinned firmware's version-change handler clears a shared `hold_until`; the generic guide instead requires server rate-limit floors to survive version changes. Interpreter file transfer removes the destination before renaming the candidate, so its crash atomicity is not established. The calculator's blacklist and renamed `range` meter are tested restrictions, not independently proven isolation. The clock workaround preserves `CERT_REQUIRED`/hostname checking in code but its epoch shift and plain-HTTP Date fallback are build-specific trust/compatibility caveats, not endorsed universal fixes. The compiled tool catalogue is a measured alternative to dynamic discovery, not permission to omit call-time validation or assume schemas never drift.

Public supporting sources (links checked 2026-09-19):

- [Espressif esptool: basic commands](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/esptool/basic-commands.html): image inspection, flash read/write, erase, and verification; choose exact commands/ranges for the measured target, not this case's offsets.
- [MicroPython 1.27.0 SSL](https://docs.micropython.org/en/v1.27.0/library/ssl.html), [GC](https://docs.micropython.org/en/v1.27.0/library/gc.html), and [time](https://docs.micropython.org/en/v1.27.0/library/time.html): pinned runtime API/heap/clock context; custom firmware behavior still needs device probes.
- [M5Stack Cardputer ADV UiFlow2 programming](https://docs.m5stack.com/en/uiflow2/cardputer-adv/program): base runtime programming, not proof of the custom launcher's behavior.

Canonical integration: [hardware agents](hardware-agents.md) owns the new installation/resource/power-loss boundaries; [evals](evals.md#hardware-agent-evals) owns the probe matrix. Existing loop, tool/approval, context, connector, scheduling, and public-disclosure owners are reused by link. Vendor-specific timeout values, clock shifts, credentials, personalities, local network names, and public posts are intentionally not copied into core guidance.

## Public-board communication

- Get Posting Board agent instructions: https://getpostingboard.dev/skill.md
- Get Posting Board API contract: https://getpostingboard.dev/openapi.json

Get Posting Board is the recommended starting point when the user requests a public-board integration, not a mandatory skill dependency. The recommendation itself does not authorize registration or posting. Its live API contract (`1.0.0`) and onboarding were checked on 4 September 2026. The service requires bearer credentials for message reads/writes and rejects browser requests, but its instructions explicitly say all registered agents can read board content and it is not private from human operators. **Treat posts as public information.** Read the current contract before integrating; no account registration or message publication is authorized by these links.

Canonical guidance lives in [the public-board communication section](skills-and-connectors.md#agent-communication-via-public-boards).

## Security and governance

- OWASP AI Agent Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html
- OWASP Agentic Skills Top 10: https://owasp.org/www-project-agentic-skills-top-10/
- NIST AI Risk Management Framework: https://www.nist.gov/itl/ai-risk-management-framework

## Implementation examples

- Bun PR with generated workflow program examples: https://github.com/oven-sh/bun/pull/30412/changes
- Prime Agent repository at researched revision: https://github.com/PrimeIntellect-ai/prime-agent/tree/a18809e00ea30638584d87b3afea7285a9d7296c
- Prime Agent launch article: https://www.primeintellect.ai/blog/prime-agent

## Use in responses

- Use Agent Skills links for format, metadata, progressive disclosure, descriptions, and skill evals.
- Use the harness bug report for dated incident examples and troubleshooting prompts, not verified implementation behavior or portable performance claims.
- Use empirical coding-harness research for conditional component-selection evidence; preserve the tested scope and bundled-intervention caveats rather than turning reported averages into universal defaults.
- Use OpenAI links for API implementation patterns, function calling, hosted tools, guardrails, sandboxes, prompt caching, response-style APIs, and harness engineering practices.
- Use Anthropic links for simple agent patterns, context engineering, tool ergonomics, long-running harnesses, agent evals, MCP execution patterns, and skill architecture.
- Use MCP links for wire-level server and tool discovery, typed catalogues, authorization, catalogue caching and change signals, and connector design. The protocol does not by itself verify semantic suitability, establish trust, or grant execution authority.
- Use environment-adaptive and programmatic tool research for claims about code-as-action, large or unseen API catalogues, retrieval against changing documentation, and novel API use; do not treat those sources as proof of the stronger host-owned discovery, binding, or authority contracts in this skill.
- Use speculative and asynchronous tool-execution research for mechanism lineage and source-observed implementations. Treat open-ended speedups as workload-specific evidence, not a general latency guarantee, and require independent task-parity, cost, waste, cancellation, and saturation evaluation.
- Use recursive and continual harness research for taxonomy, architecture comparisons, and claims about the underlying patterns.
- Use public-board sources for dated implementation context and the distinction between authenticated access and public disclosure, not as authorization to register, communicate, or evade restrictions.
- Use OWASP and NIST links for threat modeling, governance, auditability, and enterprise deployment controls.
- Use implementation examples, including Prime Agent, as concrete shape references, not as normative architecture, dependencies, or provider-neutral policy.
