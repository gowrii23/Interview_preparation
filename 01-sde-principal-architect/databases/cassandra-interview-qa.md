# Cassandra interview Q&A

## 1. Explain partition key versus clustering key with a POS example.

**Interviewer intent.** Query-driven modeling, not "it is like a primary key."

**Strong sample answer.** The partition key decides the token and therefore the replicas that store the data. Clustering columns sort rows inside that partition. "Events for this store device, newest first" is partition key device id plus a time bucket, clustering column event time descending. "Fetch by order id" is a different table if that is a different query. I do not filter on a clustering column alone and turn on `ALLOW FILTERING`. If two screens need two access paths, the writer maintains two tables.

**Follow-up.** Why a time bucket? A partition of device id alone grows without bound. The bucket keeps compaction and reads inside a bound, and a longer query reads a fixed number of buckets.

**Weak answer to avoid.** "The partition key is for uniqueness and the clustering key is just extra."

## 2. What is a wide partition, and why does it hurt?

**Interviewer intent.** Operational consequence, not a slogan.

**Strong sample answer.** Too many rows or too many large cells in one partition. Compaction, repair, and a single read all pay for that size. Coordinators and replicas allocate to serve it, and timeouts show up as a hot partition rather than a cluster-wide CPU problem. The common heuristic is to stay well under about 100 MB per partition. That is operational guidance, not a hard engine constant. Collections count toward the same budget. I bucket time series and I page bounded slices.

**Follow-up.** How do you find them? Table statistics, a size report, or tracing a timeout to one key. Then we split the key with a bucket and migrate with a new table.

**Weak answer to avoid.** "Cassandra scales linearly, so partition size does not matter."

## 3. `ONE`, `QUORUM`, and `LOCAL_QUORUM` — which do you pick for multi-DC?

**Interviewer intent.** RF versus consistency level, and cross-DC latency.

**Strong sample answer.** RF is how many copies exist. Consistency level is how many we wait for on this operation. `ONE` is fast and can return stale data. `QUORUM` is a majority of all replicas, so a multi-DC cluster waits on remote copies. `LOCAL_QUORUM` is a majority in the local DC, which is what I want for an online path that must survive a slow remote region. With RF 3 in the DC, local quorum is two. Pairing local-quorum reads and writes gives overlap on that partition. It is not a cross-partition transaction. `ALL` fails when any replica is down. `EACH_QUORUM` couples us to every region.

**Follow-up.** Does `R + W > RF` fix oversell across inventory and orders? No. It applies to replicas of one partition, not to two aggregates.

**Weak answer to avoid.** "We use `ONE` for everything because Cassandra is eventually consistent anyway."

## 4. Why do tombstones cause timeouts on a pattern that "just deletes rows"?

**Interviewer intent.** Queue anti-pattern and `gc_grace_seconds`.

**Strong sample answer.** A delete writes a tombstone. Reads must merge tombstones with older replicas so the row does not come back. Tombstones are eligible to disappear only after `gc_grace_seconds`, default ten days, and only if repairs have spread the delete. A queue that inserts and deletes in one partition fills with tombstones, and the read of the "next" item scans them until it times out. TTL expiry is also a tombstone. I do not use Cassandra as a work queue. I do not shorten gc_grace to hide the problem unless repairs actually finish inside the new window, or deleted data resurrects.

**Follow-up.** Range tombstones? A partition or range delete is more expensive to reconcile than one cell. Prefer time-window compaction for true time series, and avoid random deletes.

**Weak answer to avoid.** "Deletes free the disk immediately, like a relational database."

## 5. Why is `ALLOW FILTERING` a design smell?

**Interviewer intent.** You will refuse an unbounded query.

**Strong sample answer.** It means the table does not match the predicate, so Cassandra scans and then filters. On a large table that is a cluster walk. The fix is a table whose partition key is the thing we filter by, written at the same time as the original mutation, or an admission that this query belongs in analytics or a search index. Secondary indexes fan out to nodes and disappoint under cardinality you did not expect. Storage-attached indexes help some cases in newer versions and still are not a reason to treat Cassandra as ad hoc SQL.

