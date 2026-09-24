# Evaluation and alignment — interview Q&A

## 1. What does SFT optimize, and what can it not learn from a single demonstration?

**Intent.** Imitation versus preference.

**Strong answer.** Supervised finetuning minimizes next-token loss on demonstrations, usually masked to the assistant span. The model imitates the answer you wrote, including format and the habit of answering when you always answered. It does not learn a ranking between two replies unless that comparison is written as text. If the demos are narrow or wrong, SFT copies them. I treat label review as the main lever, and I keep a holdout the trainer cannot use for early stopping.

**Follow-up.** Why can a helpful-only SFT set increase unsafe completions?

**Weak answer.** "SFT is reinforcement learning with a reward model built in."

## 2. Explain RLHF without deriving PPO. What is the reward model, and why is there a KL term?

**Intent.** The three pieces: preferences, reward, constraint.

**Strong answer.** People label which of two completions they prefer. A reward model is trained so the preferred completion scores higher on that prompt. The policy is then optimized to increase expected reward while a KL penalty keeps it close to a reference policy, usually the SFT model. The KL term matters because the reward model is only trustworthy near the text it saw. Without the penalty, the policy can drift into gibberish or a loophole that the reward model scores highly: length, flattery, or a stock phrase. PPO is one way to optimize that objective. The idea is reward plus a tether, not a specific optimizer brand.

**Follow-up.** Give an example of reward hacking you would actually instrument.

**Weak answer.** "RLHF means the model talks to a human in the loop at inference and learns online."

## 3. Why can DPO train from preference pairs without a separate reward model?

**Intent.** The reparameterization, stated honestly.

**Strong answer.** Under a KL constraint to a reference policy, the optimal reward can be written as beta times the log probability ratio between the policy and the reference, plus a term that depends only on the prompt. In a pairwise comparison that prompt term cancels. The preference model then becomes a classification loss on the policy: raise the margin of the winning completion over the losing one, relative to the reference. You never fit a scalar reward head or run an on-policy RL loop. You still need pairs, a reference, and a held-out eval. DPO will fit a biased preference, such as "always longer," just as a reward model would.

**Follow-up.** What breaks if you drop the reference policy and only maximize the winner's likelihood?

**Weak answer.** "DPO is SFT on the winning answers, and the losing answers are ignored."

## 4. How do you measure hallucination on a question-answering product that shows sources?

**Intent.** A defined unit and a denominator.

**Strong answer.** I define the unit as a claim or a sentence. Given the retrieved passage, I label supported, contradicted, or not present, and I score citation precision and recall separately: did the cited span support the claim, and were the needed spans cited. Abstention is not the same error as invention. I report rates with the mix of answerable and unanswerable items. An NLI model or an LLM judge can scale labeling only after I measure agreement with humans on this corpus. A single "hallucination percent" without that definition is not a metric I would ship on.

**Follow-up.** Retrieval was wrong and the model faithfully summarized it. Whose error is that?

**Weak answer.** "I compute BLEU against Wikipedia. Low BLEU means hallucination."

## 5. What makes a golden set useful, and how do teams accidentally turn it into a training set?

**Intent.** Holdout discipline.

**Strong answer.** A golden set is a fixed, reviewed collection that matches production: languages, lengths, tool failures, and unanswerable items. I split development from final. I use development to edit prompts and pick checkpoints. I report the final slice once. The set becomes a training set when I add today's failure, retune until it passes, and quote the score. I also refresh the set when traffic changes, because a stale set is the opposite mistake: a clean number about a product I no longer run.

**Follow-up.** How do you include a rare safety failure without letting it dominate the average?

**Weak answer.** "The golden set is whatever the model got wrong this week, relabeled until accuracy is high."

## 6. What are two LLM-as-judge pitfalls, and how do you detect them?

**Intent.** Position, verbosity, and circularity.

**Strong answer.** Position bias: the judge prefers whichever answer is shown first or second. I detect it by swapping order and counting flips. Verbosity and self-preference: longer answers, or answers that sound like the judge, win even when a short answer is righter. I detect it with pairs that are equal except for length, and with a human sample. I also do not ask a judge to grade facts it does not know, and I do not treat the candidate's own family of models as an unbiased judge of itself. I report agreement with humans before a judge can block a release.

**Follow-up.** When is a pairwise judge more defensible than a 1–5 score?

**Weak answer.** "GPT grading GPT is objective because the grader is larger."

## 7. How do you run a human preference eval so a small gap is believable?

**Intent.** Guidelines, blinding, agreement.

**Strong answer.** I write a guideline with accept, reject, and borderline examples. Raters are trained and blind to which system produced which side. I show the same prompt side by side rather than collecting isolated 1–5 scores. I measure inter-rater agreement before I interpret a two-point gap. Domain experts grade domain claims. I use the human sample to calibrate an automatic judge, and I do not rescore every checkpoint by hand. Disagreement in the guideline is a spec bug, not a rater failure to hide.

**Follow-up.** Raters agree with each other and disagree with your spec. What do you fix first?

**Weak answer.** "I ask one teammate which answer feels better and average three chats."

## 8. What belongs in a safety eval that a helpfulness score will miss?

**Intent.** Compliance, over-refusal, tool actions.

**Strong answer.** A separate slice of disallowed requests, benign lookalikes that share keywords, and jailbreak wrappers. I score harmful compliance and over-refusal separately so a model cannot win by refusing everything. For tool-using systems I include cases where the harmful step is a tool call, because a polite paragraph can still emit the call. After a finetune on helpful demos I rerun this slice as a forgetting check. I do not invent a pile of harmful training text in order to "see what happens" without a review process. The eval set and the refusal policy should be ones I am allowed to store and describe.

**Follow-up.** Over-refusal went up and compliance went down. Is that a ship?

**Weak answer.** "If the helpfulness rating is high, the model is safe enough."

## 9. SFT, RLHF, and DPO all disappoint on the same bug in your labels. What is the bug, and why does a fancier method not remove it?

**Intent.** Data quality dominates.

**Strong answer.** The labels encode the wrong preference: every unanswerable question has a fluent answer, or the longer answer always wins, or one template leaked into the inputs. SFT imitates it. A reward model learns it. DPO's implicit reward learns it. More rank or a more elaborate optimizer fits it more thoroughly. I stop training new variants and review disagreements, unanswerable items, and length bias in the pairs. The method changes how comparisons are consumed. It does not create a standard of correctness that was never in the data.

**Follow-up.** How would you test for length bias in a preference set before you train?

**Weak answer.** "The loss went down, so the labels must have been fine. Alignment fixes noisy data."

## 10. You may ship either a prompt change or a DPO run. What evidence do you require for each?

**Intent.** Tie alignment work to an eval plan.

**Strong answer.** For the prompt, a versioned golden set, fixed decoding, and a written contract for abstention and format, with the injection and empty-input cases included. For DPO, the same evals plus a documented pair guideline, a reference policy, a check that winners are not just longer, a forgetting or safety slice, and a final test I did not use to pick the checkpoint. I want human agreement on a sample if an automatic judge is in the loop. I ship the smaller change that moves the product metric without a safety regression, and I keep the data hash next to the number.

**Follow-up.** DPO wins on the preference val set and loses on groundedness. Which number blocks the release?

**Weak answer.** "DPO is the newer method, so I ship it if training finishes."
