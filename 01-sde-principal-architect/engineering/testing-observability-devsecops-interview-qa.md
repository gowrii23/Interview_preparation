# Testing, observability, and DevSecOps — interview questions

## Q1. How do you test checkout without booting Spring for every case?

**What the interviewer is probing:** Layering, and a Mockito test that asserts money behavior.

**Sample answer:** The domain test is plain JUnit. `Order.addLine` and `Money.plus` run with `new`, no container, and they assert minor units, currency mismatch, and the rejection of a line after authorization. The application service test constructs `CheckoutService` with a fake `PaymentGateway` and a fake repository. Mockito stubs `authorize` to return a `Declined`, and I assert the order was not marked authorized and that no capture was called. I verify the interaction that has a side effect, not every query.

I do not `@SpringBootTest` the decline rule. I do use `@WebMvcTest` for validation and the problem JSON, and Testcontainers for the idempotency unique constraint, because a mock map will not prove two inserts fail. One full context test can boot the real checkout path with the gateway stubbed at the port. If the service needs twenty `@Mock` fields, I split the service before I write more tests. I keep `Thread.sleep` out. Time is injected as a clock so a reservation expiry test is deterministic. The principal point is the pyramid for this bug: domain for the rule, contract or web slice for the status code, database test for the constraint. Coverage percent is not the design.

**Follow-up:** What does `verifyNoMoreInteractions` hide when you add logging?

**Weak answer:** "I mock the EntityManager and assert that SQL contains the word UPDATE."

## Q2. A test passes alone and fails in the suite. How do you handle it?

**What the interviewer is probing:** Shared state and flakes you refuse to ignore.

**Sample answer:** I assume shared mutable state until proven otherwise: a static cache, a singleton dirty from another test, an embedded database that did not roll back, a port left in a stubbed state, or a clock. I run the test in isolation to confirm, then I look at order dependence. `@DirtiesContext` hiding the problem is a last resort; it slows the suite and leaves the leak. I fix the test to build its own fixture and to assert on ids it created, not on "the only row in the table."

A race in the test itself, `Thread.sleep` until a future finishes, gets a latch or a deterministic scheduler. I do not retry the test in CI to make the pipeline green. A flake on the tender path will flake in production for a different reason, or it will train the team to ignore red builds. If the failure is a Testcontainers port collision, I let the container pick the port. I quarantine only with an owner and a ticket, and I say that in the interview so I do not sound rigid about a broken third-party sandbox. The default is fix or delete. A green suite that sometimes lies is not a quality gate SonarQube can save.

**Follow-up:** How do you find which earlier test polluted the static map?

**Weak answer:** "Add a retry annotation and move on."

## Q3. What do you assert in a contract test between order and inventory?

**What the interviewer is probing:** Drift across services, Feign and OpenAPI.

**Sample answer:** I assert the wire contract, not the Java method names. A published OpenAPI or a recorded pact includes the reserve path, the idempotency header, the request fields, and the error body for not-enough-stock. The inventory provider verifies it can answer that contract. The order consumer fails CI when it requires a field the provider does not send. I include one unknown enum value so the consumer proves it will not throw a `500` on a forward-compatible response. I do not copy a shared DTO jar as the only contract; a binary jar couples releases and still lets the JSON annotations drift.

The test runs without both services being up if the contract artifact is versioned. An end-to-end environment test is extra and slower, useful for one happy path, useless as the only check because it is red for too many unrelated reasons. I keep fixtures free of real card data. When Apigee transforms a header, the contract I care about is the one the service actually sees, so I note the gateway mapping in the test or I test the route. The principal standard: a field rename in inventory fails the order build before a store sees it.

**Follow-up:** How do you version the contract artifact?

**Weak answer:** "We deploy both services together, so a contract test is unnecessary."

## Q4. How do you design a JMeter run you would trust before a peak?

**What the interviewer is probing:** Honesty about performance evidence.

