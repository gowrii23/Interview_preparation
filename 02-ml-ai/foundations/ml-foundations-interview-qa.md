# ML foundations — interview Q&A

Ten questions for an applied AIML interview. Each item has the interviewer's intent, a strong answer, a follow-up, and a weak answer.

## 1. How do supervised, unsupervised, and self-supervised learning differ?

**Intent.** Check whether the candidate separates "no human label" from "no target."

**Strong answer.** Supervised learning trains on inputs paired with labels a human or a system provided, and the loss compares the prediction to that label. Unsupervised learning has no target; clustering or density estimation looks for structure in the inputs alone. Self-supervised learning builds a target from the input by a rule: predict the next token, or predict a masked token. The optimization is still a supervised loss, but the label is free. Next-token pretraining is self-supervised. Instruction tuning is supervised, because someone wrote the desired response.

**Follow-up.** If next-token training has a target, why do people still call pretraining unsupervised?

**Weak answer.** "Unsupervised means deep learning without labels, and GPT is unsupervised."

## 2. Training loss is low and validation loss is high. What is going on, and what do you change?

**Intent.** Diagnose overfitting versus leakage or a broken split.

**Strong answer.** The model fits the training sample and not the held-out sample. That is the high-variance pattern: overfitting, or a train/validation leak that makes training easier than it looks, or a validation set from a different distribution. I would plot both curves, check for duplicates across the split, and confirm the split unit is the user or document we will see in production. Remedies are more or better data, earlier stopping, stronger regularization (weight decay, dropout, a smaller model, or a frozen base with a small adapter), and a lower learning rate or fewer epochs. I would not touch the test set while doing this.

**Follow-up.** When would you call the same curves underfitting instead?

**Weak answer.** "Add more layers until validation accuracy goes up."

## 3. What is the bias–variance tradeoff in a decision you would actually make?

**Intent.** See the idea used, not recited.

**Strong answer.** Bias is error from a hypothesis that cannot represent the pattern. Variance is error from fitting the particular sample. A linear model on a nonlinear decision has high bias. A large network on a few hundred noisy labels has high variance. If validation error is high and training error is also high, I add capacity or features or train longer. If training error is low and validation error is high, I constrain the model or get more data. At finetune time, choosing LoRA rank is this tradeoff: small rank is a biased update subspace; large rank on a tiny set is high variance.

**Follow-up.** Where does label noise sit in that decomposition?

**Weak answer.** "Bias is when the dataset is biased, variance is when accuracy moves."

## 4. Why is accuracy a bad headline for rare events, and what do you report instead?

**Intent.** Precision, recall, and the operating point.

**Strong answer.** If the positive class is 0.1% of rows, a constant "negative" predictor is 99.9% accurate and never finds the event. Precision is the fraction of flagged items that are truly positive. Recall is the fraction of positives you caught. F1 summarizes them with a harmonic mean, which is useful for a single comparison and useless for choosing a threshold. I report precision at the recall the product requires, or a precision–recall curve, and I set the threshold on validation using the cost of a false positive versus a false negative.

**Follow-up.** The positive class is rare in production but you upsampled it in training. What do you watch?

**Weak answer.** "I would use F1 because it is better than accuracy."

## 5. What is calibration, and why is a language model's token probability not a calibrated fact score?

**Intent.** Separate ranking, accuracy, and probability meaning.

**Strong answer.** A calibrated classifier's stated probability matches the observed frequency: among items scored 0.8, about 80% are positive. You can have excellent ranking and poor calibration. Temperature scaling on a validation set is a simple post-hoc fix for a classifier. A language model's next-token probability is the probability of that token under the training distribution, not the probability that a claim is true. A fluent falsehood can have high probability. If I need a confidence for a decision, I measure it on a labeled set or use a separate verifier.

**Follow-up.** How would you check calibration on a binary production classifier?

**Weak answer.** "If the softmax is 0.99, the model is sure, so the answer is right."

## 6. Give two concrete leakage paths and how you would prevent each.

**Intent.** Practical split discipline, including LLM contamination.

**Strong answer.** First, the same user or near-duplicate document lands in train and test because the split was a random row split. I split by user or document, and I deduplicate with a similarity check, not only exact string match. Second, the scaling statistics or the vocabulary were fit on the full frame before the split, or the benchmark was in the pretraining corpus. I fit preprocessing on train only, and for LLMs I treat unexpected benchmark jumps as contamination until I can show the eval text was absent and the split is clean. I also keep a final test set that is not used to pick checkpoints.

**Follow-up.** How does time order change the split for a support-ticket model?

**Weak answer.** "I shuffle and use an 80/10/10 split, so there is no leakage."

## 7. What does L2 regularization change, and why do LLM trainers mention AdamW specifically?

**Intent.** Regularization mechanics, not a list of names.

**Strong answer.** L2 adds a term proportional to the squared weight norm, which pulls weights toward zero and reduces the chance that a few large weights memorize noise. In classic Adam, putting that penalty inside the gradient interacts badly with the adaptive second moment. AdamW applies weight decay directly on the weights, decoupled from the adaptive step, which is why it is the default in transformer training. Weight decay is one regularizer among others: early stopping, dropout, more data, and freezing most parameters.

**Follow-up.** If weight decay is very large, what failure do you expect on the training loss?

**Weak answer.** "Regularization makes the model more accurate on the training set."

## 8. How do you handle class imbalance without fooling yourself at threshold time?

**Intent.** Training distribution versus production distribution.

**Strong answer.** I confirm the metric is precision–recall, not accuracy. On the training side I may use class weights, focal loss, or resampling so the rare class influences the gradient. Resampling changes the prior. I therefore choose the decision threshold on a validation set that matches production frequencies, not on the resampled training distribution. I would rather collect more positive examples than stack resampling tricks. I also check calibration after reweighting, because the scores may no longer mean frequencies.

**Follow-up.** Name a case where missing a positive is cheap and a false positive is expensive.

**Weak answer.** "Oversample until the classes are 50/50 and then ship the 0.5 threshold."

## 9. What is an embedding, and what can "nearby" mean that you did not intend?

**Intent.** Geometry plus the training signal.

**Strong answer.** An embedding maps a discrete object to a vector so that a dot product or cosine reflects a trained notion of similarity. Token embeddings give symbols a geometry the rest of the network can use. A retrieval embedding puts texts near each other when they matched the training pairs, which might be clicks, duplicates, or labels. Nearby does not mean true, recent, or safe unless those were the training signal. I would state what the vectors were trained on before using them in search or as features.

**Follow-up.** Why can two sentences with the same meaning be far apart in a raw token-average?

**Weak answer.** "Embeddings are compressed text, so cosine similarity is semantic meaning."

## 10. You must explain cross-entropy to a teammate who ships features. What do you say?

**Intent.** Connect the loss to probability.

**Strong answer.** The model outputs a probability \(p\) for the correct class or the correct next token. Cross-entropy is \(-\log p\). If the model puts probability 1 on the right answer, the loss is 0. If it puts probability 0.1 there, the loss is about 2.3 nats. Minimizing it pushes probability mass onto the observed target and, as a side effect, discourages being confidently wrong. It does not encode business cost. A rare class and an expensive error still need weights, a threshold, or a different decision layer on top.

**Follow-up.** Why is the log, rather than \(1 - p\), the usual choice?

**Weak answer.** "Cross-entropy is the loss functions use for classification because it is standard."
