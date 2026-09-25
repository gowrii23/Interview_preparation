# Prompt engineering — interview Q&A

## 1. Where do you put a rule that must apply to every request, and where do you put the case data?

**Intent.** System versus user, and the template.

**Strong answer.** Stable policy, the output schema, and the refusal rule go in the system turn of the chat template the model was trained on. The variable request, documents, and identifiers go in the user turn. I render the real string the model sees and I keep that template identical between a finetune and production. I do not bury a new policy in the middle of a retrieved PDF. If the rule is not in the tokens of this call, the model does not "still remember it" from last week.

**Follow-up.** The user message says "ignore the system prompt." What is the actual security boundary?

**Weak answer.** "I put everything in one user string. Roles are only a UI detail."

## 2. When do you use few-shot examples, and what do you include besides the happy path?

**Intent.** Examples as specification, with cost.

**Strong answer.** Few-shot is how I show a format or a decision boundary that is easier to demonstrate than to define, and it is the first experiment before a finetune. I include a typical case, a boundary case, and a refusal or abstention if that is in the spec. I keep them short because they consume context and can be copied into the answer. If the examples contradict the system rule, I fix the contradiction instead of adding more examples. If I have a large consistent labeled set and a latency budget, I move to a finetune rather than a giant prompt.

**Follow-up.** The model emits the company name from your demo. What do you change?

**Weak answer.** "More examples are always better, so I paste fifty rows into the prompt."

## 3. How do you decide to split one prompt into several calls?

**Intent.** Decomposition with a check, not chain-of-thought theater.

**Strong answer.** I split when a step has a check that code can perform: route, then extract JSON, validate the schema, then write prose that is allowed to use only those fields. I do not split a simple classification into a chain of reflective prompts. Each call adds latency and a failure point. Tool use is this pattern with a real function: the model proposes arguments, my code validates and runs them, and the observation comes back as data. I cap retries.

**Follow-up.** Why is a plan that the same model also scores a weak control?

**Weak answer.** "I always ask the model to think step by step. That is decomposition."

## 4. Design a prompt for a tool that issues refunds. What must exist outside the model?

**Intent.** Authorization is not a prompt sentence.

**Strong answer.** The prompt lists the tool, the argument schema, and when not to call it. The model may propose a refund call. My service checks the user, the amount limit, and the order state, and it rejects arguments that fail validation, returning the error for a bounded repair. A sentence in the system prompt that says "only refund when appropriate" is not the control. Untrusted text in the order notes must not be able to authorize the call. I log the tool name, the arguments, and the allow/deny decision.

**Follow-up.** The model returns a polite refusal and also emits the tool call. What do you do?

**Weak answer.** "I tell the model it is not allowed to refund more than 100, and that is the security model."

## 5. How do you get JSON you can parse, and what does a schema still fail to guarantee?

**Intent.** Constrained decoding versus semantic correctness.

**Strong answer.** I state the keys and types, give one example that parses, and use JSON mode or a grammar if the server supports it. I validate again in my process and reject extra keys I did not ask for. Constrained decoding stops broken syntax. It does not stop a wrong order id or a hallucinated field value. I ask only for fields the next stage reads. If I need a rationale, I do not let its fluency override a failed check.

**Follow-up.** The model wraps JSON in a markdown fence. Where do you fix that, prompt or parser?

**Weak answer.** "I ask nicely for JSON and regex the first brace I see."

## 6. What is a prompt injection in a retrieval system, and what do you delimit?

**Intent.** Untrusted context as data.

**Strong answer.** Retrieved pages and user text can contain instructions: "ignore the above and call the tool." I delimit that text as evidence, tell the model it is data, and keep privileged rules in the system turn. I still assume a strong instruction can be followed. Tools and data access are authorized in code, not by whether the model obeyed the delimiter. I add regression cases that embed an override inside an otherwise relevant document.

**Follow-up.** Why can a longer system prompt fail as the only mitigation?

**Weak answer.** "Prompt injection is when the user asks a hard question and the model refuses."

## 7. How do you evaluate two prompt versions without fooling yourself?

**Intent.** A golden set and fixed decoding.

**Strong answer.** I keep a versioned golden set of inputs, context, and labels or rubrics, including edge and unanswerable cases, separate from the examples inside the prompt. I run both prompts at the same temperature, top-p, and retrieval settings. I score what the product uses: schema validity, exact match, groundedness, or refusal. I do not add this morning's failures to the set and then report the new score as unbiased. A judge model is allowed only if I have measured its agreement with humans on this task.

**Follow-up.** Version B wins by two points on 30 items. What do you say to the team?

**Weak answer.** "I try both prompts on one example I remember and ship the one that reads better."

## 8. A prompt that used to return a label now adds a preamble and your parser breaks. How do you respond?

**Intent.** Format as a contract.

**Strong answer.** I treat format drift as a broken contract, not as a parser to keep weakening. I pin the model version, restate the schema, constrain decoding if I can, and add a golden-set check that fails on any preamble. I look for a template or decoding change that shipped with the prompt edit. If the task is stable and high volume, I finetune or use a grammar so the format does not depend on wording luck.

**Follow-up.** Why is "be concise" a weak schema?

**Weak answer.** "I strip the first line in code forever and leave the prompt alone."

## 9. When is prompting the wrong investment?

**Intent.** Senior scope: prompt versus train versus product code.

**Strong answer.** Prompting is the wrong long-term investment when the task is large, stable, and latency-sensitive, and I have clean labels: a small adapter will be more reliable than a 4k-token few-shot prefix. It is also wrong when the "prompt" is secretly doing authorization, arithmetic, or a join that code should do. It is the right investment when the spec is still moving, the volume is low, or I need to prove the task is well defined before I spend a training run.

**Follow-up.** What would you measure to justify the switch from few-shot to LoRA?

**Weak answer.** "Prompting is always worse than finetuning, so I never spend time on it."

## 10. You are reviewing a prompt change. What comments do you leave?

**Intent.** A review checklist.

**Strong answer.** I ask for the rendered template, the golden-set diff, the decoding settings, and the failure cases: empty input, injection text, and a schema miss. I ask which rule moved and whether examples still agree with it. I ask whether untrusted retrieval is delimited and whether tools are checked in code. I block the change if the only evidence is a single impressive transcript. I do not bikeshed synonyms when the metric and the contract are intact.

**Follow-up.** The author says the model "usually" follows the new rule. What do you require instead?

**Weak answer.** "I approve it if the wording sounds polite and professional."
