# Remote AI-training work: interview Q&A

Use these as craft answers in your own words. They are not scripts for a live task and not permission to share assessment content.

### Q1. What is remote AI-training work, in one minute?

**Intent:** See whether you confuse annotation with model training, or understand the supervision signal you actually produce.

**Strong sample answer:** I produce judgments that other pipelines use as supervision or evaluation data. Depending on the project, that is a preference between two outputs, a Likert score on a named axis, a rewritten answer, a code verdict, or a rubric. I am not running the optimizer. My job is to apply a written guideline consistently, including ties and refusals, and to justify the label from the spec and the evidence in the output. My edge is eleven years of Java, Spring, and AWS, plus AIML study, so I am most useful when the item is code, a service design, or an agent trace that has to be checked for correctness rather than fluency.

**Follow-up:** How is that different from software QA?

**Strong follow-up:** QA usually has an oracle: a ticket, a test, a log. Here the oracle is often a guideline plus domain judgment, and two fluent answers can both compile. I still look for a checkable failure, and I say when the guideline does not decide the case.

**Weak answer:** I train ChatGPT by chatting with it and telling it when it is wrong.

### Q2. How do Mercor, Alignerr, and vendors like DataAnnotation differ?

**Intent:** Check that you know public positioning without inventing internal process.

**Strong sample answer:** Mercor, publicly, matches experts to short-term AI-training work and other technical contracts, often after an interview or sample task. Alignerr is an AI-trainer platform associated with Outlier-style work; people often misspell it "aligner." The work there, and on vendors such as DataAnnotation, is typically ranking outputs, writing prompts, and domain labeling under guidelines. I treat brand names as routing, not as a different profession. Before I apply I recheck the current application flow on the company's site, because matching steps and task mix change.

**Follow-up:** Which one should a senior Java lead prefer?

**Strong follow-up:** Whichever project has a real code or systems queue and a guideline I can apply. I do not pick a brand for prestige. I pick work where a wrong label from me would be caught by tests, traces, or a rubric I understand.

**Weak answer:** They are all the same site with different logos, and I already know the exact application clicks.

### Q3. What does a good labeling justification look like?

**Intent:** Separate guideline citation from personal taste.

**Strong sample answer:** One short paragraph: the criterion, the evidence, the label. For example, "The guideline ranks correctness above brevity. Response B throws on an empty list, which the prompt's contract treats as a valid input returning zero. Response A returns zero and still handles the non-empty case. I prefer A." I do not write "B feels more professional" or "I like streams." If the schema has a harmlessness axis separate from helpfulness, I score that axis on its own instead of averaging them in my head.

**Follow-up:** What if you dislike both?

**Strong follow-up:** I still apply the schema. If both fail the same hard criterion and the project allows a tie or a dual-fail flag, I use it. If I must pick a lesser failure, I say which criterion breaks worse and why.

**Weak answer:** I explain my personal coding philosophy so the reviewer sees I am senior.

### Q4. How do you handle a case the guideline does not cover?

**Intent:** Escalation judgment versus silent guessing.

**Strong sample answer:** I check whether the case is an instance of a stated rule, including examples and counterexamples. If it is a true gap, I use the project's comment or escalation path and I do not invent a house rule. I record what I observed and which criterion almost applied. Guessing to protect throughput creates a private standard that will not match the next rater or the golden set.

**Follow-up:** What if there is no escalation button?

**Strong follow-up:** I follow the written fallback, often a specific label such as "insufficient information" or a tie, and I put the gap in the rationale. I do not email task content to a friend to crowdsource the label.

**Weak answer:** I go with my gut and move on so my hourly rate stays high.

### Q5. Why should a project trust your labels on code?

**Intent:** Evidence of method, not credentials theater.

**Strong sample answer:** Because I score against the prompt's contract, not against my favorite style. I check behavior on the stated inputs and the edge the contract implies: empty, null, duplicates, error paths. I look at tests if they are part of the answer: do they assert the failure mode or only the happy path? I flag security footguns the guideline cares about, such as injection, secrets in logs, or trust of user input in a query. I separate "works" from "I would refactor." My background is production Java services, Spring, and AWS, and I am in an AIML master's, so I can also say when an ML-flavored explanation overclaims.

**Follow-up:** Give a subtle bug you would catch.

