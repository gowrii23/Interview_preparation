# Omnichannel POS system design interview Q&A

Use variables (`R_browse`, `R_checkout`, `T_pay`, `T_hold`). Do not invent peak transactions per second.

## 1. Sketch the end-to-end flow. What do you clarify before drawing?

**Interviewer intent.** You drive the problem. You do not dump a reference architecture.

**Strong sample answer.** I ask which channels share stock, whether pickup and ship-from-store are in scope, and what the payment provider's timeout and idempotency look like. I assume browse rate is far above checkout rate, a hero SKU can dominate checkout for a window, and the primary is multi-AZ. Then the flow: catalog read through a cache, cart owned by one service without decrementing stock, checkout reprices on the primary, reserves with a hold, creates an order with an idempotency key, calls payment without holding a row lock, confirms or releases, and publishes from an outbox to fulfillment. Returns are a new aggregate. I draw ownership, not every framework.

**Follow-up.** They remove ship-from-store. I drop allocation across stores and keep the same order state machine. I do not redraw the whole company.

**Weak answer to avoid.** Starting with "we use microservices and Kafka" before the stock invariant.

## 2. Who owns inventory, and how do you avoid oversell?

**Interviewer intent.** A single writer and a conditional update.

**Strong sample answer.** One inventory owner holds allocatable quantity. Reservation is a conditional write: insert a hold or decrement only if remaining quantity covers the request, with a unique reservation id. The hold expires at `T_hold` unless the order is paid. Checkout does not decrement in Redis. If order and inventory are different databases, a worker releases holds whose orders never reached paid. That worker is part of the design. A unique constraint on the client idempotency key stops a double submit from reserving twice. Popular SKUs serialize on one row; I either accept that and shed load, or I record holds as rows and define the sum carefully so I do not create a slower scan.

**Follow-up.** They ask for a distributed lock. I explain why a lease can have two holders and why the database condition is the authority.

**Weak answer to avoid.** "We check stock in the UI and then insert the order."

## 3. Where is price decided?

**Interviewer intent.** Stale cache must not charge the customer.

**Strong sample answer.** Merchandising owns the price of record in the relational database. The device page may show a cached price keyed by publish version. Checkout loads the price from the primary, applies the plan rules for that moment, and stores the priced lines on the order. A change during the session is a business rule: honor the cart for a window, or reprice and show the customer. I pick one and state it. I do not let the channel send a price the server trusts.

**Follow-up.** Promotion stack. The rules live in a service we can test, not in a trigger. The result is snapshotted on the order so a later rule edit does not rewrite history.

**Weak answer to avoid.** "The client displays the price and the server charges whatever was displayed, from the cache."

## 4. How do you make submit idempotent, including payment?

**Interviewer intent.** Keys, stored responses, unknown outcomes.

**Strong sample answer.** The client sends one idempotency key for the purchase attempt and reuses it on retry. The order service stores the key, a hash of the body, and the response. Same key and same body returns the same order. Same key and a different body is a conflict. The provider call uses a key derived from the order id so our retry does not authorize twice. A timeout is an unknown, not a failure: we inquire before we charge again. Webhooks carry an event id and the handler is idempotent. A Feign retry that mints a new UUID per attempt is a bug.

**Follow-up.** How long do you keep the key? At least the client retry window and the reconciliation window. I would set it from those, not from a guess about "forever."

**Weak answer to avoid.** "POST is not idempotent, so we tell the client not to retry."

## 5. How do you reason about capacity?

**Interviewer intent.** Ratios and bottlenecks, not a memorized QPS.

**Strong sample answer.** Checkout writes are proportional to `R_checkout`, times a small number of statements: order, lines, reservation, outbox. Browse database reads are `R_browse` times the cache miss ratio. I size the primary for the miss storm and for the hero SKU's share of checkout, not for a perfect cache. Connections are `pods * pool_size` and must sit under the database limit. Redis memory is hot keys times entry size, and one key can still pin a shard. The payment timeout `T_pay` dominates checkout latency, so internal work has to be much smaller or the customer times out first. I would load-test at a stated miss ratio. I would not quote a transactions-per-second number I do not have.

