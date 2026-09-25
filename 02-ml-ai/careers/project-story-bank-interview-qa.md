# Project story bank — interview Q&A

Tell only the triage agent, Agentic Universe and its order-metrics demo, and lead themes you actually lived: mentoring, incidents, code quality. **[add your number]** means look up a real figure or do not say one.

## 1. Tell me about a system you shipped that used an LLM.

**Intent.** A real project, not a tutorial.

**Strong answer.** I built a defect-triage path. A Jira webhook starts it. The model does not classify from the summary alone. Over MCP it can read OpenSearch logs, recent commits, and deployment packages, then draft an RCA that says code defect or business-rule change. The write back to Jira stays behind a person. I can describe the controls precisely. Time saved or acceptance is **[add your number]** only if I measured it.

**Follow-up.** “What did you personally own?” Answer with the boundary you coded or led: webhook worker, tool allow list, prompt contract, or review bar. Do not claim the whole company platform if you owned a slice. Name the slice.

**Weak answer.** “We plugged GPT into Jira and it writes RCAs.”

## 2. What tradeoff did you make in that design?

**Intent.** Judgment under constraints.

**Strong answer.** I kept the webhook, the output schema, and the write gate as a fixed chain, and I allowed a small agent only while choosing read tools. If the service and window are known, I would rather fetch in parallel and call the model once. A free loop can find a missing clue and can also spin. The cap and the insufficient-evidence outcome are the tradeoff I would defend. I did not run a formal A/B unless I can cite **[add your number]**.

**Follow-up.** “What did you give up?” Some tickets that needed a sixth clever query will escalate. I prefer that to a confident draft with no sources.

**Weak answer.** “There was no tradeoff. The agent handles it.”

## 3. Tell me about Agentic Universe.

**Intent.** Platform, not a second chatbot.

**Strong answer.** It is a template-driven host for attaching MCP tools to an LLM workflow. The template sets the prompt, the servers, and the allow list. Telling the model about every tool in the building was the failure mode it avoids. The concrete demo was an executive view of on-demand order metrics, with metric tools only. How many workflows landed is **[add your number]** or “we proved it with that demo,” if that is the truth.

**Follow-up.** “What is hard about templates?” People copy a permissive template. Budgets and allow lists have to be required fields, not comments.

**Weak answer.** “It is a UI where you drag agents.”

## 4. Why was the dashboard a good demo?

**Intent.** Scope.

**Strong answer.** It was a different permission set from triage: read metrics, do not touch Jira or logs. It forced the template to be the security boundary. It also showed a case where a tool result should be rendered directly instead of paraphrased by a model that might remember a stale total. I would quote latency or executive adoption only as **[add your number]**.

**Follow-up.** “Did an executive use it?” Only say yes if one did. Otherwise: “It was a demo of the platform path. I will not invent adoption.”

**Weak answer.** “Executives loved it and it saved hours every day.”

## 5. Tell me about mentoring.

**Intent.** Lead evidence. This collapses if you invent a person.

**Strong answer.** As tech lead I mentored on real work, not a separate program. The example I will use is **[the real task they owned]**. I had them drive the design, I reviewed failure handling, and they shipped it. I will not invent a productivity score. If I tracked review turnaround or their later scope, that figure is **[add your number]**.

**Follow-up.** “What did you change in your reviews?” One habit: require the timeout, the idempotency story, or the bad tool-output fixture, whichever you actually required.

**Weak answer.** “I mentor everyone and my team is like a family.”

## 6. Tell me about an incident.

**Intent.** Ownership. A fictional outage will fall apart.

**Strong answer.** If you owned one: what broke, your role, what you mitigated, what you refused to do, and the durable fix. Impact and duration are **[add your number]** or “I will not guess.” If the only true story is a bad RCA draft: you stopped the write, kept traces, and made missing sources explicit. If you have not owned an incident, say so and describe the on-call failure you designed the agent to avoid.

**Follow-up.** “What would you page yourself for?” A write that fired without approval, or a tool loop on production logs. Those are the pages that match this system.

**Weak answer.** A dramatic outage with a precise customer count you never had.

## 7. How do you enforce code quality on agent code?

**Intent.** The work is software.

**Strong answer.** Prompt and allow-list changes go through review. Fixtures include a tool result that tries to override the instruction, and the test is that the write does not fire. The trace must carry the template version. Webhook handlers stay idempotent. I pick the two of these I actually enforced. Escaped-bug counts are **[add your number]** if I counted them.

**Follow-up.** “Show me the test.” Describe the poisoned log fixture in one minute. That is more credible than a quality slogan.

**Weak answer.** “We trust senior people to prompt carefully.”

## 8. What would you do differently?

**Intent.** Reflection without fake failure theater.

**Strong answer.** I would put the eval set in place earlier: labeled tickets for code versus rule, replayed on each template change. I would also split read and write credentials from the first PR, not after the first convenient demo. I will not invent a postmortem date. If something specific did bite you, tell that and skip the moral.

**Follow-up.** “Did the missing eval set cause a bad post?” Only if it did. Otherwise: “I am naming the gap in the method, not a fictional incident.”

**Weak answer.** “I would use a bigger model and more agents.”

## 9. Why this kind of role, and why not over-claim Sarvam?

**Intent.** Motivation tied to evidence.

**Strong answer.** The work I can show is production tool use: triage with logs, commits, and deploys, plus a template host. I want to apply that to multilingual products, which means data, evals, and residency constraints, not only English prompts. I am completing an AIML M.Tech at BITS Pilani WILP, expected 2027. I will talk about Sarvam from their public pages the morning of the interview. I will not quote funding or a model card I have not opened.

**Follow-up.** “What do you want to learn in the first months?” Per-language evaluation, how their speech and text stacks are meant to be called, and the deployment constraints on customer data. Not “I will train the next foundation model” unless the posting says that.

**Weak answer.** “Your models are the best in the world and I have always wanted to work on them.” Unsourced.

## 10. A stakeholder wants the agent to close Jira tickets automatically. What do you say?

**Intent.** Pushback with a path.

**Strong answer.** I would not give the triage template a close permission. The draft can recommend a class and attach evidence; a person closes. If they need speed, we measure how often reviewers accept the draft unchanged, which is **[add your number]** if it exists, and we tighten the evidence rules. Auto-close is a later decision with an error budget they must accept in writing. Until then the fallback is the queue, not a silent write.

**Follow-up.** “They insist on a pilot.” A pilot on internal tickets, with the close still confirmed by the assignee, and a weekly read of disagreements. Not a silent production close.

**Weak answer.** “Sure, we can let the model close them if confidence is above 0.9.”
