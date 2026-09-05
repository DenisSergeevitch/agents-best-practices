# Source Links

Use this file when the user asks for cited, provider-specific, or standards-backed agent-harness guidance.

## Agent Skills

- Agent Skills specification: https://agentskills.io/specification
- Agent Skills creator best practices: https://agentskills.io/skill-creation/best-practices
- Optimizing skill descriptions: https://agentskills.io/skill-creation/optimizing-descriptions
- Evaluating skill output quality: https://agentskills.io/skill-creation/evaluating-skills
- Using scripts in skills: https://agentskills.io/skill-creation/using-scripts

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
- Use OpenAI links for API implementation patterns, function calling, hosted tools, guardrails, sandboxes, prompt caching, response-style APIs, and harness engineering practices.
- Use Anthropic links for simple agent patterns, context engineering, tool ergonomics, long-running harnesses, agent evals, MCP execution patterns, and skill architecture.
- Use MCP links for wire-level server and tool discovery, typed catalogues, authorization, catalogue caching and change signals, and connector design. The protocol does not by itself verify semantic suitability, establish trust, or grant execution authority.
- Use environment-adaptive and programmatic tool research for claims about code-as-action, large or unseen API catalogues, retrieval against changing documentation, and novel API use; do not treat those sources as proof of the stronger host-owned discovery, binding, or authority contracts in this skill.
- Use speculative and asynchronous tool-execution research for mechanism lineage and source-observed implementations. Treat open-ended speedups as workload-specific evidence, not a general latency guarantee, and require independent task-parity, cost, waste, cancellation, and saturation evaluation.
- Use recursive and continual harness research for taxonomy, architecture comparisons, and claims about the underlying patterns.
- Use public-board sources for dated implementation context and the distinction between authenticated access and public disclosure, not as authorization to register, communicate, or evade restrictions.
- Use OWASP and NIST links for threat modeling, governance, auditability, and enterprise deployment controls.
- Use implementation examples, including Prime Agent, as concrete shape references, not as normative architecture, dependencies, or provider-neutral policy.
