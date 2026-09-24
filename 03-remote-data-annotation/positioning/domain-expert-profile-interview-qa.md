# Domain-expert profile: interview Q&A

Say only what you can unpack for several minutes. Do not add contact details, fake metrics, or assessment content.

### Q1. Give the ninety-second profile.

**Intent:** Staffable and bounded.

**Strong sample answer:** I am a senior tech lead with about eleven years in enterprise backend systems, mainly Java, Spring, and AWS. I review and build services where the hard part is the contract: error behavior, retries, and what gets logged. I am also an M.Tech AIML student. That study is why I can talk about preference labels, rubrics, and rater bias. It is not a claim that I have trained production reward models. The themes I can discuss are defect triage, agentic workflows, and enterprise Java. I want evaluation work or a short technical contract where model code is judged for correctness, tests, and security footguns. I will not quote metrics I did not measure, and I am the wrong person for clinical or legal expert labels.

**Follow-up:** What should we put you on next week?

**Strong follow-up:** A code-evaluation or rubric-writing queue in backend systems, or a Mercor-style contract in Java services. Not a research-scientist seat.

**Weak answer:** I am a full-stack AI leader passionate about alignment and cloud at global scale.

### Q2. Why should we trust your labels?

**Intent:** Method over résumé.

**Strong sample answer:** Because you can predict them from the guideline. I restate the prompt as a contract, check the edges that contract names, and write the criterion plus the evidence. A fluent Java method that throws on an empty list when the contract returns 0 is a correctness fail, even if it looks senior. I keep harmlessness, helpfulness, correctness, and style on separate axes when the schema does. I tie or escalate only when the written rule says so. I watch length, fluency, sycophancy, position, and my own familiarity with Spring idioms. Trust is reproducibility, not years of service.

**Follow-up:** What if your label disagrees with a consensus of non-engineers?

**Strong follow-up:** If the consensus matches the guideline, I update my extract. Seniority does not outrank the spec. If the consensus contradicts the written criterion, I escalate with the quote and the input, and I do not run a shadow standard.

**Weak answer:** I have eleven years of experience, so my standard is higher than the rubric.

### Q3. Tell us about the defect-triage theme without numbers.

**Intent:** Honest project talk.

**Strong sample answer:** The problem was that defect reports arrived with incomplete evidence, and a person or a tool-using workflow had to decide what to inspect next. I worked on the enterprise side of that problem: which context was allowed, how duplicates showed up, and how a human could override a bad suggestion. A fluent summary pointing at the wrong log line is a failure even when the sentences are clean. I will not give you a precision percentage or a time-saved figure, because I am not going to invent one in an interview. I can walk through one real ambiguity and how we chose to surface it instead of hiding it.

**Follow-up:** How would you label a model trace of that workflow?

**Strong follow-up:** Hard fail if it cites evidence it did not retrieve. Then score whether the recommendation follows from the evidence, separately from whether the prose is polished.

**Weak answer:** We built an agent that automated triage and improved accuracy dramatically.

### Q4. What have you actually done with agentic workflows?

**Intent:** Scope control.

**Strong sample answer:** I have worked with workflows that use tools and need to show their work: fetch context, propose an action, and stop when the evidence is weak. My contribution is the engineering around those boundaries in a Java enterprise setting, not a claim that I trained the underlying model or replaced a team. I care about hidden tool failures, retries that are not idempotent, and rationales that sound sure. If your project needs a researcher who has published on agents, I am not that person. If it needs someone to score traces against a rubric, I am.

**Follow-up:** Do you call yourself an AI engineer?

**Strong follow-up:** Only with a qualifier. I am a backend lead who is studying AIML and who has built around agent-style workflows. The unqualified title oversells the model-training side.

**Weak answer:** I design autonomous agents end to end for any business problem.

### Q5. How do you describe the AIML master's if they push?

**Intent:** No research inflation.

**Strong sample answer:** I say it is a master's program I am doing, and I name the status accurately, in progress or complete. I can explain how human preference labels are used, why Likert anchors matter, and how length bias contaminates feedback. Coursework stays coursework. I do not translate assignments into "industry RLHF." If you need publications, ask me and I will tell you I do not have that research record. The combination I do offer is production review skill plus enough theory to know what a bad label does downstream.

**Follow-up:** Have you fine-tuned a large model?

