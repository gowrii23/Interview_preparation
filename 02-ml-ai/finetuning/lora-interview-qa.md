# LoRA — interview Q&A

At least three of these ask for a calculation or a freeze/rank argument: questions 2, 3, and 4. Question 8 is a fourth quantitative check.

## 1. What problem does LoRA solve, in one diagram you can say aloud?

**Intent.** Frozen base plus a low-rank update.

**Strong answer.** Full finetuning stores a new copy of every weight and a gradient and optimizer state for each. LoRA leaves \(W_0\) frozen and learns \(\Delta W = BA\) with rank \(r\) much smaller than the matrix dimensions. The forward pass is \(W_0 x + (\alpha/r) B A x\). \(B\) starts at zero so the update starts at zero. At serve time I can merge the product into \(W_0\) and run a normal dense layer.

**Follow-up.** Why initialize \(B\) at zero rather than both matrices at random?

**Weak answer.** "LoRA compresses the model so it fits on a laptop by deleting weights."

## 2. A linear map is \(4096 \times 4096\). Rank is 8. How many trainable parameters does LoRA add, and what fraction is that of the full matrix?

**Intent.** Compute \(r(d+k)\) correctly.

**Strong answer.** \(B\) is \(4096 \times 8\) and \(A\) is \(8 \times 4096\), so the count is \(8 \times (4096 + 4096) = 65{,}536\). The full matrix has \(4096^2 = 16{,}777{,}216\) parameters. The adapter is \(65{,}536 / 16{,}777{,}216 = 1/256\), about \(0.39\%\). If I adapt four such matrices in a layer, I multiply by four. I do not multiply by the whole model if embeddings and the MLP stay frozen. Biases are usually not included.

**Follow-up.** Repeat the count for \(r = 64\). What else must you change so the update magnitude stays similar?

**Weak answer.** "Rank 8 means 8 parameters per layer."

## 3. What is frozen during a standard LoRA run, and which tensors have Adam moments?

**Intent.** Memory follows the trainable set.

**Strong answer.** Frozen: the base weights \(W_0\), and typically the token embeddings, the output head unless targeted, and layer norms. Trainable: \(A\) and \(B\) on the chosen projections. Adam keeps first and second moments only for those adapter tensors, not for \(W_0\). That is why a 7B-scale base can be finetuned on one GPU when the full Adam state would not fit. If a training log shows trainable parameters equal to total parameters, the freeze failed.

**Follow-up.** You add the LM head as a target. What happens to the parameter count?

**Weak answer.** "Everything is frozen except the last layer, which is what LoRA means."

## 4. What does \(\alpha / r\) do, and what happens if you double \(r\) and leave \(\alpha\) and the learning rate fixed?

**Intent.** Scaling is part of the method.

**Strong answer.** The forward update is scaled by \(\alpha / r\). Alpha is a constant, not a second learning rate inside Adam. Holding \(\alpha\) fixed while raising \(r\) shrinks the multiplier, which is meant to keep \(\Delta W\) from growing just because more low-rank directions were added. If I double \(r\) from 8 to 16 and keep \(\alpha\) at 16, the factor drops from 2 to 1. Effective step size changes even though the learning rate string in the config did not. I would retune or set \(\alpha\) proportional to \(r\) and confirm on validation loss.

**Follow-up.** Your library applies \(\alpha/r\) only at merge time, not in training. How do you notice?

**Weak answer.** "Alpha is the learning rate of the LoRA layers."

## 5. Why can a low-rank update adapt a model that has huge weight matrices?

**Intent.** Intrinsic-rank intuition without fake theorems.

**Strong answer.** Pretraining already built a rich feature map. Many downstream tasks need a small rewrite of how those features combine, not a full-rank replacement of \(W_0\). The hypothesis behind LoRA is that the useful update has low intrinsic rank, so a product \(BA\) can approximate it. This is empirical, not a promise. A very new output format, a new language, or many unrelated skills can need a larger \(r\) or more target modules. If validation stalls and a higher rank improves it, the subspace was too small.

