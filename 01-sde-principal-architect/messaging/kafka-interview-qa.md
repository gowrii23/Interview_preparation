# Kafka — interview questions

## 1. What ordering guarantee does Kafka actually give?

**Interviewer intent:** Stop the "the topic is ordered" myth.

**Strong answer:** Order holds inside a partition, not across a topic. The key selects the partition, so one aggregate id stays ordered if it always hashes to the same partition. A null key spreads records and gives up per-entity order. Global order means one partition and a throughput ceiling. I say that limit in the design review instead of promising order and then adding consumers.

**Follow-up:** You add partitions later. What happens to key order for new writes, and what is the catch?

**Weak answer to avoid:** "Kafka keeps the whole topic in order no matter how many partitions we add."

## 2. How does a consumer group assign work?

**Interviewer intent:** Parallelism equals partitions.

**Strong answer:** Members of one group divide the partitions. One partition goes to at most one member. Extra members sit idle. A second group has its own offsets and reads the same log without stealing work. Lag is log end minus the committed offset. I alert on lag and on time lag, because a poison record stalls a partition while CPU looks idle.

**Follow-up:** Ten consumers, three partitions. What do you change if you need more parallelism?

**Weak answer to avoid:** "Consumers in a group all get every message, like a topic exchange."

## 3. What is a rebalance, and why can it hurt?

**Interviewer intent:** Stability under deploys and slow handlers.

**Strong answer:** A rebalance runs when membership, partition count, or heartbeats change. An eager rebalance revokes partitions and pauses everyone; a cooperative incremental rebalance moves only what must move. A handler that does not poll in time looks dead and can cause a storm. I bound work between polls and shut down cleanly on SIGTERM so deploys do not look like crashes. I do not treat rebalance as free.

**Follow-up:** How does `max.poll.interval` interact with a slow database call?

**Weak answer to avoid:** "Rebalance is just Kubernetes restarting the broker."

## 4. What do replication, ISR, and `acks` mean together?

**Interviewer intent:** Durability without fairy tales.

**Strong answer:** Each partition has a leader and followers. The ISR is the set caught up enough to be eligible for election. `acks=1` waits for the leader. `acks=all` waits for the current ISR. I pair `acks=all` with a minimum ISR of two if a single broker loss must not accept a write that only one node saw. The producer can still time out and must retry. Followers do not accept produces.

**Follow-up:** The ISR shrinks to one. What does `acks=all` mean then if min.insync.replicas is 1?

**Weak answer to avoid:** "`acks=all` means exactly-once."

## 5. What does an idempotent producer guarantee, and what does it not?

**Interviewer intent:** Scope of deduplication.

**Strong answer:** It sends a producer id and sequence numbers so the broker can drop duplicates from retries of that producer on a partition during the producer session. It fixes the timeout-after-success double append. It does not dedupe two different producer instances, and it does not make the database update exactly once. I still design the consumer to tolerate duplicates.

**Follow-up:** When would you still see two business effects after enabling it?

**Weak answer to avoid:** "Idempotent producer means the whole pipeline is exactly-once."

## 6. Where do transactions help, and where do they not?

**Interviewer intent:** Conceptual accuracy, no fake API details.

**Strong answer:** A transaction lets a producer commit or abort writes across partitions, and it can include the consumer offset commit so a Kafka-to-Kafka read-process-write is atomic for consumers using read-committed isolation. It does not enlist a SQL database. For database side effects I use an outbox written in the same database transaction as the business row, then publish. I describe that boundary explicitly so nobody "turns on transactions" and thinks Oracle is included.

**Follow-up:** What does a consumer with read-uncommitted see?

**Weak answer to avoid:** "Kafka transactions are XA with our Spring `@Transactional` database."

## 7. How do you get effectively-once handling on the consumer?

**Interviewer intent:** The answer finance will accept.

**Strong answer:** Assume at-least-once delivery. Commit offsets after the side effect succeeds, or you can lose work; if you crash after the side effect and before commit, you will redeliver. I make the handler idempotent by storing the event id or business key in the same database transaction as the mutation, with a unique constraint. A duplicate hits the constraint and does nothing. That is effectively-once. I name the key in the design.

**Follow-up:** The side effect is a call to a third party that cannot share the transaction. What now?

**Weak answer to avoid:** "We ack immediately so the queue does not build up."

## 8. How do retention and compaction differ?

**Interviewer intent:** Log policy matches the data.

**Strong answer:** Time or size retention deletes old segments whether or not anyone read them. The log is not a queue that drains. Compaction keeps the latest record for each key and is right for a changelog. It does not keep every event, so it is the wrong policy for an audit stream. I pick delete for facts that subscribers must each see, and compaction for "current state by key."

**Follow-up:** Can a compacted topic still contain more than one record per key for a while?

**Weak answer to avoid:** "Compaction means the topic stays small and still stores full history."

## 9. How do you evolve a schema without breaking consumers?

**Interviewer intent:** Compatibility as an architectural control.

**Strong answer:** I use a schema registry with Avro, Protobuf, or JSON Schema and an agreed compatibility mode. Backward compatible changes add optional fields and do not reuse identifiers or change types, so old consumers read new data. If producers cannot move first, I need forward compatibility and I say which side deploys first. The schema is reviewed like an API. Unknown fields must not crash the consumer.

**Follow-up:** Someone wants to change a field from int to string. What do you tell them?

**Weak answer to avoid:** "JSON has no schema, so evolution is free."

## 10. A consumer is lagging. How do you reason about it?

**Interviewer intent:** A structured incident answer.

**Strong answer:** I check whether one partition is hot because of a bad key, whether a poison message is stuck, whether poll time is exceeded and rebalances are churning, and whether the handler's downstream is slow. Adding consumers only helps up to the partition count, and only if the lag is spread. I do not skip offsets until I know the business impact. I scale the dependency or park the poison record, then watch time lag, not only record lag.

**Follow-up:** Lag is only on one partition. What is the likely shape of the key?

**Weak answer to avoid:** "I add twenty consumers and restart the cluster."
