# Sarvam AI and Indic LLMs — interview Q&A

Public positioning only. If a fact is not on the page you reread that morning, do not say it. Do not quote funding, unreleased models, or benchmark tables from memory.

## 1. Why is Indian-language AI hard?

**Intent.** Depth beyond “many languages.”

**Strong answer.** Several scripts and complex Unicode, heavy code-mixing and romanization, rich morphology, thin data outside a few languages, dialects and register, and speech that is noisier than clean text. A model that is fine on formal Hindi news can fail a recharge complaint in Latin script. I would evaluate per language and per channel, not with one English-heavy score.

**Follow-up.** “Which of these have you shipped?” I have shipped tool-using backends, not a foundation model. The language issues are how I would design evals and data checks on a team that does. I am in the BITS Pilani WILP M.Tech AIML program, expected 2027, to go deeper.

**Weak answer.** “Indian languages are low resource.” Stop there.

## 2. What goes wrong with tokenization?

**Intent.** You understand fertility without reciting a paper.

**Strong answer.** An English-centric subword tokenizer spends many tokens on Indic text, so context, latency, and cost get worse for those users. It can split inside a written syllable and treat native script and romanized Hindi as unrelated. Exact ids still should not be generated if a tool can return them. Sarvam has publicly discussed a tokenizer aimed at Indic text for Sarvam-1. I would reread the vocabulary size and any fertility table instead of quoting them.

**Follow-up.** “How would you notice this in production?” Tokens per request sliced by language. A sudden gap versus English on the same task length is a tokenization or prompt-template smell.

**Weak answer.** “We use BPE so tokenization is solved.”

## 3. Why are English benchmarks not enough?

**Intent.** Evaluation taste.

**Strong answer.** They do not measure a Tamil or code-mixed support task, translation into a low-resource language, or whether ASR errors change the tool call. I want per-language sets, a task metric after speech, faithfulness to policy, and a human spot-check. A judge model that is weak in that language will hide errors. I will not quote Sarvam benchmark numbers I have not just read.

**Follow-up.** “What would you label first?” Fifty real code-mixed chats for one flow, with gold intent and gold tool calls, before a large general set.

**Weak answer.** “If it scores well on a popular English test, Indic quality follows.”

## 4. What does sovereign AI change in your design?

**Intent.** Product constraint, not a slogan.

**Strong answer.** Some buyers will require a known region, limited prompt retention, and no casual copy of citizen or customer text to an arbitrary API. I design minimization, audit, and an explicit run location into the path. I do not invent Sarvam’s infrastructure. I reread how they currently describe sovereign compute before I repeat a phrase from their homepage.

**Follow-up.** “How does your Spring boundary help?” Redaction and policy sit at the edge. The model service receives a minimized payload and a region assumption it must not silently override.

**Weak answer.** “Sovereign means the model is better because it is Indian.”

## 5. How should you introduce yourself for this role?

**Intent.** Fit, without cosplay as a pretraining researcher.

**Strong answer.** I am a senior tech lead who has built a Jira-triggered triage agent with MCP tools for logs, commits, and deploys, and a template runtime that attached tools to more than one workflow, including an order-metrics demo. I am doing the AIML M.Tech while leading. I want applied work: multilingual data, evals, agents, production boundaries. I have not trained their models and I will not imply I have.

**Follow-up.** “Why not a pure research role?” Unless the posting asks for that, my evidence is production systems and a degree in progress. I would rather say so than stretch.

**Weak answer.** “I am passionate about AI and use ChatGPT daily.”

## 6. How would you handle code-mixing in support text?

**Intent.** Practical NLP.

**Strong answer.** Detect it, do not strip Latin tokens, and do not assume a translation to formal Hindi is lossless. Index policy in the languages you serve, and test retrieval on romanized queries. If the embedding model has not seen that mix, say so and use a lexical field or a translation step you have measured. Reply in the user’s mix if that is the product choice, consistently.

**Follow-up.** “Translate everything to English first?” Only as a measured fallback. It can break entity names and tone, and it hides a weak Indic model behind an English one.

**Weak answer.** “Translate to English, solve, translate back. Always.”

## 7. Where does speech change the architecture?

**Intent.** You noticed Sarvam’s public speech emphasis without inventing products.

**Strong answer.** Public material emphasizes speech as well as text. I would confirm current products on the site. Architecturally, ASR sits in front of the same intent and tool path, in the approved region. Low confidence skips automation. I measure task success after ASR, not only word error. Telephony noise and code-mixed speech are the cases I would put in the eval set.

**Follow-up.** “Have you built ASR?” No. I can own the service boundary, latency budget, and escalation when recognition is weak.

**Weak answer.** Name a speech model and a WER you did not verify that morning.

## 8. What will you re-read on their website?

**Intent.** Discipline.

**Strong answer.** Homepage product categories. About page wording on the founders Vivek Raghavan and Pratyush Kumar, and the exact public line on AI4Bharat, which I will not turn into “Sarvam is AI4Bharat.” The Sarvam-1 post and any later model page they feature, skipping numbers I cannot point to. The docs index for APIs that are actually listed. The careers posting I am answering, verb by verb.

**Follow-up.** “They ask about a model you do not know.” I say I have not reviewed that card and ask which capability they care about. I do not invent a parameter count.

**Weak answer.** Recite a funding round or a benchmark from an old memory.

## 9. RAG, finetune, or tool for a low-resource language?

**Intent.** Method choice under data scarcity.

**Strong answer.** Live account facts are tools in any language. Policy text is retrieval if you have the documents in that language; if you only have English policy, say you are answering from an English source. Finetuning does not create missing parallel data and goes stale. I would spend scarce labels on a small eval and on error tags before I spend them on a fine-tune I cannot refresh.

**Follow-up.** “The model refuses a low-resource language.” Escalate to a person, or to a language it does handle, with the user’s consent. Do not answer fluently in the wrong language.

**Weak answer.** “Finetune on a scraped dump of that language over the weekend.”

## 10. What will you not claim in the interview?

**Intent.** Trust.

**Strong answer.** No funding figures, no unreleased models, no customer metrics, no benchmark I did not reread, no implication I built their stack. No phone, no personal identifiers beyond the professional story. For my own work, no time-saved or accuracy number unless I fill it from a real measurement. AI4Bharat stays a public association in founder bios, not an internal claim.

**Follow-up.** “Then what do you talk about for forty minutes?” The triage design, MCP boundaries, how I would evaluate a code-mixed support agent, and how Spring and Python split. That is enough material without fiction.

**Weak answer.** A confident tour of their private roadmap.
