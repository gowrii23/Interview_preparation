# Spring Cloud and microservices — interview questions

## Q1. How do you split order, inventory, and payment without a distributed transaction?

**What the interviewer is probing:** Consistency, compensation, and the cashier experience.

**Sample answer:** I do not ask XA to span three databases and a card network. I give each service its own data. Checkout is an orchestration I can draw. The order service writes a `PENDING` order and an outbox row in one local transaction. It calls inventory to reserve with an idempotency key. It calls payment to authorize with its own key. It then marks the order `AUTHORIZED` locally. If authorization is declined, it calls inventory to cancel the reservation. If the process dies after authorize and before the local mark, reconciliation reads the gateway and repairs the order. The reservation has an expiry so a crashed caller does not hold the last unit until a human notices.

The cashier sees a spinner only inside a budget I set, then a clear result: authorized, declined, or "unknown, do not retry blindly." Loyalty is not in this sequence; it consumes `OrderPlaced` from the outbox and is allowed to lag. I reject a shared order-inventory schema "for the join." The join is a snapshot or a query API, and a report can be eventually consistent. The principal point is the compensation and the unknown state, not the number of repositories. Feign is just the client. The design is the keys, the expiry, and the outbox.

**Follow-up:** What do you do if cancel-reservation fails after a decline?

**Weak answer:** "I use a distributed transaction so everything commits together."

## Q2. When do you retry a Feign call?

**What the interviewer is probing:** Idempotency plus timeouts, not Resilience4j trivia.

**Sample answer:** I retry only when the operation is safe to repeat and I still have budget. `GET` availability can retry inside the cashier's timeout, with a short connect timeout and a bounded read timeout, and with a limit of one retry so I do not multiply load on a service that is already sick. `POST` reserve can retry only if the same idempotency key is sent and the inventory service stores the result. A capture or a "charge" without that key must not retry automatically. A timeout is an unknown outcome, not a decline. Retrying an unknown capture is how we double-charge.

The circuit breaker opens when the dependency is failing so we fail fast and stop occupying Tomcat threads and Hikari connections. It is not a retry. The bulkhead caps concurrent calls to inventory so a slow database there cannot consume every thread in the order pod. A fallback may return a cached price only if the business accepts stale prices. A fallback that returns success or a zero price is a defect. I set the budgets so that retries fit inside the caller's deadline. Three retries of a one-second call behind a one-second API gateway is a design error I would reject in review. I also propagate the correlation id so Kibana shows one gesture across the hops.

**Follow-up:** What metric tells you the breaker is flapping?

**Weak answer:** "I retry every Feign call three times with a five-second timeout."

## Q3. Explain the outbox. What failure does it remove?

**What the interviewer is probing:** Dual-write, and consumer idempotency.

**Sample answer:** Dual-write is: commit the order, then publish a message, and crash in between. Inventory or loyalty never hears about the order, or a naive retry publishes twice. The outbox puts the message row in the same database transaction as the order change. Either both commit or neither does. A publisher reads unpublished rows and sends them. If it crashes after send and before marking the row published, it sends again. That is at-least-once, so every consumer must be idempotent. I use an inbox table of processed message ids or a natural key, the reservation id, and a unique constraint.

I do not publish inside the transaction by calling the broker before commit. A rollback would have already emitted the event. I do not use the outbox for the synchronous reserve on the cashier path if the cashier is waiting; that call is still a request with its own idempotency key. The outbox is for work that can trail, and for integration that must not be lost. Polling interval and batch size are operational knobs: too slow and loyalty lags, too aggressive and we hammer the order database. I mention a retention cleanup so the outbox does not become an unbounded table. The principal sentence: one transaction locally, idempotent consumers, no distributed transaction required.

**Follow-up:** How do you order events for a single order without a global sequence?

**Weak answer:** "I publish after save and catch the exception if the broker is down."

## Q4. Where is Redis safe in this architecture?

**What the interviewer is probing:** Cache versus source of truth.

