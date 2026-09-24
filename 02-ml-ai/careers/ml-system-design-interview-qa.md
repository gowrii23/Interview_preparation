# ML system design — interview Q&A

Use the telecom assistant as the main design and the triage agent when they ask what you have already shaped. Numbers are budgets and policies, not fake production results.

## 1. What do you clarify before designing the telecom assistant?

**Intent.** Requirements before boxes.

**Strong answer.** Channels in v1, which languages I will actually evaluate, whether the user is authenticated, which actions move money, data-residency limits, and the latency the user will tolerate. I would cut v1 to chat, one language plus English, recharge and bill explanation, read-only account tools, and human handoff for anything that changes a plan or issues credit. Voice is a later path with ASR in front of the same policy.

**Follow-up.** “Why not all languages?” Because an average over languages I cannot label is a demo. I add a language when I have policy text and a small eval set.

**Weak answer.** “We will support every Indian language and voice on day one with one agent.”

## 2. Walk the request path.

**Intent.** End to end.

**Strong answer.** Authenticate. Detect language and code-mix. Classify intent. Retrieve policy with metadata filters and hybrid search. If the intent needs this account, call billing with the session’s account id, not an id from the text. Draft in the user’s language with policy ids and tool fields. Schema and safety checks. Send, or escalate with a summary. Unauthenticated users get public policy only.

**Follow-up.** “What is in the summary to the human?” Language, intent, tool facts, policy ids, and why you escalated. Not a hidden chain-of-thought.

**Weak answer.** “The user message goes to the LLM, which calls tools if it wants.”

## 3. What is retrieved versus called as a tool?

**Intent.** Source of truth.

**Strong answer.** Grievance steps and plan rules that must be quoted come from versioned policy chunks. The bill, the recharge result, and eligibility for this number come from APIs. The model may choose among `get_recharge`, `get_bill`, and `get_plan`. It may not supply the customer id. A conflict between a stale PDF and the API escalates; the API wins on the amount.

**Follow-up.** “Why can the model choose the tool at all?” Intent phrasing varies. The allow list is small. Argument injection cannot widen it to a refund.

**Weak answer.** “Embed the data warehouse nightly and retrieve balances.”

## 4. How do you keep the assistant from moving money?

**Intent.** Safety.

**Strong answer.** Refunds, SIM swaps, and plan changes are not tools in v1. The host injects identity. Observations and CMS pages are untrusted data. A prompt that says “waive the charge” does not create a tool. Low confidence, tool failure, and any money request escalate. PII is minimized and stays in the approved region.

**Follow-up.** “A user insists.” The assistant repeats what the read tools returned and hands off. It does not negotiate a credit it cannot post.

**Weak answer.** “The system prompt forbids refunds, but the refund API is available just in case.”

## 5. How do you evaluate this assistant?

**Intent.** Metrics that match harm.

**Strong answer.** Offline: intent accuracy, context recall, faithfulness, tool-argument accuracy, correct escalation, sliced by language and code-mix. Online: audited containment, escalation rate, and agent override rate. I watch a high-severity slice: wrong amount due. I do not optimize containment alone. A weekly human sample checks the judge model on the weaker language.

**Follow-up.** “What is the launch bar?” I would set it with the support lead on wrong-amount rate and escalation of money requests. I will not invent a target percentage in the interview.

**Weak answer.** “CSAT and a demo conversation.”

## 6. Where does latency come from, and what do you cut?

**Intent.** Budget.

**Strong answer.** Auth, retrieval, billing, and the model, plus ASR only on voice. Run retrieval and billing in parallel once intent is known. Stream a short preamble if a person is waiting, but do not stream an unvalidated amount. Timeout billing inside the user budget and escalate without guessing. Cache versioned public plan text. Do not cache bills. Skip the reranker when the error code is an exact match.

**Follow-up.** “The model is still late.” Shorten the context to the reranked policy plus the tool JSON. Do not add a second agent hop.

**Weak answer.** “Use a bigger context window so we only call the model once with all documents.”

## 7. How do you harden the defect-triage agent you already have?

**Intent.** Production judgment on your system.

**Strong answer.** Spring verifies the Jira webhook and dedupes. Python returns a versioned object: class, evidence, missing sources, draft, confidence. Read tools are scoped to the service and the window. The write credential runs only after approval. Caps and duplicate-call detection force insufficient evidence instead of a loop. A timed-out model stores the pack and does not invent an RCA. Traces carry template version and tool status. A golden set of code-versus-rule tickets is replayed when the prompt changes.

**Follow-up.** “What is the first alert?” Schema-failure rate and repeated identical tool calls. Both mean the loop is unhealthy before users finish complaining.

**Weak answer.** “Put a retry around the whole agent.”

## 8. Would you use several agents in the telecom design?

**Intent.** Restraint.

**Strong answer.** Not in v1. I want one draft with a read allow list and a separate checker only if the draft’s faithfulness fails often. Speech-to-text is a service, not an agent. Billing is a tool. A manager agent that delegates to a “billing agent” and a “policy agent” adds latency and splits the trace. I would add a second model when its permissions or its eval differ, and not before.

**Follow-up.** “Where would a second model earn its place?” An offline judge on a language slice, or a specialist that only classifies intent and never sees write tools.

**Weak answer.** “A planner, a translator, a retriever agent, and a tone agent.”

## 9. How do you handle a code-mixed voice complaint?

**Intent.** Indic constraints inside the design.

**Strong answer.** ASR in region, with a confidence threshold. Below it, go to a person with the audio handle, not a guessed transcript. Above it, mark code-mix, run the same intent and tools, and retrieve policy the user can be quoted. Reply in the language the product owner chose, consistently. Score the path by whether the recharge tool was the right call, not only by word error. If I have not checked Sarvam’s current speech API names, I describe the box and not a product id.

**Follow-up.** “Romanized chat, no speech?” Same flag. Do not strip English tokens. Test retrieval on that surface before launch.

**Weak answer.** “Translate the audio transcript to English in the prompt and ignore script.”

## 10. What do you explicitly leave out of v1, and why?

**Intent.** Scope control.

**Strong answer.** Plan changes, refunds, unauthenticated account lookups, all-language coverage, and an open agent with every internal API. I also leave out finetuning on raw chats because they contain PII and old plans. What remains is enough to learn: authenticated reads, quoted policy, per-language evals, and a clean handoff. The triage system teaches the same lesson on a different domain: drafts with evidence, writes behind a person.

**Follow-up.** “A stakeholder wants refunds in v1.” Then the design changes: a separate approved action, idempotency, a human confirm, and an eval for unauthorized credit. I do not bolt a refund tool onto the chat prompt.

**Weak answer.** “V1 is the full vision, and we will add guardrails later.”