**Strong follow-up:** A Java method that uses `==` on boxed integers inside the cache range and looks fine on small tests, or a stream that `findFirst` after a sort that is not stable in the way the comment claims. I cite the input that breaks it.

**Weak answer:** Trust me because I have eleven years and a master's, so my taste is the standard.

### Q6. How do you balance speed and quality?

**Intent:** See if you will game throughput.

**Strong sample answer:** I make the mandatory pass non-negotiable: criterion, edge named by the schema, evidence sentence. I cut optional prose, not the check. If a project publishes a target time, I use it as a budget for ordinary items and I let genuine ambiguity take longer, with a note. I would rather lose a few items per hour than fail golden questions. Audits sample rationales, so a fast empty justification is not actually fast once it is returned.

**Follow-up:** What do you do after you miss a golden item?

**Strong follow-up:** I reread the guideline section that item targeted. I write down the rule I missed in my own words, then I apply it to the next three similar items before I speed up again.

**Weak answer:** I skim and trust pattern matching. Most items are obvious.

### Q7. What rater biases do you watch in yourself?

**Intent:** Self-calibration, especially length, fluency, sycophancy, and position.

**Strong sample answer:** I notice four. Length: a longer answer feels thorough and may bury a wrong step. Fluency: clean prose hides a false API. Sycophancy: the answer that agrees with the user can violate the task or safety rule. Position: the first or last candidate gets an unearned lift, so I judge each against the rubric before I compare. On code I also watch familiarity bias: I over-score the idiom I use at work. The correction is a behavior check, not a vibe check.

**Follow-up:** How do you catch position bias on a three-way rank?

**Strong follow-up:** I score each output independently on the dimensions, then sort the scores. If the order matches reading order too often across a session, I rescore a sample from the bottom of the list first.

**Weak answer:** I do not have biases. I have been coding for a long time.

### Q8. How do you describe your AIML master's without overselling?

**Intent:** Honesty about research depth.

**Strong sample answer:** I say I am an M.Tech AIML student and a senior backend lead, not an RLHF researcher. I can explain preference pairs, why a reward model can pick up length bias, and how evaluation differs from training. I do not claim papers, production fine-tunes, or metrics I did not measure. If a project needs a published research record, I am the wrong match, and I say so. If it needs someone who can judge Java and reason about model failure modes, I am a fit.

**Follow-up:** Have you trained a reward model?

**Strong follow-up:** Not as a research or production owner. I understand what preference labels are for, and I can produce those labels carefully. I will not describe coursework as a shipped training stack.

**Weak answer:** My master's means I have done cutting-edge alignment research for industry models.

### Q9. What will you refuse to do on a project?

**Intent:** Integrity boundary: no cheating, no identity games, no leaked tasks.

**Strong sample answer:** I will not share live prompts, model outputs, or rubrics outside the project. I will not use disallowed tools, look up an answer key, or have someone else label under my account. I will not invent credentials, employers, or metrics in my profile. If a task asks me to produce something harmful that the guideline says to refuse, I refuse and label it as the schema requires. Getting paid does not outrank the spec or the law.

**Follow-up:** A peer offers a shared doc of "correct" assessment answers. What do you do?

**Strong follow-up:** I decline and I do not open it. An assessment is a work sample. Using a key is misrepresentation, and it also means I cannot do the live queue.

**Weak answer:** I would only use the doc if I was stuck, and I would reword it.

### Q10. How do you choose among projects once you are in?

**Intent:** Professional targeting, not status chasing.

**Strong sample answer:** I take queues where the guideline is specific and the domain is one I can defend: Java and Python correctness, Spring-style service behavior, AWS-shaped architecture judgments, defect triage, and agentic workflows I have actually worked on. I am cautious on medical, legal, or low-resource language tasks where I am not an expert. I reread the pay, time, and tool rules for that project rather than assuming the last project's rules still apply. Processes change, so I recheck the current flow when I move to a new platform or a new contract.

**Follow-up:** What do you do in the first hour of a new guideline?

**Strong follow-up:** I extract the dimensions, the hard fails, the tie rule, and two examples of close calls. I keep that sheet next to me. I do a few items slowly and compare my rationales to any sample the project gave before I increase speed.

**Weak answer:** I accept every queue that pays. Expertise is optional if the English is fine.