**Sample answer:** The scenario matches a cashier: a basket with a realistic line count, a reserve of a hot SKU from many threads, and a payment stub with a latency we have actually measured. The database has a catalog and indexes, not ten rows. The driver is not the server under test. I record error rate, a high percentile, Hikari pending, GC pauses, and downstream time. The pass line is the SLO the business already set. I do not invent a transactions-per-second trophy, and I would not quote one in an interview as if it were a production measurement.

I ignore the first interval while the JIT warms, or I run long enough that warmup is a small fraction. I include a case where inventory is slow, because that is how peak fails. The result I want is a named bottleneck: row lock, pool, or allocation. A green JMeter with assertions only on HTTP 200 misses a double charge that returned 200 twice. I use unique idempotency keys per logical attempt and a deliberate replay case. I do not run this against production. I do not treat a laptop against shared dev as evidence. The artifact is the scenario file, the environment description, and the bottleneck, checked in or attached to the release.

**Follow-up:** What threshold would make you stop a rollout even if CPU looks idle?

**Weak answer:** "We ran JMeter and the average was low, so we are ready."

## Q5. A store reports a failed tender. What do you look up?

**What the interviewer is probing:** Correlation, safe logging, Kibana and APM.

**Sample answer:** I take the correlation id from the receipt or the client log, or I search Kibana by store id, a time window, and the last four of an order number — never by a full card number, which we must not have logged. The log line should already include the idempotency key, the order id, the gateway result code, and the dependency that failed. If the line says "error" with none of those, the gap is the defect I fix after the incident.

I open the same id in New Relic or the trace backend and see which hop used the time: order, inventory, Redis, or the gateway. A timeout with no gateway reference means we never got a response and the state may be unknown; I do not tell the store to tap again without checking whether the first attempt is `UNKNOWN` and in reconciliation. I check whether a deploy or a config change landed in that window. I write the timeline for the incident in those ids. The principal behavior is: one id across logs and traces, no PAN in the query box, and a distinction between decline, timeout, and bug. I also note if the alert fired before the store called. If it did not, the alert is wrong.

**Follow-up:** What do you add to the log if the trace shows a gap you cannot explain?

**Weak answer:** "I grep the server for Exception and read the stack trace emails."

## Q6. What do you alert on, and what do you refuse to page for?

**What the interviewer is probing:** Symptom-based alerts versus noise.

**Sample answer:** I page on symptoms a cashier feels or is about to feel: tender error rate above the budget, a jump in timeouts, Hikari pending above zero for a sustained window, the circuit breaker open on payment, disk or database sessions near the limit. I chart GC pauses and latency percentiles, and I page on pauses only when they breach the SLO, not on every young collection. I do not page on a single stack trace of a client `400`, or on a decline code that means insufficient funds. Those are business outcomes or client bugs, and paging them trains the on-call to mute the channel.

Each page has a runbook line: the dashboard, the likely dependency, and the mitigation (scale, roll back, fail closed). An alert without an action is a metric. I keep the threshold tied to a number we can defend, and I would rather have a quiet page for "unknown tender outcomes rising" than a noisy page for every exception class. Logs stay in Kibana for diagnosis. Metrics stay aggregated. I mention cardinality: a label per order id on a metric will blow up the series and the bill, so order id belongs on the log and the trace, not on every metric tag. That is the design I defend.

**Follow-up:** How do you alert on a dead reconciliation job that fails quietly?

**Weak answer:** "Email the team on every ERROR log."

## Q7. What does a useful SonarQube gate look like for this service?

**What the interviewer is probing:** Judgment about quality gates, not worship of a score.

**Sample answer:** I fail the build on new bugs and vulnerabilities, on security hotspots that are real (string-concatenated native SQL, hardcoded secrets), and on coverage of new code in the payment and inventory modules. I do not set a vanity 90 percent on the whole legacy tree, and I do not exclude the payment adapter because it is "hard to test." I do exclude generated code. Cognitive complexity on `CheckoutService` is a signal to split the class, not a reason to suppress the rule. A suppression needs a reason in the review.

