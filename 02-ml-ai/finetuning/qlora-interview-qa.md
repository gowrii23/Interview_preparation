# QLoRA — interview Q&A

Questions 2, 3, 5, and 8 ask for a memory or freeze calculation.

## 1. What is QLoRA, precisely?

**Intent.** Separate the paper's stack from "LoRA plus a cast."

**Strong answer.** QLoRA is LoRA on a base stored in 4-bit NormalFloat, with double quantization of the scale constants and paged optimizers for memory spikes. The base stays frozen. Adapters train in higher precision, usually bfloat16. The forward pass dequantizes weights to the compute dtype, then adds \((\alpha/r) BA x\). Saying only "we quantize and then finetune" misses which tensors move.

**Follow-up.** Which of those pieces reduces optimizer memory, and which reduces weight memory?

**Weak answer.** "QLoRA means the gradients are 4-bit."

## 2. During a QLoRA step, which tensors are 4-bit, which are bf16, and which receive gradient?

**Intent.** Correct the "trained in 4-bit" misstatement.

**Strong answer.** The frozen base is stored as 4-bit NF4 codes plus scales. For the matmul those codes are dequantized to bf16. The LoRA matrices \(A\) and \(B\) are bf16 parameters and they receive the gradient. Adam moments for \(A\) and \(B\) are fp32 in a typical setup. The 4-bit codes get no gradient and no optimizer state. Activations are in the compute dtype. If someone updates the 4-bit base with SGD, that is a different and much riskier method, not QLoRA.

**Follow-up.** Why is a useful update a poor fit for a 4-bit code anyway?

**Weak answer.** "The whole network, including the adapter, is stored and updated in 4-bit."

## 3. Sketch bytes for the base weights of a 7 billion parameter model in bf16 versus 4-bit payload, and say what you left out.

**Intent.** Memory math without fake benchmark scores. 7e9 * 2 = 14e9 bytes ≈ 14 GB. 7e9 * 0.5 = 3.5e9 bytes ≈ 3.5 GB.

**Strong answer.** bf16 is 2 bytes per parameter: \(7 \times 10^9 \times 2 \approx 14\) GB for the weights alone. A 4-bit payload is 0.5 bytes per parameter: about 3.5 GB. I left out quantization scales, embeddings that may stay in higher precision, activations, the KV cache if I were serving, and the adapter's Adam state. Double quantization cuts the scale overhead from about 0.5 bits per weight on a block of 64 with fp32 scales toward roughly 0.13 bits per weight. It does not turn 3.5 GB into megabytes. Full finetune would also store gradients and Adam moments on all 7 billion parameters, on the order of 12 or more extra bytes per parameter, which is the bill QLoRA avoids.

**Follow-up.** Your GPU is 24 GB and the 4-bit base is ~3.5 GB. Why can the run still run out of memory?

**Weak answer.** "4-bit means the model is a quarter of the size, so a 7B model is about 7 GB and training will fit anywhere."

## 4. Why NormalFloat rather than uniform int4?

**Intent.** Codebook shape.

**Strong answer.** Weights are concentrated near zero, roughly bell-shaped. Uniform int4 spends equal width on the tails and the center, so the center, where most values sit, is coarse. NF4 chooses its 16 levels from quantiles of a normal, which puts more resolution near zero, then a per-block scale matches the actual magnitude. It is not the same algorithm as int4, and it is not a claim that every activation is standard normal. I would not compare an nf4 run to an int4 run as if only the seed changed.

**Follow-up.** What does the per-block scale do that the codebook alone cannot?

**Weak answer.** "NF4 is float4, so it has an exponent and a mantissa like fp16."

## 5. Walk the double-quantization overhead for block size 64.

**Intent.** Derive the bits, do not recite a speedup.

