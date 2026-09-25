# API design and resilience interview Q&A

## 1. How mature should these REST APIs be?

**Interviewer intent.** Usefulness over a maturity-model score.

**Strong sample answer.** Resources with explicit state: cart, checkout, order, reservation, return. A command like submit is a `POST` on a sub-resource. Clients in this estate use a versioned OpenAPI contract, not runtime hypermedia discovery, so I do not spend the design on link relations nobody will follow. I do spend it on idempotency, paging, and an error body that says whether a retry is safe. HTTP status matches the outcome. A declined card is not a 200 with a flag buried in text.

**Follow-up.** Richardson level three? I can describe it. I would not block a partner on hypermedia if the contract and the error model are solid.

**Weak answer to avoid.** "REST means CRUD on tables exposed one-to-one."

## 2. Design idempotency for checkout.

**Interviewer intent.** Key ownership and stored responses.

**Strong sample answer.** The channel creates the key once per purchase attempt and sends it on every retry. The server stores key, request hash, and response for a retention window that covers client retries and payment reconcile. Same key, same body, same response. Same key, different body, 409. The payment provider gets a stable key derived from the order id. A retry filter that generates a new key per attempt is worse than no retry. Get-by-id is naturally idempotent and still not a substitute for the key on create, because the client may not have received the id.

**Follow-up.** Two tabs. Two keys, two attempts. The business rule may then be a conflict on the cart version. The idempotency key does not merge humans.

**Weak answer to avoid.** "We dedupe on customer id for five minutes."

## 3. Offset or cursor pagination?

**Interviewer intent.** Stability under inserts.

**Strong sample answer.** Cursor for orders and audit: a stable sort such as created time plus id, a page cap, and a next cursor. Offset skips and repeats when rows arrive, and a large offset is a sequential tax on the primary. Jump-to-page for a catalog is a search index problem. I do not offer an unbounded export on the checkout API. Exports land in object storage from a job.

**Follow-up.** The client needs a total count. I give an estimate or a separate count API that can be stale. I do not `COUNT(*)` the orders table on every page request.

**Weak answer to avoid.** "We return all the customer's orders in one payload; pagination is premature."

## 4. What is in your error model?

**Interviewer intent.** Retryability and leakage.

**Strong sample answer.** A stable code, a message that is safe to log, a correlation id, and a retryable flag. 400 validation, 401 and 403 for auth, 404 missing, 409 conflict or idempotency mismatch, 422 for a business decline such as out of stock or card declined, 429 with `Retry-After` when we can say, 503 or 504 when a dependency failed and a retry might help. Not retryable: validation, auth, conflict, decline. Retryable only with the same idempotency key: timeout and 503 on submit. I do not put SQL or a stack in the Apigee response. The detail stays in the service log next to the correlation id.

**Follow-up.** They want one code for all failures so the app can show a generic toast. I keep the machine-readable code anyway. The toast can be generic. The client logic cannot.

**Weak answer to avoid.** "HTTP 200 and a status string, so the mobile parser is simpler."

## 5. How do you set timeouts and retries?

**Interviewer intent.** Deadlines shrink, and retries multiply load.

**Strong sample answer.** Every call has a connect and a read timeout. The downstream budget is shorter than the caller's so a stuck inventory call fails before the channel gives up and retries the whole checkout. I retry only idempotent operations, or the same idempotency key, with a small cap and jittered backoff. I do not retry 409 or a card decline. A payment timeout is "unknown": inquire, do not fire a second authorize. Retries without jitter turn a blip into a stampede. I include the database in the timeout story so a stuck query does not pin the pool.

**Follow-up.** How many attempts? Few. I would rather fail and let the client retry with the key than hide three hidden attempts inside Feign.

**Weak answer to avoid.** "We retry five times with no delay on every exception."

## 6. Circuit breaker and bulkhead. What does each prevent?

**Interviewer intent.** Different failures. You should not conflate them.

**Strong sample answer.** A breaker stops calling a dependency that is already failing, so threads do not sit until timeout. It opens on a rate of failures, fails fast or serves a degraded catalog, then probes half-open. It is per dependency. It does not make a payment timeout safe; it only stops new unknowns from piling up. A bulkhead is a separate pool. Payment latency must not consume the connections catalog needs. In-process, that is separate executors and client pools. Across the platform, it is a separate deployment so a bad recommendations release cannot scale into the order database. One global thread pool is neither pattern.

**Follow-up.** Breaker open on inventory during checkout. We fail the reserve quickly and the customer can retry. We do not skip reservation.

**Weak answer to avoid.** "We wrap the whole service in one breaker."

## 7. What Feign defaults would you change?

**Interviewer intent.** You have been hurt by the abstraction.

**Strong sample answer.** I set timeouts explicitly. I turn the client's own retry off and put policy where idempotency is known. I define which statuses throw. I propagate a correlation id and a deadline, and I do not forward the caller's credential to a third party. I log status and URL, not the body if it has personal data. A 503 that is HTML must not surface as a JSON decode bug in our client; decode failure on an error status is a dependency failure. I also do not treat a method call as proof the transaction is local. It is HTTP, and it can time out after the other side committed.

**Follow-up.** Hystrix? I would not introduce it. I use the resilience library the platform runs now, and I can still explain the states.

**Weak answer to avoid.** "Feign retries GET and POST the same way by default and that is what we want."

## 8. What belongs in Apigee, and what must not?

**Interviewer intent.** Edge versus domain.

**Strong sample answer.** TLS, client identity, quotas, spike arrest, and a stable external contract belong at the gateway. Cart rules, pricing, and reservation do not. A quota is not a substitute for a hot-SKU limit inside the service. I align gateway timeouts with the service so the gateway does not wait longer than the client and shorter than nothing useful. External errors are mapped so partners do not see internal class names. Versioning for partners lives here. Internal services can move faster underneath as long as the published contract holds.

**Follow-up.** A product owner wants a promotion rule in the gateway so it is "configuration." I keep it in the pricing service where it is testable and traceable. The gateway can route.

**Weak answer to avoid.** "The gateway is a pass-through, so we do not set timeouts there."

## 9. How do you version without stranding store clients?

**Interviewer intent.** Compatibility window. Mobile and associate builds linger.

**Strong sample answer.** Compatible changes are the default: optional fields, new endpoints, enums the client treats as unknown. I do not reuse a field or change a JSON type. A break is a new version with a window I name, and both versions run. The server tolerates old and new schema during deploy because two pod versions overlap. That is expand, then contract, for the database as well. Event consumers follow the same rule: add fields, do not rename, park unknown event types where a human can see them.

**Follow-up.** How long do you keep v1? Until the clients you can observe have moved, or until the business accepts cutting the remainder. I do not pick "two weeks" without that fact.

**Weak answer to avoid.** "We version in the body and change required fields in place."

## 10. What do you want on a dashboard the night of a device launch?

**Interviewer intent.** Signals that drive action.

**Strong sample answer.** Per dependency: rate, latency, timeouts, retries, breaker state, pool saturation. Separately, checkout outcomes: reserved, pending payment, paid, declined, unknown. Idempotency conflicts. Outbox age. Database connections and lock waits. Redis evictions and latency on the hot key if we can see it. An error ratio alone is not enough; I need to know if we are timing out because the pool is full. Correlation id must pivot from the gateway span to the SQL. I would rather five boards we trust than a catalog of charts. I do not claim a specific Verizon launch figure I cannot remember.

**Follow-up.** The first graph goes red. I say what I would check in order: pool saturation, downstream latency, then a recent release. I do not start by restarting everything.

**Weak answer to avoid.** "CPU on the node is the primary signal, and we page on every 500."