**Follow-up.** They force a number. I say what I would measure on the current primary and what headroom means. I do not fabricate Verizon peak traffic.

**Weak answer to avoid.** "Stateless services scale horizontally, so capacity is handled."

## 6. What are the failure modes you design for explicitly?

**Interviewer intent.** A list tied to behavior, not "we retry."

**Strong sample answer.** Payment timeout: reconcile, do not double-charge. Decline: release the hold. Crash after reserve: expiry worker. Duplicate submit: idempotency key. Duplicate webhook: event id. Redis down: browse degrades in a way the database can survive, checkout still prices on the primary. Primary down: stop selling. Replica lag: do not confirm the order from it. Hero SKU lock pileup: shed or isolate that key. I want each of these to have a customer-visible state, including "pending" when the outcome is unknown.

**Follow-up.** Poison message on the bus. Dead-letter after bounded attempts, metric on age, and a human path. Do not block the order commit on the consumer.

**Weak answer to avoid.** "We have retries and a circuit breaker, so failures are covered."

## 7. How does shipping and store pickup attach without coupling?

**Interviewer intent.** Outbox and a state machine.

**Strong sample answer.** The order commit includes an outbox row. A publisher emits order-paid. Fulfillment creates a shipment or a store pick and writes its own store. Status comes back as events the order service folds into a summary the customer can see. Allocation can be recomputed; the chosen result is stored. The associate UI reads the primary so a just-paid pickup is visible. The order API does not call the carrier synchronously. If fulfillment is down, the order is still paid and the outbox retries.

**Follow-up.** Split shipment. Fulfillment owns multiple units; the order summary is derived. Do not encode every package in the original order status enum.

**Weak answer to avoid.** "Checkout calls the warehouse synchronously and rolls back the payment if the warehouse is slow."

## 8. How do returns work without corrupting the order?

**Interviewer intent.** History and partial quantities.

**Strong sample answer.** A return is a new aggregate referencing order lines and the quantity already returned. Two partial returns cannot exceed the sold quantity, enforced in the return transaction. Refund is a provider call with its own idempotency key. Restock is a decision in inventory: a returned device may be graded and not allocatable. I do not update the original order row in place and lose the sale record. Chargebacks are a case, not another value crammed into order status.

**Follow-up.** Return in a store for a digital purchase. Same aggregate, channel recorded, refund path explicit. Inventory owner still decides restock.

**Weak answer to avoid.** "We delete the order and insert a refund."

## 9. What do the APIs look like between channel, cart, order, and inventory?

**Interviewer intent.** Boundaries and sync versus async.

**Strong sample answer.** Channels call catalog, cart, and checkout. They do not touch databases. Cart is synchronous and does not call inventory. Checkout calls inventory synchronously to reserve, with a timeout shorter than the caller, and calls payment with an unknown-on-timeout policy. Fulfillment is async from the outbox. Reads of order history for a customer go through the order API or a read model, not a shared table. Commands are `POST` with an idempotency key. Lists are cursor-paged. Errors say whether they are retryable.

**Follow-up.** A store associate and the website edit the cart together. One owner, version column, conflict visible to the client. Not last-write-wins on a Redis blob from both apps.

**Weak answer to avoid.** "One API gateway route to one database schema all the apps share."

## 10. What would you cut if the first release cannot include the full omnichannel picture?

**Interviewer intent.** Sequencing and risk.

**Strong sample answer.** I keep one channel, one stock pool, reprice on the primary, reservation with expiry, idempotent submit, and payment reconcile. I cut ship-from-store, split tenders, and a second search stack. Cache is allowed to be simple. I do not cut the idempotency key or the hold expiry to save time, because those are the correctness pieces. I would write the ADR for what we deferred so the next team does not think the shortcut is the target. Scope cuts are product decisions; invariant cuts are not.

**Follow-up.** They insist on both channels on day one. Then the shared inventory owner is in scope and the fancy browse features are what I cut instead.

**Weak answer to avoid.** "We build the platform first and the purchase path later."
