# Rubrics, writing, and calibration: interview Q&A

### Q1. What makes a dimension usable?

**Intent:** Observable criteria.

**Strong sample answer:** A usable dimension is one decision a second rater can make from the artifact. "Correctness on the stated contract" is usable if I define the edges. "Senior engineering quality" is not, because it imports each rater's employer. I also check that two dimensions are not the same thing renamed. If clarity and correctness always move together in my examples, I have not specified them. I name out-of-scope items, especially style, so they do not leak back in.

**Follow-up:** How many dimensions do you want?

**Strong follow-up:** Three or fewer graded ones, plus hard fails that sit outside the average. More than that and calibration becomes a debate club.

**Weak answer:** I include every best practice I know so the rubric is complete.

### Q2. Why are weights not enough for safety?

**Intent:** Hard fails versus averages.

**Strong sample answer:** A weight still lets other dimensions compensate. If safety is 30 percent and clarity is 20, a charming violation can outscore a plain safe refusal once someone fills in high marks everywhere else. If the client treats policy violations as unacceptable, I write a gate: safety fail means the overall label is a fail, and the remaining dimensions are diagnostic only. I put that in the first lines of the rubric. A footnote will be missed at hour six.

**Follow-up:** The client insists on a single 1-to-5 with no gate.

**Strong follow-up:** Then the 1 anchor must be the safety fail, with an example, and the instructions must say not to lift it for good prose. I would still push once for an explicit gate because single scales get laundered.

**Weak answer:** A weighted average is always fairer because it uses all the information.

### Q3. Write a 1 and a 5 for code correctness, out loud.

**Intent:** Anchors, not adjectives. Original example.

**Strong sample answer:** Contract: `countPositive` returns 0 for an empty list and throws `NullPointerException` for null. A 1 is the method that throws on empty because it lumps empty with null, even if it uses a stream. A 5 is the method that throws only on null, skips or rejects elements only as the contract allows, and returns 0 when there is nothing to count. I state the counterexample next to the 5: idiomatic code that fails empty is not a 5. I do not mention whether I prefer loops.

**Follow-up:** Where does a missing test sit?

**Strong follow-up:** On a separate dimension if tests were required. I do not silently drop correctness from 5 to 3 for a missing test unless the anchor says that.

**Weak answer:** 5 means production ready. 1 means I would reject the pull request.

### Q4. How do you stop style from capturing the rubric?

**Intent:** Senior-engineer failure mode.

**Strong sample answer:** I put style in the out-of-scope list with examples: stream versus loop, comment density, import order. I add a worked item where the plain correct answer beats the fluent wrong one, and I show the rejected temptation. In calibration, if someone argues from "I would not merge this," I ask which dimension contains that preference. If none does, the preference does not move the label. My own Java habits are the ones I distrust most, because they feel like facts.

**Follow-up:** The client actually wants a style dimension.

**Strong follow-up:** Then it gets anchors of its own and a small weight, and it cannot override a correctness 1. "Matches the requested format" is instruction following, which I do not call style.

**Weak answer:** Style is part of correctness for professionals. Separating them is pedantic.

### Q5. Describe a calibration session that works.

**Intent:** Process, not personality.

**Strong sample answer:** Raters label the shared items privately first. We reveal scores and group the disagreements. Each split is a fact, a priority, or a missing sentence. We change the rubric by one sentence or one weight, then we rescore a fresh item independently to see if the split closed. The output is an updated document, not a memory of who won. Chat-only decisions get copied into the extract the same day. If a golden depends on the old rule, we flag it for rebuild.

**Follow-up:** Two seniors still disagree after the edit.

**Strong follow-up:** We do not vote by title. We either find an observable difference or we explicitly allow a tie for that pattern. Leaving it as "use judgment" recreates the split on the queue.

**Weak answer:** The most experienced person explains the right score and everyone aligns.

### Q6. What do you do with disagreement after the rubric is published?

**Intent:** Compliance plus escalation.

