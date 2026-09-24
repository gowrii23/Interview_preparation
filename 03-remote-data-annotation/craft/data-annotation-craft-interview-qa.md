# Data-annotation craft: interview Q&A

### Q1. What is your job on a labeling task, in one sentence?

**Intent:** Spec-following versus taste.

**Strong sample answer:** I apply the project's schema so that another expert with the same guideline would produce the same label for the same cited reason. My seniority matters only when it helps me see evidence. It does not replace the schema.

**Follow-up:** Where does personal judgment still enter?

**Strong follow-up:** On true gaps, and only as an explicit assumption or an escalation, not as a silent house rule.

**Weak answer:** I improve the model's answers until they match industry best practice.

### Q2. How do you read a guideline you have never seen?

**Intent:** Method in the first hour.

**Strong sample answer:** I extract priority order, hard fails, the tie rule, axis definitions, and two borderline examples. I restate them in my own words and check that I did not add a rule. I label the official samples before I touch the queue, and I compare. Only then do I start timed work. If the prose is long, the extract is what I keep on screen, and I return to the source when an item hits a phrase I did not extract.

**Follow-up:** The guideline contradicts itself.

**Strong follow-up:** I follow the section it tells me wins, if it says. Otherwise I escalate the contradiction with both quotes and I do not average them.

**Weak answer:** I skim the introduction and rely on experience.

### Q3. Give an instruction-following miss that looks minor and is not.

**Intent:** Constraint sensitivity.

**Strong sample answer:** The user asked for a method that returns an empty list when there are no matches, and the model throws `NoSuchElementException` after explaining the domain well. The explanation can be clear and still fail the contract. I would also fail "return only the command" when the answer wraps it in a tutorial. Format constraints are part of the task, not decoration. I mention the constraint I am enforcing so the reviewer sees it is the guideline and not my preference for terse answers.

**Follow-up:** The exception is documented as a reasonable API.

**Strong follow-up:** Reasonable is irrelevant if the user specified the return. I follow the user constraint unless the project guideline says to override unsafe or impossible requests.

**Weak answer:** Close enough if the idea is right. Users can adapt.

### Q4. How do you justify a label?

**Intent:** Citation, not vibes.

**Strong sample answer:** I use a fixed shape: deciding criterion, evidence from the output, label, and if needed a note on what I saw and did not score. Example: "Correctness outranks style. On input `[]` the method returns null, but the prompt requires an empty collection. The other response returns an empty list and passes the non-empty example implied by the prompt. I prefer the second." I do not write that I personally dislike null returns in general, even though I do.

**Follow-up:** The UI gives you almost no rationale box.

**Strong follow-up:** I still include criterion and evidence, compressed to one or two sentences. I cut adjectives first.

**Weak answer:** I write a detailed code review so they see my range.

### Q5. What do you do when two raters would reasonably disagree?

**Intent:** Disagreement handling.

**Strong sample answer:** I check whether the disagreement is about facts, about priority, or about a missing rule. Facts can be settled by the artifact. Priority should already be in the guideline. A missing rule goes to the comment or escalation path with both readings. I do not message other raters to align votes on a live item. If calibration later picks a side, I adopt it going forward and update my extract.

**Follow-up:** You think the consensus is wrong.

**Strong follow-up:** I comply while I ask for a revision with a single counterexample. I do not run a private standard in parallel. Two standards is how overlap scores collapse.

**Weak answer:** I stick to my view. Consensus dilutes senior judgment.

### Q6. How do golden questions affect your behavior?

**Intent:** Anti-gaming.

**Strong sample answer:** They do not change item-level behavior. I cannot reliably know which items are golden, and I should not try. Every item gets the same checklist. When I am told I missed one, I classify the miss: domain fact, ignored priority, unknown convention, or a possible key error. I fix the first three in my extract. I escalate the fourth with quotes. I never build a list of "tricks the test uses."

**Follow-up:** You spot an item that matches a sample from onboarding.

**Strong follow-up:** I apply the guideline again from scratch. Memory of the sample is fine only if it is the same rule, not a shortcut that skips reading.

**Weak answer:** I slow down only when an item looks like a test, and I rush the rest.

### Q7. How do you balance time and quality?

**Intent:** Sustainable pace.

**Strong sample answer:** Mandatory steps are the ask, the hard fails, the named edge, the label, and a short evidence note. Optional steps are style advice and extra alternatives. I watch my miss pattern more than the clock. After a miss I slow down on that criterion, not on everything forever. Ambiguity gets an escalation instead of a five-minute essay. I stop a session when I notice middle-score collapse or a habit of always picking the bottom candidate, because that is fatigue, not judgment.

**Follow-up:** Your hourly count is below the project hint.

**Strong follow-up:** I look for over-writing first. If the checklist itself does not fit, I ask whether I am staffed on the wrong queue rather than skipping checks.

**Weak answer:** Quality is whatever speed the leaderboard rewards.

### Q8. Choose a schema feature and explain a misuse.

**Intent:** Schema literacy.

**Strong sample answer:** Multi-axis Likert is misused when someone averages helpfulness and harmlessness into one feeling. A response can be a 5 on fluency and a 1 on policy. If I submit a single high score, I have deleted the safety signal. Pairwise labels are misused when I pick A because it was on the left after a long day. Rankings are misused when I force a total order even though the guide allows two outputs to tie on the deciding dimension. I name the schema out loud before I click.

**Follow-up:** The project asks for one overall score plus axes.

**Strong follow-up:** I score axes first from their anchors, then compute the overall the way the guide says, not the way that flatters the answer I liked.

**Weak answer:** All of these scales are basically stars, like a review.

### Q9. What edge cases do you check on code items by default?

**Intent:** Senior-engineer edge habit, bounded by the prompt.

**Strong sample answer:** I check edges the prompt owns: empty and single-element collections, null only if the contract mentions it, error versus throw, off-by-one, and whether tests assert the failure. I look at security only when input crosses a trust boundary the task cares about: query concatenation, command building, secrets in logs. I do not fail a snippet for lacking distributed tracing. That is how a tech lead wastes a rubric. If I cite a bug, I include the input and the actual versus required behavior.

**Follow-up:** The code is incomplete but the algorithm is right.

**Strong follow-up:** I follow the guide's rule for partial solutions. Often instruction following fails if the user asked for compiling code, while the approach can still score on a separate reasoning axis. I do not invent a combined score.

**Weak answer:** I always review it as if it were a pull request to my team's main branch.

### Q10. Why does this craft matter to model training?

**Intent:** Connect labels to downstream use without overclaiming research.

**Strong sample answer:** Preference labels and ratings become supervision or evaluation data. If I reward length, the model learns to be long. If I reward sycophancy, it learns to agree. If I miss a wrong Java boundary, it learns that the boundary is fine. I have not trained the downstream model myself. I do understand that noisy or biased labels do not average out just because many raters are confident. My responsibility is a spec-true label and an honest limit when I am outside my domain.

**Follow-up:** Then whose job is the model?

**Strong follow-up:** The lab's. Mine is the integrity of the examples I touch. Blurring those roles in an interview is the oversell I am trying to avoid.

**Weak answer:** My labels directly control the model weights the next day, so they should match my personal bar.
