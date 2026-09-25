# Production LLM systems — interview Q&A

Answer as a lead who already runs webhooks and would put the model behind a contract. Do not invent latencies or bills.

## 1. How do you set a latency budget?

**Intent.** You design the wait before the prompt.

**Strong answer.** I split the user wait into webhook and auth, parallel read tools, one model call, and validation. Independent evidence calls run together. A second agent loop is only if the first pack is insufficient. The order-metrics demo should render a tool result, not sit behind a long loop. I set the budget from the product: an on-call draft can take longer than a first token in chat. I will not quote a millisecond number I did not measure.

**Follow-up.** “What dominates?” Usually the slowest tool or the model, once parses are parallel. Measure spans before you optimize the prompt.

**Weak answer.** “LLMs are slow, so we accept it.”

## 2. What do you stream, and what do you buffer?

**Intent.** UX versus contracts.

**Strong answer.** Stream tokens to a person watching a draft. Buffer machine output until the JSON schema validates, then persist. If the client disconnects, cancel the model call. A preview channel can stream while the system of record stores only the validated object.

**Follow-up.** “What about a half-written RCA posted to Jira?” That is a bug. The write happens after validation and after human approval, on the complete object.

**Weak answer.** “We stream JSON straight into the database.”

## 3. What are your timeout and retry rules?

**Intent.** Blast radius.

**Strong answer.** Every call has a timeout shorter than its caller. Retry idempotent reads a few times with jitter on transport errors and overload, not on bad arguments. Do not retry a Jira write unless an idempotency key is honored. If the model times out, store the evidence pack and let a person draft. One bad OpenSearch query must not pin the worker pool; circuit-break it.

**Follow-up.** “The ticket webhook fires twice?” Spring dedupes on issue id plus update time so you do not double-draft or double-post.

**Weak answer.** “Retry everything three times.”

## 4. How do you control cost without a fake bill?

**Intent.** Levers.

**Strong answer.** Cap tool bytes into the prompt, cap turns on the template, use a smaller model for routing only if quality holds, cache embeddings and unchanged retrieval, and run judge models offline. Log tokens per template version so a prompt edit that bloats input is visible. I do not quote a monthly spend I have not looked up.

**Follow-up.** “Cache the RCA?” Not on ticket text alone. Logs change. Cache a closed time window only if that index slice is immutable.

**Weak answer.** “Always use the largest model.”

## 5. What is a guardrail versus a prompt sentence?

**Intent.** Enforcement.

**Strong answer.** A prompt sentence is advice. A guardrail is code: input size and redaction, tool allow list, argument schema, output schema, citation ids subset of tool results, write blocked until approval. A toxicity classifier does not catch a wrong root cause. Failed checks escalate; they do not get retried into a looser prompt.

**Follow-up.** “Where does it run?” In the host, so a model bug cannot skip it. The Python service can pre-validate; Spring still refuses a non-conforming write.

**Weak answer.** “We say ‘do not hallucinate’ in the system prompt.”

## 6. What do you put on a trace?

**Intent.** Operability.

**Strong answer.** Template version, model id, each tool’s name, sanitized arguments, latency, status, bytes, token counts, schema result, and whether a human approved. Child spans match MCP calls, the same way I trace a Spring request. Alerts: duplicate tool calls, tool errors, schema failures, timeouts. I redact PII before the trace store.

**Follow-up.** “Can you replay?” Yes, from stored observations into a new prompt, without hitting production tools again.

**Weak answer.** “We log the whole prompt and the user’s phone number for debugging.”

## 7. How do you handle PII?

**Intent.** Data minimization.

**Strong answer.** Tickets and logs can hold account ids and phone numbers. The model receives the minimum needed to classify. Correlation ids are better than raw lines when the line is not the evidence. Provider retention has to be acceptable for the data class; a sovereign deployment may forbid a default public API. Eval spreadsheets get hand-stripped samples. Traces and cache keys do not contain raw PII.

**Follow-up.** “Who enforces that?” The Spring edge, before the Python call. The model service should not be the first place a secret is noticed.

**Weak answer.** “The model provider is trusted, so we send the ticket as-is.”

## 8. Why split Java and Python?

**Intent.** Your actual architecture.

**Strong answer.** Spring owns webhook verification, auth, idempotency, audit, PII policy, approval state, and the Jira write. Python owns the agent loop, model SDK, and schema-shaped draft. The contract is versioned JSON, `triage.v1`, with deadlines and typed errors: tool timeout, schema invalid, model unavailable. Python does not hold the write credential. Spring times out Python and can store evidence without a draft.

**Follow-up.** “Why not call the model from Spring only?” You can, for a single completion. An agent loop with tokenizers, eval tooling, and Python SDKs stays easier to test on that side. The boundary stays thin either way.

**Weak answer.** “Python is for AI, so the whole product moves to notebooks.”

## 9. How do you version prompts?

**Intent.** Change management.

**Strong answer.** A template version includes the prompt, the allow list, and the model id it was tested against. Traces store that tuple. Changes go through review with a fixture, including a poisoned tool result. I keep the previous version runnable so I can shift traffic back. I do not edit a prompt on a live server I cannot rebuild. Adding a required JSON field is a breaking change.

**Follow-up.** “How do you decide the new version won?” Replay the labeled tickets. Look at class agreement and schema failures, not at one impressive draft.

**Weak answer.** “We tweak the prompt in the console until the demo works.”

## 10. Design the fallback when OpenSearch is down.

**Intent.** Degraded mode.

**Strong answer.** The log tool returns unavailable. Commits and the deployment package may still return. The draft is blocked from claiming logs were checked. `missing_sources` includes logs. If policy says logs are mandatory to classify, the outcome is escalate and the evidence that did return is attached. No second model is asked to imagine the stack trace. The circuit breaker stops a retry storm.

**Follow-up.** “What does the engineer see?” A Jira note that the automatic draft did not run, the sources that failed, and the partial pack. Not a confident RCA.

**Weak answer.** “Fall back to the model’s memory of similar outages.”