**Strong answer.** One fp32 scale per 64 weights costs \(32/64 = 0.5\) bits per weight on top of the 4-bit payload. Double quantization stores those scales in 8 bits and keeps a second-level fp32 scale for a group of 256 first-level scales. Overhead is \(8/64 + 32/(64 \times 256) \approx 0.125 + 0.002 = 0.127\) bits per weight. The weight codes stay 4 bits. The savings is the constants. Attributing a large throughput gain only to double quantization mixes this overhead with the jump from 16-bit storage to 4-bit storage.

**Follow-up.** If the block size were 32 instead of 64, would the fp32 scale overhead rise or fall?

**Weak answer.** "Double quantization stores each weight twice, once in 4-bit and once in 2-bit."

## 6. What do paged optimizers actually page, and what do they not speed up?

**Intent.** Unified memory versus a faster Adam.

**Strong answer.** Paged Adam uses CUDA unified memory so optimizer-state pages can spill to CPU when gradient checkpointing spikes GPU memory, then come back. The moments are still first and second moments of the adapter parameters. Paging does not shrink that state and it does not make the step faster. If the steady-state working set does not fit, the run thrashes. I still estimate activation memory from batch and sequence length instead of treating paging as infinite VRAM.

**Follow-up.** Which tensors in QLoRA are large enough that their Adam state is worth paging, and which are already small?

**Weak answer.** "Paging quantizes the optimizer to 4-bit on CPU."

## 7. When would you choose full finetune, bf16 LoRA, or QLoRA?

**Intent.** A decision, not a ranking of papers.

**Strong answer.** Full finetune if the behavior needs a large change, I have the memory, and I will regression-test old skills. bf16 LoRA if the base fits in 16-bit and I want adapter math that merges exactly onto the weights I trained against. QLoRA if the 16-bit base does not fit or I need the leftover memory for sequence length. I expect QLoRA quality to be close to bf16 LoRA on the same pairs and I verify that on my validation set, because the forward pass sees dequantized weights. I do not promise they are identical.

**Follow-up.** You have 80 GB and a 7B model. Why might you still use LoRA rather than full finetune?

**Weak answer.** "QLoRA is always higher quality because 4-bit is a regularizer."

## 8. After QLoRA, how do you merge for serving? What is inexact?

**Intent.** No in-place add into 4-bit codes.

**Strong answer.** I dequantize the frozen base to bf16, add \((\alpha/r) BA\), and save the dense matrix for vLLM or TGI. I do not add the low-rank product into the 4-bit codes in place. The dequantized base is already an approximation of the original checkpoint, so the merge matches the training graph's base, not the pre-quantization weights bit for bit. If I quantize the merged model again for serving, that is a second quantization error and I evaluate the artifact I will call, not the unmerged training loss.

**Follow-up.** Can you keep five QLoRA adapters on one loaded 4-bit base without merging? What is frozen in that setup?

**Weak answer.** "Merge writes the adapter back into NF4 so there is no extra error."

## 9. Name a misstatement you would correct in a design review.

**Intent.** Conceptual accuracy under social pressure.

**Strong answer.** I would correct "the model is finetuned in 4-bit." The stored base is 4-bit and frozen; compute is bf16; only adapters train. I would also correct a claim that double quantization halves the model again, or that paged Adam reduces the number of moments. I would ask which 4-bit type is configured, nf4 or int4, and whether the reported metric is the merged bf16 model or the dequantized training path.

**Follow-up.** The loss curve looks healthy and the merged model is worse. What do you compare first?

**Weak answer.** "Those details do not matter if the library checkbox says QLoRA."

## 10. QLoRA training loss is good and a long-context validation batch OOMs. What do you change first?

**Intent.** Activations still dominate.

**Strong answer.** Weight storage is no longer the bottleneck. I shorten the sequence, enable gradient checkpointing, reduce batch size and raise gradient accumulation if I need the same effective batch, or disable packing that accidentally builds very long sequences. I do not raise rank to fix an OOM. I do not turn on paging and ignore a throughput collapse. I log sequence-length histograms so a few long examples cannot hide inside the mean.

**Follow-up.** Checkpointing is on and the step time doubled. Is that a bug?

**Weak answer.** "OOM means the 4-bit quantization did not apply, so I quantize the adapters too."