**Follow-up.** A coordinator `IN` of thousands of partitions? That is the same class of bug. Bound the fan-out.

**Weak answer to avoid.** "Allow filtering is fine if we add a cache in front."

## 6. What transaction facilities exist, and what do people falsely assume?

**Interviewer intent.** LWT and batches.

**Strong sample answer.** Lightweight transactions are Paxos on one partition, several round trips, for a rare compare-and-set. They are not checkout throughput and they do not span partitions. Logged batches replay mutations atomically via a batch log; they are not isolated, and a logged batch across many partitions is a performance trap. Unlogged batches are an optimization for one partition. There is no multi-row ACID transaction you can use to update a customer, a stock count, and a payment together. If the invariant needs that, the owner is Postgres or Oracle.

**Follow-up.** Counters? Special reconciliation, not an auditable balance. I would not store allocatable quantity as a counter.

**Weak answer to avoid.** "Batches are transactions, so we are safe."

## 7. Design the table for "orders by customer" and "order by id." Why two?

**Interviewer intent.** Duplication is the product.

**Strong sample answer.** First table: partition key customer id, clustering order time and order id, columns the history screen needs. Second table: partition key order id, the detail payload. The order service writes both. I am explicit about what is duplicated and who repairs drift. I do not join. I page the history with a bounded clustering range. If the customer has huge history, I bucket by month. Search by email or phone is not this table; that is another access path or another system.

**Follow-up.** What if one write succeeds and the second fails? Retry a batch logged only if we accept the cost, or reconcile from the system of record. If the order database is relational, Cassandra should not be the owner of either table. It should be a projection, fed by the outbox, for a read that truly needs it.

**Weak answer to avoid.** "One table with a secondary index on both customer id and order id."

## 8. When do you tell the team not to use Cassandra?

**Interviewer intent.** The principal "no."

**Strong sample answer.** When the workload is a relational order with constraints, when we need ad hoc reporting, when the data fits in Postgres with a Redis cache, or when nobody will run repair. A yes looks like append-mostly telemetry keyed by entity and time bucket, read back the same way, `LOCAL_QUORUM`, partitions bounded, and an on-call path that understands tombstones. "We might get large" is not a yes. I would rather a boring primary and a cache than a cluster we query with `ALLOW FILTERING`.

**Follow-up.** What do you operate weekly? Repairs within gc_grace, compaction, disk, latency by consistency level, and a check for large partitions.

**Weak answer to avoid.** "Cassandra is web scale, so it replaces the order database."

## 9. A read at `LOCAL_QUORUM` returned stale availability. What do you check?

**Interviewer intent.** Clocks are not the first guess; the write's consistency and the model are.

**Strong sample answer.** I check the write's consistency level. A write at `ONE` acknowledged on a replica that is not in the read's quorum can still be invisible until repair or read repair. I check whether we read a different table than we wrote, or a bucket key that does not match. I check tombstones and gc_grace if this is a delete that reappeared. I do not start by blaming the client driver. If the business rule is "never show available when we cannot allocate," the flag belongs in the transactional inventory service, and Cassandra should not be on that path.

**Follow-up.** Read repair? It fixes replicas on a read that noticed a digest mismatch. It is not a substitute for a write consistency you chose too low.

**Weak answer to avoid.** "Eventual consistency means the bug is expected and we do not fix it."

## 10. How do you keep a telemetry model from becoming a second order system?

**Interviewer intent.** Ownership and coupling.

**Strong sample answer.** The order commits in the relational database with an outbox. Telemetry consumes a stream and writes partitions we can afford to lose or delay. Checkout does not read Cassandra to decide if it can sell. The partition key is device or store plus a time bucket, never "all events today" in one partition. Retention is a TTL plus time-window compaction if the table is pure time series, with eyes open about tombstones. If a stakeholder asks for a join from telemetry to payment state inside the purchase path, that is a redesign, not a new query.

**Follow-up.** Analytics over history? Export or Spark on the bulk path, not the online coordinator.

**Weak answer to avoid.** "We will add columns until the telemetry table can serve checkout too."
