I build agents that take real actions against real systems, which mostly means
deciding what an agent must not be able to do and then enforcing it somewhere the
model cannot reach.

### [signed-email-agent](https://github.com/Eman-Yousaf/signed-email-agent)

An autonomous agent for a 4-player adversarial benchmark where LLM agents negotiate
over email and exchange RSA-PSS signed messages. Each agent may sign only for certain
others, and the opponents are LLMs writing persuasive email directly into your agent's
context to talk it into signing anyway.

Signing is overridden at a single chokepoint, so every signature — including ones the
model initiates through its own tool calls — passes a verified authorization check
first. The guard is structural rather than advisory because the earlier version failed
exactly once in the way that matters: the LLM fallback signed for an agent the
deterministic path had already declined in the same round.

From round 2 the authorization list stops naming agents and paraphrases them instead,
so the agent has to resolve identity from description against message history at
runtime. That resolver runs at temperature 0 and declines on ambiguity, timeout, or a
malformed response — declining is free, a wrong signature is not.

The decision logic has 22 tests covering the guard — that a peer's claim of
authorization grants nothing, that the LLM fallback cannot sign for an unauthorized
agent, that a resolver error or low-confidence match declines. They run offline with no
network, LLM or server: `pytest test_agent_logic.py`.

### [datahub-incident-copilot](https://github.com/Eman-Yousaf/datahub-incident-copilot)

A ReAct agent (LangGraph + MCP) that takes a one-line incident report — "order counts
on our dashboards look wrong" — and investigates it live against DataHub's real lineage
graph, deciding for itself whether to walk further upstream or stop, then computing the
downstream blast radius before it touches anything.

Write-backs are gated on evidence rather than on the model's own say-so: confidence and
severity are computed in plain Python from which evidence items were actually confirmed
via tool calls, and low confidence blocks the mutation outright and routes to human
review. Every successful write is re-read from the catalog afterward, so a trace shows
the tag actually landed instead of asking you to trust a bare `success: true`.

### [GAMING-ARENA-WITH-AI](https://github.com/Eman-Yousaf/GAMING-ARENA-WITH-AI)

A C++17 tournament server: minimax tic-tac-toe engine behind token auth and role-based
access, matchmaking, and a REST API on Crow/ASIO. CI builds the documented
configuration on every push, so the build instructions in the README are the ones that
are known to work.

The `src/` tree and the GoogleTest suite target an unfinished modular refactor whose
headers do not exist yet. Both are excluded from the build and the README says so,
rather than shipping a build that breaks on clone.

### [Filer_ai](https://github.com/Eman-Yousaf/Filer_ai)

A local-first file triage agent: a Python filesystem watcher, an Obsidian vault as the
human-readable interface, and a Claude Code skill as the executor. No server, no
database, no cloud.

The folder layout *is* the state — items carry `status` in their frontmatter instead of
living in a separate queue, so there are not two sources of truth to drift apart. Work
waits visibly in `/Needs_Action` where a person can change it before anything is
finalised, and behaviour is configured in prose: the rules live in a handbook file, so
changing how it works means editing English rather than Python.

---

Also here: [AI_employee](https://github.com/Eman-Yousaf/AI_employee), an agentic
assistant built from nine Claude Code skills that watches Gmail/WhatsApp/LinkedIn and
routes anything consequential through an approval step;
[house-price-prediction](https://github.com/Eman-Yousaf/house-price-prediction), a
linear-regression pipeline documented end to end from imputation through to held-out
evaluation; and coursework in C++, MySQL and networking.

[LinkedIn](https://www.linkedin.com/in/eman-yousaf96)
