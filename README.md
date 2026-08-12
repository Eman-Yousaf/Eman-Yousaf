The check that decides whether a language model gets to sign something:

```python
def sign_and_respond(self, to_agent, message_to_sign, response_body, subject="Signed Message"):
    ok, reason = self._authorize(to_agent, message_to_sign)
    if not ok:
        self.log.warning(f"BLOCKED sign attempt for {to_agent} | reason={reason} ...")
        ...  # reply saying we won't sign it
        return {"success": False, "error": "not authorized - refused"}
    return super().sign_and_respond(to_agent, message_to_sign, response_body, subject)
```

They exist because of a specific failure. In an adversarial benchmark where four LLM
agents negotiate over email and exchange RSA-PSS signed messages, the model was never
given the authorization list — so when an opponent wrote a convincing enough request,
the LLM fallback signed for an agent the deterministic path had declined *in the same
round*. Checking at each call site would have meant trusting every future caller,
including the model. Overriding the method means there is no call site left to forget.

### Where the guard lives decides whether it holds

In [signed-email-agent](https://github.com/Eman-Yousaf/signed-email-agent), every
signature — the deterministic path, the LLM's own tool calls, the legacy signing tool —
leaves through one overridden method, so authorization is enforced once rather than
promised in several places. 22 offline tests cover the cases that matter: a peer's claim
of authorization grants nothing, the LLM fallback cannot sign for an unauthorized agent,
a resolver error declines. No network, no LLM, no server — `pytest test_agent_logic.py`.

In [AI_employee](https://github.com/Eman-Yousaf/AI_employee) the same idea is structural
rather than syntactic: the component that can actually send reads from exactly one
directory, and the component the model drives writes to a different one. Moving a file
between them is a human action. It is a separation of paths, not a sandbox, and the
README says so — an agent with vault write access could defeat it, which is worth knowing
before extending it to actions where being wrong is expensive.

### Declining is free; being wrong is not

From round 2 of that benchmark the authorization list stops naming agents and starts
*describing* them — paraphrasing something they said earlier, deliberately without
reusing the words. Identity has to be resolved at runtime against message history. That
resolver runs at temperature 0 and returns strict JSON, and any ambiguity, timeout, or
malformed response declines. A missed signature costs nothing; a wrong one costs a point.

[datahub-incident-copilot](https://github.com/Eman-Yousaf/datahub-incident-copilot) makes
the same trade against a live data catalog. It takes a one-line incident report, walks
DataHub's real lineage graph to find root cause, and computes downstream blast radius
before touching anything. Whether it may write back is not the model's call: confidence
and severity are computed in plain Python from which evidence it actually confirmed via
tool calls, and low confidence blocks the mutation and routes to human review instead.
Every successful write is re-read from the catalog afterward, so the trace shows the tag
landed rather than asking you to trust a bare `success: true`.

### A README that overstates is a bug

[GAMING-ARENA-WITH-AI](https://github.com/Eman-Yousaf/GAMING-ARENA-WITH-AI) is a C++17
tournament server — minimax tic-tac-toe engine behind token auth and role-based access,
matchmaking, REST API on Crow/ASIO. Its `src/` tree and GoogleTest suite target a modular
refactor whose headers were never written, so both are excluded from the build, and
asking for the tests stops with an explanation rather than a missing-header error from
inside a dependency. CI builds the configuration the README documents, on every push, so
the instructions are the ones known to work.

[Filer_ai](https://github.com/Eman-Yousaf/Filer_ai) is smaller and makes one decision
worth defending: the folder layout *is* the state. Items carry `status` in frontmatter
instead of living in a queue somewhere else, so there are never two sources of truth to
drift apart, and behaviour is configured in prose — the rules live in a handbook file,
so changing how it works means editing English.

### Knowing which minutes to spend is the part behind the paywall

Free SAT content is already solved — Khan Academy gives away the lessons, College Board
gives away the tests. What a $200/hour tutor actually sells is the judgement about which
forty minutes to spend tonight. [aria-sat-coach](https://github.com/Eman-Yousaf/aria-sat-coach)
prices that judgement: Bayesian knowledge tracing over 29 skills feeds a Monte Carlo
simulation of the exam the student has not sat yet, every skill is costed in points per
minute of study, and the session spends the minutes they actually have on the skills with
the highest return. It will send a student to the skill that is *cheaper to move* rather
than the one they are worst at — those are different answers, and only one of them is
worth score. It runs on WhatsApp, on a shared phone, over 2G.

The same machinery — hold a conversation over many turns, keep state about a real person,
decide what to do next, then act — is what runs the booking and screening agents at
[zayraa.tech](https://zayraa.tech).

---

Also here: [house-price-prediction](https://github.com/Eman-Yousaf/house-price-prediction),
a linear-regression pipeline documented end to end from imputation to held-out evaluation,
and coursework in C++, MySQL and networking.

[LinkedIn](https://www.linkedin.com/in/eman-yousaf96)
