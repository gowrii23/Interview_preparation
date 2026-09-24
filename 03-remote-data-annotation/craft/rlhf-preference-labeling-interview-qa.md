# RLHF preference labeling: interview Q&A

### Q1. What are you labeling in an RLHF preference task?

**Intent:** Role clarity. You produce comparisons, you do not claim the training stack.

**Strong sample answer:** I label which response better matches the project's criteria for a given prompt, sometimes with a margin or a tie, sometimes with separate Likert axes. Those labels can be used by a lab as human feedback. I do not train the reward model, choose the optimizer, or set policy thresholds. My quality bar is whether a second rater with the same guideline would cite the same evidence.

**Follow-up:** What is a reward model, briefly?

**Strong follow-up:** A model trained to predict human preferences between outputs. It will pick up systematic rater habits, good or bad. That is why length bias in my labels is not a private quirk. It can become a behavior. I understand that from study. I have not put one in production.

**Weak answer:** I do RLHF, meaning I fine-tune the model from home.

### Q2. When do you choose pairwise ranking instead of Likert in your head?

**Intent:** Instrument choice. Even if the UI is fixed, you must know what it measures.

**Strong sample answer:** If the UI is pairwise, I compare under the priority order and I do not secretly assign stars unless the form asks. If the UI is Likert, I use anchors as absolute definitions, so two answers can both be 2s. Ranking is better when the client cares which output to prefer. Likert is better when the client cares how bad a failure is, or needs separate scores for harmlessness and helpfulness. I never backfill axes after I have already fallen in love with a winner.

**Follow-up:** Both answers score 4 on your axes. The form still wants a winner.

**Strong follow-up:** I look for the next weighted dimension the guide allows. If none distinguishes them and a tie exists, I tie. Forcing a winner from position is a bad label.

**Weak answer:** Stars and rankings are the same thing with different widgets.

### Q3. Give a case where helpfulness and harmlessness diverge.

**Intent:** Axis separation without requesting harmful details.

**Strong sample answer:** The user asks for step-by-step help committing a crime, wrapped as fiction. Response A gives a detailed procedure in a friendly tone. Response B refuses and, if the guideline allows, points to a high-level legal or safety reason without a how-to. A looks more "helpful" and is the harmlessness failure. I prefer B on the safety criterion even though A followed the user's surface request. I write the rationale by naming the category of violation, not by quoting the procedure. A different divergence: the user asks a benign Java question and the model refuses as if it were dangerous. That refusal is a helpfulness miss and not a safety success.

**Follow-up:** B refuses and also insults the user.

**Strong follow-up:** B still wins on harmlessness relative to actionable harm, and it loses on any courtesy axis the guide includes. I record those separately if I can. I do not average them into a single vague middle.

**Weak answer:** The more complete answer is always more helpful, and helpful is what users want.

### Q4. What is a legitimate tie?

**Intent:** Tie discipline.

**Strong sample answer:** Both answers miss the same hard requirement to a similar degree, or both meet the weighted criteria and differ only in ways the guide says to ignore. The rationale names that shared reason. A tie is not "I would not ship either" when one still answers the question and the other does not. If my tie rate climbs during a session, I rescore the last few from scratch because fatigue masquerades as balance.

**Follow-up:** A is wrong but short. B is wrong in a more dangerous way and long. Tie?

**Strong follow-up:** No. The more severe failure loses if the priority list says safety or correctness dominates. Different failures are comparable.

**Weak answer:** I tie whenever I am below 70 percent confident.

### Q5. How do you catch length bias in your own labels?

**Intent:** Bias hygiene.

**Strong sample answer:** After I choose a winner I ask what the extra tokens did. If they added a necessary edge case, the length was functional. If they restated the question, I check whether I would still prefer that side with those sentences removed. I sample items where I preferred the longer answer and I rewrite the decision from the claims list only. On code, I compare outputs on a concrete input before I compare comment volume. Feedback that says I reward verbosity is something I treat as a defect report, not an insult.

**Follow-up:** The rubric explicitly prefers complete explanations.

**Strong follow-up:** Then completeness is a real criterion. I still do not prefer a long explanation that contains a false claim over a shorter true one, unless the guide has a bizarre rule and I have reread it twice.