**Sample answer:** Redis is safe as a cache-aside price book: the key is store plus sku plus a price version, the value is the minor units, the TTL is a business choice, and a miss loads from Oracle or Postgres. I can tolerate a stale price only to the extent the business signed up for, and I do not hide that staleness. Redis is also a reasonable rate limiter or a short-lived lock for a single scheduler, if the lock has an expiry and the work is safe when the lock expires early and two holders run. I treat that as a mutual-exclusion hint, not as a transaction.

Redis is not the ledger for on-hand. A lost key, an eviction, or a failover with a weak persistence setting must not create inventory. The conditional SQL update remains the authority. I set a command timeout so a hung Redis does not consume the cashier budget; the miss path is defined, usually "read the database" for price and "fail the operation" if I was wrongly using Redis as a lock on stock. I watch hit ratio, memory, and eviction count. A stampede on a hot key is mitigated with single-flight or a stale value while one pod reloads. I say this split in one breath in an interview: price may be cached, stock is updated in the database.

**Follow-up:** What Redis topology or persistence setting would change your answer about locks?

**Weak answer:** "We put all session and inventory data in Redis for speed."

## Q5. A reserve call is slow and order pods run out of threads. What do you change?

**What the interviewer is probing:** Bulkheads, timeouts, and evidence.

**Sample answer:** I confirm with a trace and a dump: order threads blocked in the Feign client, inventory's database slow or locked, Hikari on the inventory side pending. The order service cannot fix a bad inventory query by adding pods alone; more pods multiply the in-flight calls and can make inventory worse. I set a read timeout inside the cashier budget so threads return. I put a bulkhead on the inventory client so only N calls run and the rest fail fast with a clear "inventory unavailable" instead of exhausting Tomcat. I open the circuit if errors and timeouts cross the threshold, and I do not retry while the breaker is open.

On the inventory side I look for a long transaction, a missing index, or a hot-row lock, not for a larger pool first. A larger Hikari pool on inventory without a faster query just adds sessions. I add the correlation id so one slow reserve is visible from the device log through both services in Kibana and in New Relic. The design change I defend long-term is a shorter lock and an idempotent reserve, so a client timeout does not leave an ambiguous hold without an expiry. I do not solve thread exhaustion by switching the stack to reactive in the middle of an incident.

**Follow-up:** How do you decide the bulkhead limit from pool sizes?

**Weak answer:** "Increase Tomcat max threads and the Feign timeout."

## Q6. Compare choreography and orchestration for checkout.

**What the interviewer is probing:** Operability of a saga.

**Sample answer:** Choreography means each service emits events and the next service reacts: order placed, inventory reserved, payment authorized, each listener continuing the flow. No central brain is nice until a store asks why a tender hung, and I have to read four codebases and the bus to reconstruct the state. Orchestration means one checkout workflow calls reserve, then authorize, then confirm, and records the step. I can see the state in one table and compensate from one place. The tradeoff is a workflow service that must not become a god that owns everyone's tables. It owns the process, not the data.

I orchestrate cashier checkout because the steps are few, the user is waiting, and compensations are specific. I choreograph loyalty, analytics, and search, because there is no user waiting and new consumers should subscribe without editing the order service. Either style needs idempotent steps and compensations. I do not pick a workflow engine product in the abstract; I pick a place where the state machine is explicit. A boolean `paid` on the order with implicit transitions is how we get a capture without a reserve. The principal answer names the state machine and who is allowed to transition it.

**Follow-up:** How do you version the workflow when a new step is inserted for a wallet tender?

**Weak answer:** "Microservices must be event-driven, so I remove all REST calls."

## Q7. What do you put in Spring Cloud Config versus the deployment?

**What the interviewer is probing:** Configuration hygiene and secrets.

**Sample answer:** Endpoints, timeouts, feature flags, and pool sizes can live in externalized config so I can change a timeout without a code edit. I still review those values. A timeout change is a behavior change. Secrets, database passwords, and API keys do not sit in the git repo that backs the config server; they come from a secret store injected into the environment. A profile selects local versus prod. I do not put a `mockPayments=true` flag in a prod config file "just in case."