**Strong sample answer:** I apply the written rule on the live item. If I think the rule is wrong, I submit the label the rule requires, and I send one counterexample through the review path. I do not run a private exception for cases that "obviously" differ. I do not organize other raters to ignore the rule. Dotted-line escalation is part of the design. Silent rebellion looks like random disagreement in the metrics and is worse than a wrong rule you can see.

**Follow-up:** The reviewer is slower than the queue.

**Strong follow-up:** I keep following the current text. Urgency is not a reason to freelance. I can ask for a temporary tie convention if the project allows one while the review is open.

**Weak answer:** I use my best judgment until someone replies, and I do not mention the conflict in the rationale.

### Q7. How would you rubric an agentic defect-triage trace?

**Intent:** Domain theme, honest scope, no fake metrics.

**Strong sample answer:** I would gate on whether the trace invents evidence. Dimension one: the recommendation is supported by artifacts the trace actually retrieved. Dimension two: the suggested code or config change matches the stated failure, including the regression case. Dimension three, lighter: a human can see the decision without reading a novel. A 1 on evidence is "cites a file or log line that was not retrieved, or concludes with no evidence." A 5 is "every claim used in the conclusion appears in the retrieved material." I have worked on triage and agentic workflows in an enterprise Java setting. I would not attach a percentage improvement to that rubric. I never measured one for a marketing sentence.

**Follow-up:** The patch is right and the evidence is fabricated.

**Strong follow-up:** Evidence is a hard fail if the purpose is to train honest traces. A correct guess with fake support is how these systems become un-auditable.

**Weak answer:** I score the trace on how autonomous it felt.

### Q8. A junior rater applies your rubric differently. What do you assume first?

**Intent:** The rubric is at fault until proven otherwise.

**Strong sample answer:** I assume the text allows both readings. I ask them to point at the sentence they used. If both sentences exist, I delete or order them. If they missed a sentence that is actually clear, I still check whether the anchor example was too far from the item they saw. People do not calibrate on examples that do not resemble the queue. I do not open with "you lack experience." That may be true and it is not the first fix. The first fix is a sharper anchor.

**Follow-up:** It really was a careless miss.

**Strong follow-up:** Then feedback cites the sentence and the item, and I watch the next items of that type. One lecture does not update behavior. The extract does.

**Weak answer:** I assume they did not read, and I ask to have them removed.

### Q9. How do you explain rubric quality in AIML terms without overclaiming?

**Intent:** Literacy, not research cosplay.

**Strong sample answer:** A rubric is the measurement instrument for preference or evaluation labels. Vague anchors increase rater noise and systematic bias, and a reward model can learn that bias. I know this as an M.Tech AIML student and as someone who labels. I have not published a study on agreement rates, and I will not invent a kappa or an accuracy lift. My practical contribution is writing anchors a stranger can apply, especially on Java behavior, where "good code" is otherwise a status contest.

**Follow-up:** What statistic would you want the project to track?

**Strong follow-up:** Disagreement rate on overlapped items, broken down by dimension, plus golden accuracy after a rubric change. I would want those computed by the project. I would not fake them in a portfolio.

**Weak answer:** A good rubric boosts model quality by a large margin. I have seen it.

### Q10. What does a weak rubric sound like in an interview?

**Intent:** Self-check. Show you can hear the failure.

**Strong sample answer:** A weak answer says "I look for clean, scalable, production-grade solutions and score them holistically." Nothing there is observable, safety is missing, and holistic means the rater's taste. A stronger close is: "Hard fail on a broken contract or a forbidden sink. Then weighted behavior, required tests, and explanation fidelity, with a 1 and a 5 written as inputs and outputs. Ties only when those match. Style is out of scope. We calibrate by private labels first and we edit the sentence that caused the split."

**Follow-up:** Rewrite "production-grade" into a check.

**Strong follow-up:** "Does not throw on empty input when the contract returns 0, and does not build SQL by concatenating request parameters." Two checks beat one aura.

**Weak answer:** Production-grade means the kind of code I am proud to sign.
