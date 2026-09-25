# Kafka vs RabbitMQ — interview questions

## 1. How do you explain the core difference in one minute?

**Interviewer intent:** A crisp model, not a feature list.

**Strong answer:** Kafka is a retained, partitioned log. Consumers pull and store their own offset, so several groups can read the same history until retention removes it. RabbitMQ routes through an exchange into queues and deletes a message when a consumer acks. I choose Kafka when I need replay or per-key order at parallelism. I choose RabbitMQ when I need a work queue or rich routing and the message should drain. I do not call them interchangeable queues.

**Follow-up:** Which mental model matches "competing workers on a job," and why?

**Weak answer to avoid:** "Kafka is newer, so it replaces RabbitMQ."

## 2. A product owner wants "exactly-once" on either broker. What do you say?

**Interviewer intent:** You will not over-claim.

**Strong answer:** Both systems can redeliver if we crash after the side effect and before ack or offset commit, and both can lose work if we ack too early. Producer idempotence and `acks=all` improve the log write; they do not cover the database. I commit to effectively-once by making the handler idempotent on a business key stored with the mutation. Kafka transactions cover Kafka-to-Kafka. RabbitMQ confirms cover broker acceptance, not the worker. I write the guarantee in those words.

**Follow-up:** How does an outbox change the producer side of this answer?

**Weak answer to avoid:** "We turn on the exactly-once flag and move on."

## 3. When is per-key ordering a reason to pick Kafka?

**Interviewer intent:** A concrete selection rule.

**Strong answer:** If one account's events must be handled in order and many accounts must run in parallel, a Kafka key that hashes to a partition gives that, with one consumer per partition in the group. RabbitMQ competing consumers will interleave a shared queue. I would not invent a single-active consumer plus a lock service on RabbitMQ just to imitate partitions unless the volume is small and the team already lives on RabbitMQ. I also warn that a hot key collapses that partition to one consumer's speed.

**Follow-up:** What do you do if one key has half the traffic?

**Weak answer to avoid:** "RabbitMQ is FIFO, so ordering is solved."

## 4. When is RabbitMQ the better fit even if the team likes Kafka?

**Interviewer intent:** You can resist fashion.

**Strong answer:** A task that should be routed by key pattern, acked per message, retried with a delay, and parked when poison, with no replay requirement, fits RabbitMQ. Fanout to a small set of workers, request and reply, and per-queue TTL are natural. Standing up a compacted or retained topic for a short-lived job adds retention, partition planning, and lag operations I do not need. I pick the drain-the-queue semantics on purpose.

**Follow-up:** How would you dead-letter the same job on Kafka if you had already chosen it?

**Weak answer to avoid:** "We already run Kafka, so every message goes there."

## 5. How does backpressure show up on each system?

**Interviewer intent:** Operations, not brochures.

**Strong answer:** On Kafka, a slow consumer stops committing, lag grows, and the log uses disk until retention. The producer is not blocked by that consumer. On RabbitMQ, the queue depth grows and prefetch limits how much sits unacked on clients. Memory or disk alarms can block publishers, which is a sharper backpressure signal and a production incident if you sized the node for a tiny queue. I alert on lag for Kafka and on ready plus unacked depth for RabbitMQ.

**Follow-up:** Why can a Kafka consumer be "fine" on CPU and still be an incident?

**Weak answer to avoid:** "Both brokers slow the producer automatically when consumers are slow."

## 6. How do you replay yesterday's events on each?

**Interviewer intent:** Replay is the deciding feature more often than throughput.

**Strong answer:** On Kafka I start a group at an earlier offset, or a new group, within retention, and the handler must be idempotent because it will see records it may have applied. Compaction may already have dropped old values for a key. On RabbitMQ I cannot rewind the queue. I would have needed an archive, a log, or a second consumer that stored the payload at the time. If replay is a real recovery requirement, I choose Kafka up front rather than bolting a tap on later and hoping it was complete.

**Follow-up:** What do you do about side effects when you replay?

**Weak answer to avoid:** "I set a RabbitMQ offset back to yesterday."

## 7. Can you run both? How do you stop dual-write chaos?

**Interviewer intent:** Platform rules.

**Strong answer:** Yes, with a written rule. Domain facts that several contexts store and may replay go to Kafka. User-driven jobs and routed commands go to RabbitMQ. One owner publishes. If a service must update its database and emit a fact, it writes an outbox in the same transaction and a publisher relays to the broker. Publishing to both brokers "for safety" without that outbox will diverge. I do not let each team invent a third path.

**Follow-up:** The outbox publisher crashes after the database commit. What is the user-visible state?

**Weak answer to avoid:** "Every service publishes to both so we can migrate gradually without a plan."

## 8. How do poison messages differ operationally?

**Interviewer intent:** Failure handling compared, not just the happy path.

**Strong answer:** On RabbitMQ I nack without requeue to a dead-letter exchange, delay a limited retry, then park and alert. The rest of the queue keeps moving if prefetch and competing consumers are set up. On Kafka a poison record blocks a partition if the handler refuses to continue, and lag on that partition grows. I park it with an explicit decision, or send it to a dead-letter topic, and I keep the offset moving only once that decision is safe. Skipping blindly loses data. Neither tool makes a bad payload harmless.

**Follow-up:** What header or key do you require so support can find the parked message?

**Weak answer to avoid:** "Poison messages only happen if the broker is misconfigured."

## 9. A colleague says Kafka is always faster. How do you answer?

**Interviewer intent:** Judgment over benchmarks.

**Strong answer:** Kafka's log, batching, and many independent readers are excellent when those are the requirements. RabbitMQ moves serious task volume and will be simpler when the need is routing and a draining queue. Speed depends on partition design, ack mode, replication, message size, and handler time, which usually dominates. I ask for the job, the order rule, the replay rule, and the poison path before I look at a benchmark. A faster broker that cannot express the routing or the retention we need is the wrong system.

**Follow-up:** What number would you actually measure in a proof of concept?

**Weak answer to avoid:** "Kafka is always an order of magnitude faster, so the comparison ends there."

## 10. Walk through a decision for "order placed" versus "generate invoice PDF."

**Interviewer intent:** Apply the model to two concrete flows.

**Strong answer:** "Order placed" is a fact several consumers keep: billing, fraud, warehouse, analytics. I put it on Kafka with an order-id key, a backward-compatible schema, retention long enough to replay an incident, and idempotent consumers. "Generate invoice PDF" is a job. I put it on RabbitMQ, direct exchange, competing workers, prefetch set, confirms on, dead-letter after limited retries. I do not retain that job forever, and I do not require a second team to replay PDFs from a log. If invoice generation also emits "invoice ready," that fact can go to Kafka from an outbox, separate from the job queue.

**Follow-up:** Warehouse missed two hours of "order placed." What do you do on each design?

**Weak answer to avoid:** "Both go on whichever broker is already in the diagram."
