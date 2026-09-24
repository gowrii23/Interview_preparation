# Code evaluation tasks: interview Q&A

### Q1. How do you start a code-evaluation item?

**Intent:** Contract before taste.

**Strong sample answer:** I rewrite the prompt as a contract: return value, error behavior, language limits, and whether tests or an explanation are required. I run the happy path mentally once, then the edges that contract names. Only after behavior is clear do I look at style, complexity claims, and comments. My Java background helps me see unboxing and collection traps. It does not let me replace the prompt with our internal style guide.

**Follow-up:** The prompt is ambiguous about null.

**Strong follow-up:** I do not invent a null policy. If both candidates assume different policies and the guide has an underspecified rule, I apply that rule or escalate. I say which behavior was not specified.

**Weak answer:** I start by rewriting it the way I would in a Spring service.

### Q2. What is wrong with rewarding "senior-looking" code?

**Intent:** Fluency bias on code.

**Strong sample answer:** Models imitate the surface of experience: streams, records, annotations, confident comments. Sample pattern: a method that throws on an empty list because it treats empty like null, written with a stream so it looks modern. The contract said empty returns 0. A rater who scores style first gives it a 4 and calls the throw a nit. Under a correctness-first rubric that break is a 1. I cite the input, the actual throw, and the required 0. The stream is irrelevant.

**Follow-up:** The ugly version also has a bug.

**Strong follow-up:** Then both fail correctness. I compare the remaining axes only if the schema says how to rank two failures. I still do not give the pretty one a pass.

**Weak answer:** Senior-looking code is usually correct because seniors write it.

### Q3. Walk a subtle Java bug you would catch.

**Intent:** Concrete domain skill. Use an original example, not a leaked task.

**Strong sample answer:** `countPositive` is specified to throw `NullPointerException` on a null list and return 0 on an empty list. One candidate does `if (values == null || values.isEmpty()) throw new NullPointerException`. Empty input takes the throw. A test with only a null list passes. I mark correctness as fail and I quote that condition. I might also note `filter(v -> v > 0)` unboxes a null element, but the deciding defect is the stated empty-list contract. A good candidate checks null, then counts with `value != null && value > 0`, and returns 0 when the loop does not run.

**Follow-up:** Would you fail the good one for not using `Objects.requireNonNull`?

**Strong follow-up:** No. The contract required the exception type and the empty behavior, not a particular helper. Style preference is not a defect.

**Weak answer:** I would fail anything that does not use streams and `Optional`.

### Q4. How do you score tests?

**Intent:** Tests as evidence.

**Strong sample answer:** A test counts if it asserts the contract and can fail when the contract breaks. I credit an empty-list assertion when that was the risky clause. I do not credit a test with no assertion, a test that expects the buggy return, or a test that mocks the method it claims to verify. If the prompt did not require tests, I leave them off the deciding axis. If code and tests agree on the wrong value, both are wrong. Consistency is not correctness.

**Follow-up:** The test fails to compile because of a missing import, and the logic is right.

**Strong follow-up:** I apply the guide's rule for non-compiling answers. Often that is an instruction-following or completeness miss, while the reasoning axis can still be partly met. I do not wave it through as "obviously fine."

**Weak answer:** Any test file is a plus. Coverage tools are the real judge.

### Q5. How do you treat a false complexity explanation?

**Intent:** Axis separation.

**Strong sample answer:** If the code is linear and the comment says quadratic, the explanation axis drops and correctness can still pass. If the user required a linear solution and the code sorts, correctness or constraint-following fails even if the comment brags about `O(n)`. I quote the false sentence. I do not compute a fake benchmark number in the rationale. Big-O claims need the structure in the code to support them, such as a `HashMap` lookup versus a scan per element.

**Follow-up:** The code is correct and the explanation is absent.

**Strong follow-up:** If explanation was optional, I do not invent a penalty. If it was required, I score the missing explanation at the anchor for "absent," not as a style nit.

**Weak answer:** Comments do not matter. Only code ships.

### Q6. When do you flag a security footgun?

