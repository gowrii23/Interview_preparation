# Principal architect craft interview Q&A

Fill every bracket with work you actually did. If you do not remember a number, describe the movement and leave the figure out. Do not borrow these samples as fake incidents.

## 1. How is a principal different from the tech lead you have been?

**Interviewer intent.** Scope, not seniority theater.

**Strong sample answer.** A tech lead makes a known change land with one team: design, review, production safety. A principal is accountable for a problem that crosses teams and for a rule that keeps the next five projects from re-deciding it. I still go deep on the design that is risky. I stop being the person who implements every critical path. My output is a decision, an ADR, a risk with an owner, and a tech lead who can run the next incident. If I am only a faster tech lead, I am not doing the job.

**Follow-up.** Give an example of a rule you set. Use [a real boundary you enforced: ownership of a flow, a timeout policy, a ban on a second system of record].

**Weak answer to avoid.** "I will be more hands-on and review every pull request in the program."

## 2. Tell us about a time you influenced a team you did not manage.

**Interviewer intent.** STAR with a real disagreement. Structure only; the facts must be yours.

**Strong sample answer.** Situation: [team and the conflicting design]. Task: [decision needed] and I had no reporting line. Action: I wrote two options with the failure mode and the operational cost, and I took them to [the forum you actually used]. I was explicit about what I was asking them to own on-call. Result: [what was decided and what shipped]. If they chose the option I liked less, I say so and say how I supported it afterward.

**Follow-up.** What if they had said no? I would have recorded the residual risk and the trigger that would reopen it, not relitigated it in a side channel.

**Weak answer to avoid.** A story where everyone immediately agreed, or a story that uses a metric you cannot defend.

## 3. What belongs in an ADR, and what does not?

**Interviewer intent.** You will actually write them.

**Strong sample answer.** Context, the decision, the options rejected, and the consequences, including who operates the result and which NFR drove it. I write one when reversal is expensive: store choice, sync versus async on payment, partition key, public API break. I do not write one for a library bump. I supersede rather than edit history. The test is whether a new engineer can see why we live with the downside. A slide that only shows the chosen boxes fails that test.

**Follow-up.** Show them a real one if they ask. Use [an ADR or design note you wrote], or describe it faithfully. Do not invent a polished ADR you never filed.

**Weak answer to avoid.** "We keep decisions in chat so we stay agile."

## 4. How do you handle ambiguity in a checkout problem?

**Interviewer intent.** You narrow it instead of hiding in abstractions.

**Strong sample answer.** I name the invariant, the customer-visible behavior when it fails, and the budget I am assuming if the business has not given one. I propose the assumption in writing. For a stock pool shared by store and digital, I will not start drawing Kubernetes. I will ask who owns allocatable quantity and what happens at hold expiry. I leave with a boundary we can build and a list of open questions with owners. Ambiguity that stays in my head is not leadership.

**Follow-up.** They ask you to estimate anyway. Use [how you actually estimate]. Talk in slices and risks, not a false date you cannot control.

**Weak answer to avoid.** "I wait until requirements are complete" or a fog of architecture terms with no boundary.

## 5. Build versus buy. How do you decide?

**Interviewer intent.** Differentiation and exit cost.

**Strong sample answer.** Buy payments, messaging, and the company-standard edge gateway. Build plan compatibility, cart policy, and reservation, because that is the retail workflow and a package will be bent until we own it anyway. A buy includes exit and what we do when their latency moves. A build includes the rota. Open source is not free if we must operate Cassandra repairs or a custom queue. I have rejected [a real build or buy you argued]. I state the residual risk we accepted.

**Follow-up.** A vendor already in the building is still a decision. Sunk cost is not a requirement. I look at the contract we have, not the one we wish we had.

**Weak answer to avoid.** "Never build" or "never buy" as an identity.

## 6. How do you mentor without taking the design back?

**Interviewer intent.** Multiplier, not hero.

**Strong sample answer.** I ask them to state the invariant, the failure, and the metric before I offer a pattern. I let them present the data model. I stay on the cross-team call so they can run the incident timeline. Situation: [person or team]. Task: raise [reviews / on-call / modeling] without becoming the owner of their service. Action: [how you coached]. Result: [what they led next]. If the result was mixed, say what you would change.

**Follow-up.** Someone is wrong in a review. Correct the design in the room, specifically, without taking the keyboard away for the rest of the project.

**Weak answer to avoid.** "I mentor by rewriting their design overnight."

## 7. Describe a risk you stopped, and one you accepted.

**Interviewer intent.** You can tell them apart.

**Strong sample answer.** I stopped [a ship-now risk: a lock across a remote call, a migration that rewrites a hot table, a missing idempotency key]. I accepted [a residual risk: replica lag on a browse screen, a regional failover that is hours, a TTL staleness on marketing copy] and named the owner and the trigger to reopen. I do not keep a register of forty items. I keep the ones that change a rollout decision. I do not describe a risk I only heard about.

**Follow-up.** How did you make it visible? [dashboard, load test, or design review you actually used].

**Weak answer to avoid.** "I do not accept risk" or a generic "we had production issues and I fixed them" with no action.

## 8. Non-functional requirements. Which ones do you force into the open?

**Interviewer intent.** Budgets, not a checklist.

**Strong sample answer.** Latency split between browse and checkout, because one includes a payment timeout and the other should be a cache hit. Availability, RPO, RTO, and whether read-your-writes is required on which screen. Peak shape: a device launch is not average traffic. Audit retention and what we are not allowed to log. Cost of the data path, including license and NAT. If the business cannot give a number, I propose one and label it an assumption. A design that cannot be tested against a budget is not finished.

**Follow-up.** They challenge a number. Show how you would revise the design, not how you would defend your pride.

**Weak answer to avoid.** "We need five nines" with no user journey and no dependency math.

## 9. How do you keep operability in the design?

**Interviewer intent.** 2 a.m. behavior.

**Strong sample answer.** Correlation id from the channel through Feign to the SQL. Timeouts that fail visibly. A dashboard that separates dependency time from service time, in the tool we actually have, New Relic or Kibana. Logs without secrets or card data. A health check that means we can sell, not merely that the process is up. A runbook with the one knob that is safe. I have [a real incident where missing one of these hurt, if you have one]. I do not claim a blameless-culture slogan as the design.

**Follow-up.** What did you change after the incident? [the alert, the timeout, the ownership]. Only what you changed.

**Weak answer to avoid.** "Operations is the SRE team's job after we hand off."

## 10. Technical strategy. What would you write down in the first months?

**Interviewer intent.** A few rules that change reviews. You may state intent as principles. Do not invent a past strategy you did not set.

**Strong sample answer.** I would publish a short set and socialize it in design review. One transactional owner for an order. Cache is not a ledger. New services default to the relational engine we can operate, and they do not add Cassandra without an access path. Synchronous calls in checkout have a timeout shorter than the caller's, and side effects carry an idempotency key. I would tie each rule to [a repeated argument or incident you have actually seen]. A rule with no review impact gets deleted. I would also say what evidence would change my mind.

**Follow-up.** A team wants an exception. I want it in the ADR, time-boxed, with the risk owner. Silent exceptions become the real architecture.

**Weak answer to avoid.** A platform mandate list (specific frameworks, specific vendors) that does not mention invariants, or a claim that you already rolled out a company-wide strategy you did not.