**Follow-up.** How do you tell "rank too small" from "the labels are wrong"?

**Weak answer.** "Neural nets are always low rank, so rank 1 is enough for any task."

## 6. Which matrices would you adapt first, and why not always every linear layer?

**Intent.** Capacity versus memory, without invented paper scores.

**Strong answer.** Attention projections are the usual first targets. The original LoRA work highlighted query and value as a strong efficiency point; key and output are reasonable additions. MLP projections add a lot of capacity because those matrices are large, and they add a lot of adapter parameters for the same reason. I start from query and value, or all attention projections, measure validation and a forgetting slice, then add MLP modules if the task underfits. I confirm the module names exist on this architecture by printing the trainable count. Copy-pasted names that match nothing train an empty adapter.

**Follow-up.** Name a reason to leave the embedding frozen.

**Weak answer.** "You must adapt every weight or LoRA does not converge."

## 7. How do you serve one adapter versus five tasks on one base?

**Intent.** Merge versus hot side paths.

**Strong answer.** For one task I merge \(W' = W_0 + (\alpha/r) BA\) and serve an ordinary checkpoint in vLLM or TGI at the same latency as the base. For several tasks I can keep \(W_0\) in memory and attach one adapter at a time, paying a small extra matmul and the operational cost of routing. I only add adapters together when they were trained on the same base and I have checked interference on both tasks. An adapter is not valid on a different checkpoint or a different tokenization.

**Follow-up.** After a merge, can you still recover the original base from the server weights alone?

**Weak answer.** "Merging quantizes the model to 4-bit for inference."

## 8. Your trainable count is 4.2 million on a model with 32 attention matrices shaped \(4096 \times 4096\), rank 16, adapting Q and V only. Does the count match?

**Intent.** Sanity-check a training log against \(r(d+k)\) times the number of adapted matrices.

**Strong answer.** Each matrix contributes \(r(d+k) = 16 \times (4096+4096) = 131{,}072\) parameters. Thirty-two matrices give \(32 \times 131{,}072 = 4{,}194{,}304\), which matches about 4.2 million. If the log had shown ~2.1 million, I would suspect only one of Q or V was targeted, or rank 8. If it had shown tens of millions more, I would suspect MLP modules or the LM head were included.

**Follow-up.** What count would you see if dropout were on? Does dropout add parameters?

**Weak answer.** "4.2 million is the size of the base model, so the run looks fully finetuned."

## 9. Describe catastrophic forgetting in a LoRA run and one control you would actually log.

**Intent.** Frozen weights are not a forgetting shield.

**Strong answer.** The adapter is trained only on the new pairs, so behavior that never appears in those pairs can move: tone, refusal, or a previous extractor. The base is frozen, which limits the damage relative to a full update, but the sum \(W_0 + \Delta W\) is still a different function. I keep a fixed slice of the old task and score it every epoch next to the new validation loss. If the old slice drops, I cut epochs, lower rank or learning rate, or mix rehearsal examples. I do not declare success from the new training loss alone.

**Follow-up.** Why can a higher rank make forgetting worse on a small dataset?

**Weak answer.** "Forgetting cannot happen because the original weights are frozen and therefore unchanged in the output."

## 10. A LoRA train loss goes to zero and production answers are confidently wrong. Where do you look?

**Intent.** Data quality over optimizer folklore.

**Strong answer.** I look at the data first. Zero loss means the adapter memorized the training continuations, including wrong labels, leaked answers, or a template that production will not send. I read a sample of pairs against the guideline, check that the loss mask covers only the answer, and compare the train template to the served template. I also check that I did not tune on the test set. Raising rank would fit the bad labels harder. A clean held-out set is the measurement that catches this.

**Follow-up.** The pairs look correct, and the served prompt adds a system line the training data never had. What do you change?

**Weak answer.** "Loss zero means the model is ready to ship. The production prompt must be at fault in a way training cannot explain."