I treat dependency CVE scans beside SonarQube. A critical issue in a library that parses requests blocks promotion. A finding in a test-only jar is triaged, not dramatized. The gate must be fast enough that people do not bypass it. If the scanner is red on the main branch for weeks, it is not a gate. I would rather have five rules that stop a bad merge than fifty rules everyone ignores. In the interview I give one example I would block (SQL built from a sku parameter) and one I would waive with a note (a false positive on a prepared statement). That balance is the principal signal.

**Follow-up:** How do you handle a false positive that the vendor will not fix this quarter?

**Weak answer:** "The project must stay at an A and 80 percent or we do not release."

## Q8. What secrets and runtime controls do you expect on the order pod?

**What the interviewer is probing:** Practical DevSecOps, not a slogan.

**Sample answer:** Secrets come from a secret store or the platform's secret object, injected as environment or files, not from git and not baked into the image. The image runs as a non-root user, and the filesystem is read-only where the JVM and the temp directory allow it. The service account can reach its database, Redis, inventory, pricing, and the payment host, and network policy denies the rest. Apigee at the edge does not replace service-to-service authentication.

I do not log PAN, track data, or the secret itself. A debug flag that dumps headers is a review defect if it can be turned on in prod without an audit. Dependencies are scanned in CI. I keep a feature flag for a new tender type so we can enable one store, and I test both flag states. The principal framing is blast radius: a stolen order-pod credential should not read the whole estate, and a bad deploy should be reversible. I also want the pipeline to promote the same image that was tested, not rebuild from a drifting branch. Those are concrete controls. I do not claim a framework name replaces them.

**Follow-up:** How do you rotate the database password without dropping in-flight tenders?

**Weak answer:** "Security is the Apigee team's job and the cloud is secure by default."

## Q9. Mockito returned a passing test and production double-charged. What was missing?

**What the interviewer is probing:** The limits of interaction tests.

**Sample answer:** The unit test stubbed the gateway and verified `authorize` was called once inside one JVM, one thread, one fake. Production has two pods, a client retry, and a gateway timeout. The mock never had a unique constraint, never lost a response, and never ran the method twice in parallel. The missing test is a repository test that inserts the same idempotency key twice and expects a constraint violation, plus a service test where the gateway throws a timeout and the stored state is `UNKNOWN`, not "call authorize again." I also want a test that a second call with the same key returns the stored receipt and does not call the gateway.

Mockito is still right for the decline mapping. It is the wrong tool for the race. I look at argument matchers too: `any()` paired with a raw value can make `when` not match, so the test passed on a default null and production sent a real command. I assert the return value the cashier would see, not only `verify`. After the incident I add the constraint test in Testcontainers and I do not lower the production risk by adding a third retry. The interview close: name the layer that would have failed, and do not blame "mocks" in general. The gap was the missing failure mode, not the existence of JUnit.

**Follow-up:** How do you test the reconciliation job's repair of UNKNOWN without calling the real gateway?

**Weak answer:** "We need to stop using Mockito because mocks are not real."

## Q10. How do you roll out a pricing change with a way back?

**What the interviewer is probing:** Release safety, flags, and observation.

**Sample answer:** I ship the code dark behind a flag defaulted off. Automated tests cover both flag states: old price path and new price path, including a mismatch between scanned price and price book. I enable the flag for one store, watch tender success, quote latency, and the specific error codes in Kibana and New Relic, and I compare that store with a neighbor on the old path. I do not watch only CPU. A wrong price looks like success in HTTP 200 and shows up as refunds and complaints, so I want a metric or a log of price-rule id on the quote, and a business check I agreed in advance.

Rollback is flipping the flag, which must be safe for orders already priced: an in-flight tender keeps the price it already persisted, and does not reprice mid-authorization. If the change is a schema migration, the migration is expandable first (add a column, deploy code, backfill, then remove) so a code rollback does not require a database rollback that drops data. SonarQube and the test suite gate the merge. The flag is not an excuse to skip the suite. The principal point is reversibility and a defined observation, not "we use CI/CD."

**Follow-up:** When is a flag the wrong tool and a branch deploy the right one?

**Weak answer:** "We deploy on Friday and watch the logs if someone complains."