`@RefreshScope` can pick up a new URL, and it does not safely rewrite in-flight tenders. I roll that kind of change. Kubernetes can supply the same configuration with a ConfigMap and a Secret; a separate config server is justified when many services share it and we already operate it, not by default because the dependency exists. Discovery follows the same rule: Kubernetes service DNS is enough for Feign in-cluster, and Eureka beside it is another system to keep healthy. I say what I would not centralize: the idempotency rules and the code that decides a decline. Those are not configuration. The interview line is that config is production code with a different deploy path, so it gets an owner and an audit.

**Follow-up:** How do you audit who changed a payment timeout?

**Weak answer:** "All configuration belongs in application.yml so it is visible."

## Q8. How do you propagate identity and correlation without propagating a transaction?

**What the interviewer is probing:** Context that should cross a process, and context that must not.

**Sample answer:** A correlation id is generated at the edge, Apigee or the first filter, and passed as a header on every Feign call. Logs in each service include it, so Kibana can show one tender. Traces use the same id if the tracer is integrated. That header is not a credential. Identity is a token whose signature the callee validates, or a mTLS identity between services. I do not forward an internal "user id" header that any pod can spoof unless the network policy and mesh make spoofing impossible, and I still prefer a token.

I never forward a database transaction, a JDBC connection, or a thread-local persistence context. Those are local resources. A transaction id for audit is just a string. The callee starts its own short transaction. I also do not rely on a thread local surviving an async handoff unless I copy the correlation id onto the executor. `CompletableFuture` on another pool loses the MDC if I forget that copy, and the log line becomes unsearchable. In the interview I list three things that cross the wire (correlation id, auth token, idempotency key) and three that do not (transaction, entity manager, Hikari connection). That list is the design.

**Follow-up:** What do you do with a baggage header that contains a store id the token does not allow?

**Weak answer:** "I put the EntityManager in a request header so the next service can join."

## Q9. Inventory must stay up when pricing is down. How do you isolate them?

**What the interviewer is probing:** Failure domains and deployment.

**Sample answer:** Separate deployables, separate databases, separate connection pools, and a bulkhead on the client. Pricing down should fail a quote that requires a price, and should not stop a reservation that already has a price captured on the order. If every quote calls pricing synchronously with no timeout, pricing's outage becomes an order-service outage, then an inventory outage, because threads pile up and someone "restarts everything." Timeouts and bulkheads are the isolation at runtime. Separate schemas are the isolation at data level: a lock storm on promotions must not lock the inventory table because they were joined in one database for convenience.

I still allow a degraded mode the business defines: sell from the price printed on the order if it was confirmed, refuse to reprice, and do not invent a price. I test this by killing the pricing stub in a pre-prod scenario, not by assuming the breaker works because the annotation is present. Kubernetes resource limits stop a noisy neighbor on the node, and they do not replace the timeout. The principal answer is the blast radius: name what still works when pricing is killed, and the test that proves it.

**Follow-up:** What shared dependency could still take both services down?

**Weak answer:** "They are different microservices, so they are isolated by definition."

## Q10. How would you review a new "inventory lookup" Feign call inside checkout?

**What the interviewer is probing:** Chatty design and review standards.

**Sample answer:** I ask whether the call sits inside a database transaction or a row lock. If it does, it moves out. I ask whether it is once per order or once per line. A loop of Feign calls for twenty SKUs is a latency and a failure multiplier; I want one batch availability call or data already on the order. I ask for the timeout, the idempotency key if it writes, and the behavior when the call times out: fail the tender, or proceed with a fact already known. I ask what the client does with a `404` versus a `503`, because those are different cashier messages.

I ask whether the DTO is a stable contract with a test, or a shared database entity serialized by accident. I ask how the correlation id is passed. I reject a fallback that returns "in stock." I check the thread and pool budget: this call counts against the same bulkhead as reserve. If the lookup exists only to render a screen, it does not belong on the authorize path. The review comment is those questions, written against the gesture of paying, not a style note about the interface name. If the author cannot state the timeout and the failure mode, the call is not ready.

**Follow-up:** When would you replace this call with a replicated read model?

**Weak answer:** "Approved, Feign is the standard so the design is fine."
