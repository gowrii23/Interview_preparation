# LLM internals — interview Q&A

## 1. What does BPE change about how you count context and cost?

**Intent.** Tokens are not words.

**Strong answer.** Byte Pair Encoding merges frequent character or byte sequences into a fixed vocabulary. Common words may be one token; rare strings, code, and some scripts become many. The context window, the KV cache, and the price per call are all in tokens. A 4k window is not 4,000 words. Leading spaces and newlines can change the id, so a prompt that looks equivalent can be a different sequence. I would inspect the tokenizer when a language or a JSON schema behaves strangely.

**Follow-up.** Why might a model be weak at character-level spelling?

**Weak answer.** "Tokenization splits on spaces, so every word is a token."

## 2. State the pretraining objective. What does it not optimize?

**Intent.** Next-token loss versus product goals.

**Strong answer.** For a decoder-only model the loss is the sum of \(-\log p(x_{t+1} \mid x_{\le t})\) over positions. The model learns to continue text from the training distribution. It does not directly optimize factual correctness, instruction following, refusal, or calibration of answers. Those require later data: instruction demonstrations, preferences, tools, or retrieval. A low pretraining loss means better compression of that corpus, not a guarantee the model should be shipped as an assistant.

**Follow-up.** Why mask the prompt tokens' loss during instruction finetuning?

**Weak answer.** "Pretraining teaches the model to understand the user's question."

## 3. How is instruction tuning different from pretraining?

**Intent.** Same loss, different data and masking.

**Strong answer.** The loss is still next-token cross-entropy. The data becomes prompt–response pairs in a chat template, and the loss is usually applied only to the response. Pretraining builds general continuation. Instruction tuning shifts probability toward the format, tone, and task distribution of the demonstrations. It does not insert facts that were never in the weights or the prompt. A narrow SFT set can overwrite helpful general behavior, so I keep a regression set from the previous skill.

**Follow-up.** If the base model already follows instructions, when would you still finetune?

**Weak answer.** "Instruction tuning is a different architecture with a reward layer on top."

## 4. What is the KV cache, and how do prefill and decode differ?

**Intent.** Serving cost model.

**Strong answer.** Causal keys and values for past tokens do not change when a new token is generated, so they are cached. Prefill runs the whole prompt in parallel and writes the cache. It is compute-heavy and grows with prompt length, with quadratic attention in that length. Decode then emits one token at a time, reads the weights and the cache, and appends one key and value per layer. Decode is often bandwidth-bound. Short answers on long prompts stress prefill. Long answers stress decode and cache memory.

**Follow-up.** Why does grouped-query attention shrink the cache?

**Weak answer.** "The KV cache stores the prompts so we do not have to send them again from the client."

## 5. Sketch KV-cache memory for one sequence.

**Intent.** A formula the candidate can apply.

**Strong answer.** For each layer you store K and V. Bytes are about \(2 \times n_{\text{layers}} \times n_{\text{kv heads}} \times d_{\text{head}} \times \text{sequence length} \times \text{bytes per element}\). In bf16, bytes per element is 2. A batch of independent sequences multiplies this, which is why long contexts and large batches run out of memory even when the weights fit. Quantizing the cache reduces the bytes per element and can hurt quality. I would measure the real allocator number, but this is the right lower-order sketch.

**Follow-up.** What happens to this cost when the user asks for 1,000 output tokens?

**Weak answer.** "Cache size is the number of parameters times 4."

## 6. How do temperature and top-p change the next token?

**Intent.** Decoding is not training.

**Strong answer.** Temperature divides logits before softmax. Below 1 the distribution sharpens and the sample stays near the mode. Above 1 it flattens and rare tokens become more likely. Top-p keeps the smallest set of tokens whose probabilities sum to at least \(p\), then renormalizes, so the shortlist is tight when the model is confident and wider when it is not. Neither operation adds knowledge. For JSON and tool arguments I want greedy or a very low temperature. For varied prose I might raise temperature and set top-p. I compare prompts only at a fixed decoding setup.

**Follow-up.** What failure do you expect at temperature 1.5 on a factual extraction task?

**Weak answer.** "Higher temperature makes the model more creative and therefore smarter."

## 7. Give three distinct causes of hallucination.

**Intent.** Separate data, decoding, and product causes.

**Strong answer.** First, the next-token objective prefers a plausible continuation, including a continuation that was never checked. Second, the fact was rare, wrong, or newer than the training data. Third, the prompt forces an answer and never allows abstention, or retrieval returned the wrong passage and the model treated it as given. Hot decoding is a fourth, sampling a low-probability fabrication. I would measure unsupported claims on a labeled set and tag whether the context itself was wrong.

**Follow-up.** How is "ignored the retrieved passage" different from "the passage was wrong"?

**Weak answer.** "Hallucination means the temperature was above zero."

## 8. What is actually inside the context window at inference?

**Intent.** No hidden memory.

**Strong answer.** Only the tokens you send this call: system prompt, history you included, retrieved text, and the answer so far, up to the window. There is no side channel of long-term memory. If earlier turns were truncated, they do not affect the next token. Long windows still attend unevenly; relevant text buried in the middle is used less reliably than text at the ends. I would rather retrieve a short relevant passage than append every document that might help.

**Follow-up.** A summary of older turns is written back into the prompt. What can that summary destroy?

**Weak answer.** "The context window is how long the model remembers the user across sessions."

## 9. Why can a batch of decode requests be cheaper per token than one request?

**Intent.** Weight traffic versus batching.

**Strong answer.** Each decode step must read the weight matrices. One request pays that bandwidth for a single token. A batch reads the weights once and applies them to many sequences, so arithmetic intensity goes up and cost per token falls, until the KV cache or activation memory fills the GPU. Prefill can already be compute-bound on a long prompt. This is why serving stacks use continuous batching. It is also why a latency SLO and a throughput goal pull batch size in opposite directions.

**Follow-up.** What new memory term grows when you raise the batch?

**Weak answer.** "Batching is only a training concept. Inference runs one user at a time."

## 10. A benchmark score jumped after continued pretraining. What do you suspect first?

**Intent.** Contamination versus real gain.

**Strong answer.** I suspect the benchmark text, or a near-duplicate, entered the training data. I would check overlap between eval items and the corpus, look for memorized completions with a greedy prefix, and keep a private holdout the crawl could not have included. A real gain should show up on that private set and on a task metric that matches production, not only on a public leaderboard. I would not quote the public number as a ship decision until contamination is addressed.

**Follow-up.** Exact string match finds no overlap. Why might you still be contaminated?

**Weak answer.** "The score went up, so the model learned the skill. Public benchmarks are the test set."