**Intent:** Proportion, no exploit tutorial.

**Strong sample answer:** When untrusted input reaches a sink the task should care about. Examples I will name at the category level: SQL built by string concatenation, a shell command assembled from user text, a token written to an info log, certificate checks turned off, or a user path concatenated onto a base directory. I state source and sink. I do not include a working payload. If the snippet is a pure function on numbers, I do not add a security paragraph to sound thorough. A footgun the rubric calls a hard fail outranks clean layering.

**Follow-up:** The model mentions SQL injection in a comment but still concatenates.

**Strong follow-up:** The comment shows awareness and the code still fails the safety criterion. Awareness is not a fix. I might note the comment on an explanation axis as incomplete because it does not match the behavior.

**Weak answer:** I demonstrate the exploit in the rationale so the reviewer believes me.

### Q7. Python is weaker for you than Java. What do you still rate?

**Intent:** Honest limits.

**Strong sample answer:** I rate control flow, boundary conditions, tests, and claims I can verify, including mutable default arguments and confusing `is` with `==` when I am sure. If the bug depends on a version-specific library semantic I do not know, I say so and skip or escalate rather than fabricate a rule. I do not market myself as a language-agnostic staff reviewer. Java and backend service behavior are the core. Python is careful general evaluation plus the traps I have actually used.

**Follow-up:** The queue is mostly Python.

**Strong follow-up:** I take it only if the qualification shows I meet the bar. A senior title is not a substitute for a qualification set.

**Weak answer:** Syntax is syntax. I can review Rust or Python at the same level as Java.

### Q8. How do you score an agent trace that proposes a code fix?

**Intent:** Tie to defect-triage and agentic work without fake metrics.

**Strong sample answer:** I treat the trace like evidence. Did it inspect the relevant failure, or did it narrate a plausible bug? Does the proposed patch match the contract, including regression behavior? A confident summary attached to the wrong stack line is a failed triage even if the prose is clean. I have worked on that theme in enterprise Java systems. I describe the judgment method, not a made-up accuracy rate. If the trace calls tools, a hidden failed call is a defect when the rubric says tool truthfulness matters.

**Follow-up:** The patch is correct and the trace's story is wrong.

**Strong follow-up:** I split the axes: patch correctness can pass, trace faithfulness fails. I do not give one blended "pretty good."

**Weak answer:** If the final diff works, the trace can say whatever it wants.

### Q9. What does a 1 versus a 5 look like on code?

**Intent:** Anchors.

**Strong sample answer:** Under a correctness-first 1-to-5 scale I would define 1 as breaking a stated requirement, such as throwing on empty input when 0 is required, or introducing a forbidden sink. I would define 5 as meeting the stated behavior on the named edges, with explanations that match the code if explanations were required, and without an extra penalty for plain style. A 3 is not a consolation prize for fluent wrong code. A 3 is for a real partial case the anchors describe, such as the main path working and a required test missing, only if the rubric says that pattern is a 3. I keep the project's anchors if they differ from mine.

**Follow-up:** Would Sample B ever be a 3?

**Strong follow-up:** Not on a scale where a broken stated requirement is a 1. Fluency does not buy the middle.

**Weak answer:** 5 means I would merge it. 1 means I dislike the approach.

### Q10. Why should a project trust your code labels?

**Intent:** Method, not résumé.

**Strong sample answer:** I publish the contract, the input that decides the label, and the axis I refused to let style hijack. I separate tests that can fail from tests that echo the bug. I flag security by source and sink without turning the note into an exploit. I will say when a Python or domain detail is outside my certainty. That is eleven years of backend review habits pointed at model output, plus enough AIML context to know a fluent wrong label becomes bad supervision. Trust is you being able to predict my score from the rubric and the snippet alone.

**Follow-up:** Show the one-sentence version for Sample B.

**Strong follow-up:** "Fails correctness: empty list throws `NullPointerException`; contract requires 0. Style of the stream does not change the score."

**Weak answer:** Trust me because I have been a tech lead for years and I know good code when I see it.
