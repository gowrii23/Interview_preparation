# Deep learning and transformers — interview Q&A

## 1. What is backpropagation actually computing?

**Intent.** Chain rule and why activations are stored.

**Strong answer.** A network is a composition of simple maps. Backprop applies the chain rule from the loss backward, multiplying local derivatives. The forward pass saves activations because those derivatives depend on them. The result is a gradient of the loss with respect to each weight. An optimizer then steps. I am not deriving one closed-form derivative for the whole net.

**Follow-up.** Why does gradient checkpointing save memory and cost compute?

**Weak answer.** "Backprop is how the model goes back and fixes the wrong neurons."

## 2. Why do vanishing gradients happen, and how does a residual connection help?

**Intent.** A mechanism, not the phrase "vanishing gradients."

**Strong answer.** If each layer multiplies the gradient by a factor smaller than one, early layers get a product near zero. Sigmoid units do this when they saturate. Training then updates the top of the network and barely moves the bottom. A residual block computes \(x + F(x)\). The derivative includes a 1 from the skip, so gradient can flow even when the derivative of \(F\) is small. Layer norm, initialization, and non-saturating activations are the other practical controls. Clipping is for the exploding case.

**Follow-up.** Where in a transformer block does that residual sit?

**Weak answer.** "Vanishing gradients mean the learning rate is too small, so I increase it."

## 3. Define query, key, and value, and write scaled dot-product attention.

**Intent.** The formula and the role of the scale.

**Strong answer.** From hidden states \(X\), \(Q = X W_Q\), \(K = X W_K\), \(V = X W_V\). The query is what a position looks for, the key is what a position offers as an address, and the value is the content passed forward. Attention is \(\mathrm{softmax}(Q K^\top / \sqrt{d_k}) V\). The \(\sqrt{d_k}\) keeps dot products from growing with dimension. Large unscaled scores make softmax nearly one-hot and its gradient tiny.

**Follow-up.** Two keys, one query, \(d_k = 1\), scores after scaling are 0 and 0. What are the weights?

**Weak answer.** "Query is the question, key is the keyword, value is the answer from the database."

## 4. Walk a tiny attention computation.

**Intent.** Can the candidate do the arithmetic, not only name softmax.

**Strong answer.** Take \(q = [1, 0]\), \(k_1 = [1, 0]\), \(k_2 = [0, 1]\), so dots are 1 and 0. With \(d_k = 2\), divide by \(\sqrt{2} \approx 1.41\), scores \(\approx 0.71\) and 0. Softmax weights are about \(e^{0.71}/(e^{0.71}+1) \approx 0.67\) and \(0.33\). If \(v_1 = [2, 0]\) and \(v_2 = [0, 3]\), the output is about \([1.34, 0.99]\). The aligned key contributes more of its value. I would mention that a causal mask would have removed a future key before this softmax.

**Follow-up.** What happens to the weights if you forget the scale and the dots are 10 and 0?

**Weak answer.** "The values get averaged with attention weights that the model learns directly as a table."

## 5. Why multi-head attention instead of one wider head?

**Intent.** Parallel subspaces, and a correct parameter intuition.

**Strong answer.** One attention distribution has to serve every relation between tokens. Multiple heads use separate \(W_Q, W_K, W_V\) at a smaller head dimension, each with its own softmax, then concatenate and project with \(W_O\). Different heads can pick up different offsets or copy different features. Splitting \(d_{\text{model}}\) across heads keeps the parameter count in the same order as a single full-width attention, so the win is representation, not a free lunch of \(h\) times the parameters. I would not claim a specific head is "the syntax head" without a measurement.

**Follow-up.** What is grouped-query attention changing: queries, or keys and values?

**Weak answer.** "More heads always means a bigger model and better accuracy."

## 6. Why does position have to be added at all, and what is one relative method?

**Intent.** Permutation invariance, plus a modern positional scheme.

**Strong answer.** Without position, self-attention treats the sequence as a set. Reordering tokens reorders outputs and the model cannot represent "the previous word." Absolute sinusoids or a learned position table add a function of the index to the token embedding. Rotary embeddings apply a rotation to queries and keys so the dot product depends on relative distance. A learned absolute table does not extend past its trained length. RoPE extrapolates better but does not remove the quadratic cost of dense attention or the fact that the model was only trained on certain lengths.

**Follow-up.** A user doubles the context window in the server config only. What can go wrong?

**Weak answer.** "The positional encoding is the order of the layers in the network."

## 7. Contrast encoder-only, encoder–decoder, and decoder-only.

**Intent.** Pick the right diagram for LLMs.

**Strong answer.** An encoder stack uses bidirectional self-attention and fits classification or embedding models trained with a masked-token loss. An encoder–decoder reads the source with an encoder and generates with a decoder that has causal self-attention plus cross-attention into the encoder. A decoder-only model is a stack of causal blocks. That is the GPT-style LLM: pretraining and generation are the same next-token computation, and a prompt is just a prefix. There is no separate comprehension module.

**Follow-up.** Where would cross-attention appear, and why is it absent in a decoder-only chat model?

**Weak answer.** "Encoders understand and decoders talk, so every LLM has both."

## 8. What do layer norm and the MLP each do inside a block?

**Intent.** The two sublayers and pre-norm.

**Strong answer.** A pre-norm block is \(x + \mathrm{Attention}(\mathrm{LN}(x))\), then \(x + \mathrm{MLP}(\mathrm{LN}(x))\). Layer norm rescales features within a token so depth is trainable. Attention mixes information across positions. The MLP is a position-wise expansion and projection, usually with a nonlinearity such as GELU, and mixes features at a single position. Pre-norm puts the norm on the residual branch, which trains more stably than normalizing after the add when the stack is deep.

**Follow-up.** If you removed the MLP and kept attention, what kind of mixing would you lose?

**Weak answer.** "Layer norm prevents overfitting and the MLP is just another attention."

## 9. Why did transformers scale when deep RNNs struggled?

**Intent.** Parallelism and the remaining bottleneck.

**Strong answer.** An RNN's hidden state at step \(t\) waits on step \(t-1\), so sequence length is a serial chain and long-range credit assignment is hard. In a transformer layer, all query–key scores in the window are independent and run as matrix multiplies, which matches GPUs. Residuals and layer norm make deep stacks trainable. The bill is quadratic dense attention in sequence length, and a large memory for activations in training. Scale improves next-token loss when data grows with compute. It does not by itself produce calibrated truth.

**Follow-up.** Why can decode still be slow if the matmuls are parallel?

**Weak answer.** "Transformers are faster because they have fewer parameters than RNNs."

## 10. What does a causal mask change in the attention softmax?

**Intent.** Generation correctness.

**Strong answer.** Before softmax, scores from position \(t\) to positions \(> t\) are set to \(-\infty\), so their weights become zero. The token cannot depend on future tokens. That matches left-to-right generation and next-token training. Bidirectional masking would leak the answer during pretraining and would not match autoregressive decoding. Padding positions are masked separately so they are not attended as if they were content.

**Follow-up.** During training, how can the loss for every position still be computed in one forward pass?

**Weak answer.** "The mask drops random tokens the way dropout does."
