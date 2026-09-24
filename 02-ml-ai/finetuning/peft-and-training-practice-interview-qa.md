# PEFT and training practice — interview Q&A

Questions 3, 6, and 7 ask for a count, a freeze check, or a memory-relevant training choice.

## 1. What do you put in an instruction example, and what do you exclude from the loss?

**Intent.** Template and loss mask.

**Strong answer.** Each example is the chat template the base model expects: a stable task prefix, the user input as production will send it, and a target completion a grader can check. I use the same template at train and serve time. The loss is next-token cross-entropy on the assistant tokens only, so the model does not spend the update copying the prompt. I do not train on unlabeled scraped text mixed into the same run without a reason. I keep the base tokenizer unless I am willing to train new embedding rows.

**Follow-up.** The served system prompt is longer than the one in the training pairs. Why can metrics fall?

**Weak answer.** "I dump a CSV of user questions into the model and let it learn the answers unsupervised."

## 2. How do you split a small support-ticket finetune?

**Intent.** Leakage and a true holdout.

**Strong answer.** I split by ticket or customer, not by random sentences from the same ticket. I deduplicate near copies. I cut a validation slice for learning rate, rank, and early stopping, and a final eval slice I do not use to pick the checkpoint. Both slices include edge cases and a few examples of the old behavior I must keep. If I only have a few hundred labels, I say so, keep the eval fixed, and prefer cleaner labels over scraping more noisy ones.

**Follow-up.** Why is a random split of sentences from the same document optimistic?

**Weak answer.** "I use the test set to pick the best epoch so the number I report is the best one."

## 3. You adapt Q, K, V, and O in 24 layers. Each matrix is \(2048 \times 2048\), rank 16. How many trainable parameters, and what is frozen?

**Intent.** Count and freeze. 24 * 4 * 16 * (2048+2048) = 96 * 16 * 4096. 16*4096=65536. 96*65536.

96*65536 = 96 * 2^16 = (100-4)*65536 = 6,553,600 - 262,144 = 6,291,456.

**Strong answer.** Four matrices per layer times 24 layers is 96 matrices. Each contributes \(16 \times (2048+2048) = 65{,}536\) parameters. Total trainable is \(96 \times 65{,}536 = 6{,}291{,}456\). Frozen: the base \(W_0\) for those projections, the MLP, the embeddings, and layer norms, assuming I did not target them. Optimizer state exists only for the 6.3 million adapter parameters. If the framework reports a trainable count far from this, the target module list is wrong.

**Follow-up.** You switch to QLoRA. Does this trainable count change? What storage does change?

**Weak answer.** "Rank 16 and 24 layers means \(24 \times 16 = 384\) parameters."

## 4. Give a starting recipe and the two curves that would make you leave it.

**Intent.** Checklist, not a universal learning rate.

**Strong answer.** Same base and template, target query and value or all attention projections, rank 8 or 16, alpha equal to rank or twice rank, LoRA learning rate around \(1 \times 10^{-4}\) as a start, a small number of epochs, gradient accumulation to the effective batch that fits, packing only after the boundary mask is tested. I log the trainable count. I leave this recipe if validation loss rises while training loss falls, which means fewer epochs or a lower rate, or if both losses floor and a higher rank on validation still helps, which means the adapter is too small. I never pick the recipe on the final test slice.

**Follow-up.** Why might \(1 \times 10^{-4}\) be too high for a full finetune of the same model?

**Weak answer.** "Use learning rate 1e-4, rank 8, and three epochs for every model. That is the correct setting."

## 5. What is sequence packing, and how can it silently corrupt training?

**Intent.** Utilization versus boundary bugs.

**Strong answer.** Packing concatenates short examples up to the max length so a step does less padding. It is valid only if attention and the loss cannot treat the next example as a continuation of the previous one. A broken boundary lets the model learn answers from the neighbor's prompt, and the loss looks great. If I have not tested the mask, I pad. I also watch the length histogram so one long document does not dominate memory.

