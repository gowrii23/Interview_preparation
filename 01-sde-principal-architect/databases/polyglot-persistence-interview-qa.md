# Polyglot persistence interview Q&A

## 1. Walk through which store owns which POS fact.

**Interviewer intent.** One writer per fact.

**Strong sample answer.** Price and assortment of record live in the merchandising relational database, Oracle if that estate already owns them, Postgres if the service is new. Redis holds a versioned display cache and ephemeral session, not the ledger. The order, payment attempts, and the idempotency key live in one transactional database. Allocatable inventory has a single owner, same database or an inventory service with its own transactional store, not both. Cassandra, if present, takes append-mostly telemetry keyed by entity and time. Search is a projection. Channels do not get a shared schema login to write around the owner.

**Follow-up.** Anonymous cart? Redis is allowed only if loss on failover is acceptable. Submit promotes the cart into the transactional owner.

**Weak answer to avoid.** "Each team picks the database they like for the tables they share."

## 2. Why is dual-write a defect?

**Interviewer intent.** The gap between two commits.

**Strong sample answer.** If I commit the order and then update Redis and then publish to Kafka, any crash in between leaves a durable fact and a missing projection, or the reverse if I reorder it. There is no automatic rollback across those systems. The pattern I want is one transaction that writes the order and an outbox row. A publisher sends the outbox and marks it sent. Consumers invalidate cache, feed search, and notify fulfillment. They are idempotent. Dual-write plus a hope of reconciliation with no anchor is how we get a manual queue after a sale event.

**Follow-up.** Outbox growth? It is a table we prune after publish, with the same operational care as any hot table, and the unpublished age is a metric.

**Weak answer to avoid.** "We use a distributed transaction across Oracle, Redis, and Kafka."

## 3. What consistency do you promise on the device page versus at checkout?

**Interviewer intent.** You can name the anomaly you accept.

**Strong sample answer.** The device page may be stale up to a TTL or a publish version. Marketing copy lag is acceptable. Checkout reprices and reserves on the primary, read-your-writes, inside a short transaction. A Redis replica or a Postgres replica is not allowed to confirm "order placed." Cassandra at `ONE` is not allowed to answer "can I sell this." I say that to product so they do not treat the browse count as a promise. The browse count is a hint. The reservation is the promise.

**Follow-up.** Customer just updated the cart and does not see it? That is read-your-writes on the cart owner. If cart is Redis, read the primary key from the node you wrote, and still treat failover loss as a stated risk.

**Weak answer to avoid.** "Everything is strongly consistent because we use ACID somewhere."

## 4. How do you size the failure of Redis?

**Interviewer intent.** Degradation versus cascading failure.

**Strong sample answer.** I decide before the incident. Either browse falls back to the database and the database is sized for `R_browse * fallback_ratio`, or browse fails closed and checkout stays up because it never needed the cache to price. An unlimited fallback turns a cache outage into a primary outage and takes orders down with the pages. I load-test the chosen fallback. Telemetry being down must not block submit. If it does, the dependency is in the wrong place.

**Follow-up.** Cache stampede plus fallback? A single-flight rebuild and a cap on concurrent database misses. The cap is part of the design.

**Weak answer to avoid.** "Redis is fast, so we do not plan for it being down."

## 5. When would you add Cassandra to an estate that already has Oracle and Redis?

**Interviewer intent.** A bar, not a default.

**Strong sample answer.** Only with a written access path: partition key, clustering order, consistency level, partition size bound, and a reason Redis plus the relational database cannot take the write rate. Heartbeats and clickstream qualify if we will run repairs and we accept tunable consistency. Orders do not qualify. I also need an owner for the cluster. If the team cannot staff that, the answer is no even if the data model fits. Operational cost is part of the architecture.

**Follow-up.** What did you reject? A secondary index for ad hoc merchandising queries, and a queue of jobs implemented as insert-and-delete.

**Weak answer to avoid.** "We add it for scale and figure out the queries later."

## 6. Saga versus a single database for reserve-then-pay?

**Interviewer intent.** You do not distribute a transaction for sport.

**Strong sample answer.** If inventory and order can live in one database, one transaction is simpler and I keep them together until a real boundary forces a split. If they are already separate services, reserve, then pay, then confirm, with a hold expiry that releases stock when payment times out or the process dies. Compensation is a business action, release or void, and every step is idempotent. I do not call that a saga across Redis locks. The expiry worker is mandatory. Without it, abandoned checkouts leak allocatable quantity.

**Follow-up.** What does the customer see on an unknown payment result? "Pending," while we reconcile by idempotency key. Not a second charge.

**Weak answer to avoid.** "Microservices mean each table is its own database, always."

## 7. How do you explain operational cost to a stakeholder who wants four engines?

**Interviewer intent.** You can say no with a cost, not a lecture.

**Strong sample answer.** Each engine adds backup, restore drill, on-call skill, client failure modes, and capacity planning. Oracle is license and process limits. Postgres is vacuum, pooling, and migrations. Redis is memory and hot keys, cheap only while it is a cache. Cassandra is repair, tombstones, and data-model review. I map the proposal onto the decision table and drop any store that does not change consistency, latency, or a real volume problem. The stakeholder can still choose the extra cluster if they fund the rota. I will not hide that cost inside a diagram.

**Follow-up.** What would change your mind? A measured write rate and a query shape the current primary cannot take after caching, plus a named operator.

**Weak answer to avoid.** "Open source has no cost."

## 8. How should a read replica be used from the channels?

**Interviewer intent.** Lag and read-your-writes.

**Strong sample answer.** Search feeds, reporting, and browse paths that tolerate lag. Not the response that confirms an order, not the associate screen that must see the payment that just landed, not a unique-constraint check. I put the lag in the product language if a screen uses the replica. Oracle Active Data Guard and Postgres streaming replicas are the same class of decision even though the mechanics differ. The application chooses the primary deliberately, not via a random load balancer in front of both.

**Follow-up.** Conflict on a replica read used to gate a write? You will oversell or double-submit. Gate on the primary.

**Weak answer to avoid.** "Replicas are strongly consistent once we paid for multi-AZ."

## 9. What do you put in an ADR for a persistence choice?

**Interviewer intent.** Decision record contents.

**Strong sample answer.** Context: workload, RPO, latency, team skill. Decision: which store is the owner. Rejected: the other engines and why, including "Redis as the ledger" and "Cassandra for the order." Consequences: what we must operate, what staleness we accept, how we migrate. A date and a superseding note when we change our minds. The ADR is short enough that a new engineer will read it. It does not include a tutorial on MVCC.

**Follow-up.** Who approves? The teams that will carry the pager, not only the person who wrote the slide.

**Weak answer to avoid.** An ADR that says "we chose Cassandra because it is web scale" with no query.

## 10. A launch day hero SKU is contested. Where does that contention live?

**Interviewer intent.** Hot key thinking across stores.

**Strong sample answer.** The display key in Redis is a hot read key. I accept eviction and stampede control around it. The allocatable quantity is a hot row or a hot partition in the transactional owner. I isolate that contention with a reservation scheme that does not lock the world, and I shed or queue excess checkout rather than letting it exhaust the connection pool. I do not move the hot count into Cassandra to escape a hot row and then pretend quorum gives me a ledger. I do not shard the SKU across Redis locks. The principal point is that popularity concentrates, so average QPS is the wrong capacity story.

**Follow-up.** How do you know it is one SKU? Database row lock waits, Redis latency on one key, and an application metric tagged by SKU. I add that tag before launch, not during it.

**Weak answer to avoid.** "We scale the stateless tier and the database will be fine."