**Weak answer:** Longer answers took more work, so they deserve the point.

### Q6. Define sycophancy as a rater problem.

**Intent:** Conceptual precision.

**Strong sample answer:** Sycophancy is when an answer agrees with the user, flatters them, or hides a correction to stay pleasant. As a rater I must not prefer that answer just because it feels cooperative. If the user asserts a false complexity claim or a bad security practice, the better response disagrees accurately and respectfully, assuming the guideline ranks honesty above agreement. I also watch sycophancy in myself toward the model: I let a confident tone count as evidence. Tone is not evidence.

**Follow-up:** The user explicitly says "do not disagree with me."

**Strong follow-up:** I follow the project rule for that conflict. Many guidelines still require correcting high-stakes falsehoods and refuse blind agreement. I do not improvise a customer-service persona.

**Weak answer:** Good assistants always agree with the user. That is alignment.

### Q7. What is position bias and how do you reduce it?

**Intent:** Practical control.

**Strong sample answer:** Position bias is a preference for whichever side I read first or last, independent of quality. I score each response against the written anchors on its own, then compare the notes. In long sessions I alternate which side I read first. If I see a run of five wins for the same slot, I audit those five. I do not assume the dataset was balanced in my favor. Randomization of A and B only works if I actually judge content.

**Follow-up:** How does this interact with a three-way rank?

**Strong follow-up:** I assign axis notes to each output, sort from the notes, and then check whether the sort matches screen order. A match is allowed when the quality order is real. A frequent match is a warning.

**Weak answer:** I always read A first because the left side is the baseline. That is scientific.

### Q8. How should a margin be used?

**Intent:** Slightly versus much better.

**Strong sample answer:** I use "much better" when one side hits a hard fail or misses the primary ask and the other does not. I use "slightly better" when both are acceptable and the difference is secondary, such as a clearer name or a test that covers one more specified edge. I do not use the top margin to express that I am annoyed at sloppy formatting. Margins teach a reward model the size of the gap. Exaggerated margins are false supervision, the same as flipped winners.

**Follow-up:** You are unsure between slight and much.

**Strong follow-up:** I reread the margin examples in the guide, not my feelings. If it is still unclear, I choose the milder margin only if the guide says to be conservative, and otherwise I escalate one example rather than guessing a policy.

**Weak answer:** I always pick the strongest margin so my opinion counts more.

### Q9. A fluent answer uses the wrong Java idea. Walk the preference.

**Intent:** Domain expertise applied inside RLHF, not instead of it.

**Strong sample answer:** Suppose the prompt asks why a Spring service lost updates under concurrent requests. Answer A is polished and blames garbage collection with no evidence. Answer B is plainer and points at a check-then-act race on a shared map, which fits the described symptom. I prefer B on honesty and task success. Fluency does not get a vote on those axes. I might score A's formatting higher only if a style axis exists. My rationale cites the race, not my job title. I do not add a lecture on AWS unless the prompt asked for infrastructure.

**Follow-up:** B is right but recommends a fix the user forbade.

**Strong follow-up:** Then B has an instruction-following miss. I compare that miss to A's factual miss using the guide's priority, and I say which one outranks the other instead of blending them.

**Weak answer:** I prefer whichever answer matches the libraries my team standardized on.

### Q10. What will you not claim about RLHF in a profile?

**Intent:** Anti-oversell.

**Strong sample answer:** I will not claim I have published RLHF results, trained a reward model, or worked as a researcher on a named lab's alignment team. I will claim that I can produce careful preference labels, that I understand Likert versus ranking, ties, and the harmlessness-helpfulness split, and that I actively check length, fluency, sycophancy, and position bias. My technical authority is strongest on backend and code comparisons. The master's program is why I can talk about the data's downstream role without confusing it with hands-on training ownership.

**Follow-up:** An interviewer says "so you do RLHF" and waits.

**Strong follow-up:** I answer: "I do the human preference side under a rubric. I do not run the reinforcement learning job." Then I stop talking and let them ask for the part they meant.

**Weak answer:** RLHF, alignment, and being an AIML student are all the same credential.