**Follow-up.** You pack two JSON tasks and the model starts emitting the previous example's keys. What do you inspect?

**Weak answer.** "Packing compresses the weights so the sequence fits in 4-bit."

## 6. Full finetune Adam state versus LoRA: where does the memory go for a model with \(P\) parameters if you only adapt 20 million parameters?

**Intent.** Bytes on the base versus bytes on the adapter.

**Strong answer.** Full finetune in a typical bf16-weight, fp32-Adam setup stores about 2 bytes per parameter of weights, about 2 of gradients, and about 8 of Adam moments, so on the order of \(12P\) bytes before activations and any fp32 master copy. LoRA stores the frozen base once, about \(2P\) bytes in bf16, and pays gradients plus Adam only on 20 million parameters. At 12 bytes per trainable parameter that adapter overhead is about 240 MB, not terabytes. QLoRA replaces the \(2P\) base with about \(0.5P\) plus scales. Activations can still exceed all of these if the sequence is long. I would quote this as a sketch and then read the allocator.

**Follow-up.** Why can gradient accumulation lower peak memory when the effective batch stays constant?

**Weak answer.** "LoRA and full finetune use the same memory because the activations are the model."

## 7. How do you detect catastrophic forgetting before you merge?

**Intent.** A second metric, and what is frozen does not end the story.

**Strong answer.** I score a fixed slice of the previous task every epoch, with the same grader as before, next to the new validation metric. The base weights are frozen, but the served function is \(W_0\) plus the adapter, so old behavior can still move. A drop means shorter training, a lower learning rate or rank, or rehearsal data from the old task mixed into training. I make that comparison before merge, on the same template I will serve. A training loss that only measures the new answer cannot show forgetting.

**Follow-up.** The new metric is up and the old metric is down 15 points. Do you merge?

**Weak answer.** "Forgetting is impossible with PEFT, so I only log the new loss."

## 8. What is the merge step, and what do you hand to vLLM or TGI?

**Intent.** Serving artifact.

**Strong answer.** Merge builds \(W' = W_0 + (\alpha/r) BA\) so inference is a plain dense model with base latency. For a 4-bit QLoRA base I dequantize, then add, then save bf16. I hand vLLM or TGI that checkpoint, the tokenizer, and the chat template, and I set max context and dtype explicitly. Those servers handle batching and the paged KV cache. If I need several tasks on one base, I confirm the server can load adapters; otherwise I merge per task. Serving quantization is a separate choice, and I evaluate the quantized artifact if that is what production will call.

**Follow-up.** Why might an unmerged adapter call disagree with the merged model after QLoRA?

**Weak answer.** "vLLM finetunes the adapter online using the user traffic."

## 9. What must a run log for you to trust a metric a week later?

**Intent.** Reproducibility as an interview signal.

**Strong answer.** Data version and hash, split definition, row counts, template sample, base model id, target modules, trainable count, rank, alpha, dropout, learning rate, epochs, effective batch, sequence length, packing flag, seed, train and validation curves, forgetting-slice scores, decoding settings, and whether the metric is merged or unmerged. The final test number is recorded once with the checkpoint hash. If two of those are missing, I cannot tell a data bug from a learning-rate bug.

**Follow-up.** Which single log line tells you the module names matched nothing?

**Weak answer.** "I log the final accuracy. The rest is in my notebook history."

## 10. Data is noisy and the stakeholder asks for a larger rank and more epochs. What do you say?

**Intent.** Data quality dominates capacity.

**Strong answer.** A larger rank and more epochs will fit the noise more tightly. I would rather fix labels, drop duplicates, and add the missing edge cases, then use a modest rank and stop on validation. I can show the failure mode: train loss will fall and held-out quality will not, or will fall after a point. If the labels are the product spec and they are wrong, no optimizer corrects them. I will spend the next run's budget on a reviewed holdout and a forgetting slice.

**Follow-up.** How many bad pairs does it take to matter on a 500-example set?

**Weak answer.** "Noise averages out if I train long enough with a high rank."