**Strong follow-up:** Not as a production owner. If I did a small course exercise, I will describe it as that, including what I did not control. I will not blur the two.

**Weak answer:** The degree means I have done cutting-edge alignment work comparable to lab researchers.

### Q6. Mercor and Alignerr both appear in your plan. Why both?

**Intent:** Public facts, spelling, no fake process.

**Strong sample answer:** They buy related skills and package them differently. Mercor, on the public description, matches experts to short-term AI-training projects and technical contracts, often with an interview or sample task. Alignerr is an AI-trainer platform associated with Outlier-style work: ranking outputs, writing prompts, and labeling under guidelines. People often misspell it "aligner." Vendors like DataAnnotation post similar evaluation work under yet another brand. I keep one honest profile and I recheck the current application flow on each official site because the steps change. I do not memorize a click-path from a forum.

**Follow-up:** Which do you prefer?

**Strong follow-up:** The engagement whose guideline matches code and systems judgment, at hours I can actually work. Brand preference is not a strategy.

**Weak answer:** They are the same company. I already know someone who can skip the test.

### Q7. A sample task asks you to review code. How do you show seniority?

**Intent:** Craft as the signal.

**Strong sample answer:** I quote the contract, give the input that passes or breaks, and separate style from behavior. I mention tests only if they can catch the bug. I flag a security sink without pasting an exploit. I keep the note short enough to audit. Seniority shows up as the edge I noticed, such as empty versus null, or `Integer` equality, not as a speech about architecture. If I am unsure about a library detail, I write the uncertainty instead of bluffing. That is the same behavior I want you to trust on the live queue.

**Follow-up:** The sample forbids external tools.

**Strong follow-up:** I do not use them. A sample is a work trial. Bypassing the rule is a failed trial even if the label is lucky.

**Weak answer:** I rewrite their snippet into my preferred framework so they see how I think.

### Q8. What do you leave off the profile on purpose?

**Intent:** Editing judgment.

**Strong sample answer:** Contact details do not belong in a public prep note, and clutter metrics do not belong in the blurb. I leave off tools I have only touched, clouds I have not operated, and any percentage I cannot derive on a whiteboard. I leave off customer names that are confidential. I leave off slogans. I would rather be quizzed on Java concurrency, Spring transaction boundaries I have actually debugged, and how I score a preference pair. A long skills list creates questions I will fail. A short list creates questions I can answer.

**Follow-up:** Is humility a tactic?

**Strong follow-up:** No. Limits are facts. Hiding a real strength is also inaccurate. I state the strength and the boundary in the same paragraph.

**Weak answer:** I leave off nothing. More keywords mean more matches.

### Q9. Tell a story about enterprise Java that would help a rater queue.

**Intent:** Transfer from work to labels, still qualitative.

**Strong sample answer:** I use a review story: a change looked clean and still double-applied a write when a retry met a non-idempotent endpoint. The lesson I bring to labeling is that a confident explanation of "retry safety" is false if the operation is not idempotent. I would score the explanation axis down and, if the code retries blindly, the correctness or safety axis down, depending on the rubric. I will describe the mechanism without naming the client or inventing an outage cost. The point for you is that I already separate a polished design story from behavior.

**Follow-up:** How is that different from a style comment?

**Strong follow-up:** A style comment is "I prefer constructor injection." A behavior comment is "this retry can duplicate the charge because the handler is not idempotent." Only the second one decides a correctness label.

**Weak answer:** I led many mission-critical services and mentored large teams across the board.

### Q10. What will you do if the role asks you to overclaim?

**Intent:** Integrity close.

**Strong sample answer:** I will correct the claim in the room. If a client introduction calls me an RLHF researcher, I will say I am a senior backend lead and AIML student who can label and write technical rubrics. If a project asks me to present someone else's credentials, share a live assessment, or pretend a practice task is my independent work, I will refuse. Those shortcuts make the labels untrustworthy, which is the only product I am selling. I would rather lose the match than win it with a sentence I cannot defend.

**Follow-up:** They say everyone inflates a little.

**Strong follow-up:** Then I am a bad cultural fit for that room, and I should not be on the project. Inflation is how the wrong expert gets the queue.

**Weak answer:** I can adjust my story to whatever the job description needs. We can fix the résumé later.
