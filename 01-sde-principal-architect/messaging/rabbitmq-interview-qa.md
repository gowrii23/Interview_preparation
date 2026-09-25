# RabbitMQ — interview questions

## 1. What is the path a message takes inside RabbitMQ?

**Interviewer intent:** Exchange versus queue, the basic model.

**Strong answer:** The publisher sends to an exchange. Bindings route the message into one or more queues. Consumers take from a queue, and an ack deletes the message. The queue is the buffer; the exchange is the router. Publishing "to a queue" is really the default exchange using the queue name as the routing key. If nothing is bound, the message is dropped unless the publisher used mandatory and handles the return.

**Follow-up:** Two queues are bound to a fanout exchange. How many consumer deliveries happen?

**Weak answer to avoid:** "The producer writes the queue, and the exchange is optional decoration."

## 2. Compare direct, topic, fanout, and headers exchanges.

**Interviewer intent:** You can choose one and defend it.

**Strong answer:** Direct matches the routing key exactly. Topic matches patterns, where `*` is one word and `#` is zero or more, which fits event names such as `order.created`. Fanout copies to every bound queue and ignores the key. Headers match header attributes with `x-match` all or any. I default to direct for work queues and topic for a family of events. Headers are harder to operate because the routing logic is less visible, so I use them only when the selector is truly a set of attributes.

**Follow-up:** Does `order.*` match `order.created.eu`?

**Weak answer to avoid:** "All exchanges broadcast, like Kafka consumer groups."

## 3. What are competing consumers, and what happens to order?

**Interviewer intent:** Scale-out versus ordering, honestly.

**Strong answer:** Several consumers on one queue share messages; each message goes to one of them. That is how I scale workers. Order does not survive that, and a prefetch above one can finish work out of order even with a single consumer. RabbitMQ will not pin a business key to a partition the way Kafka will. If I need per-entity order, I say so and pick a different design instead of promising order on a shared queue.

**Follow-up:** When would a single active consumer be the right compromise?

**Weak answer to avoid:** "Competing consumers keep FIFO for the whole queue."

## 4. Why manual ack, and what is wrong with requeue on every failure?

**Interviewer intent:** Poison messages.

**Strong answer:** Auto-ack drops the message when it is delivered, so a worker crash loses it. I ack after the side effect commits. `nack` with requeue true puts it back immediately, which hot-loops a poison message. I nack without requeue after a bounded failure and dead-letter it. Retry belongs on a delay queue with a death count, then a parking queue for a human. I do not retry forever.

**Follow-up:** Where do you count attempts if the handler process dies?

**Weak answer to avoid:** "I requeue so we never lose a message, whatever the error is."

## 5. What is prefetch, and what happens if you leave it unbounded?

**Interviewer intent:** The main fairness control.

**Strong answer:** Prefetch caps how many unacked messages a consumer may hold. A huge prefetch lets one fast connection drain the queue into one process. Other workers idle, memory blows up on the client, and a crash redelivers a large set. I set prefetch to what one instance can finish promptly. It is both fairness and backpressure.

**Follow-up:** Prefetch is 1 and throughput is poor. What do you weigh before raising it?

**Weak answer to avoid:** "Prefetch is a producer setting for batch size."

## 6. What does a publisher confirm actually promise?

**Interviewer intent:** No false durability.

**Strong answer:** A confirm means the broker accepted responsibility for the message according to its routing and durability rules. I wait for it before I forget the send. It is not a promise that a consumer succeeded. Mandatory returns tell me the message matched no queue. Durable exchange, durable queue, and persistent delivery are all required to survive a broker restart, and a single-node durable queue still dies with that node. For node loss I want a replicated queue such as a quorum queue, described as a requirement if I do not know the cluster's exact version.

**Follow-up:** Confirms are on, but the queue is not durable. What survives a restart?

**Weak answer to avoid:** "If publish returned, the worker has finished the job."

## 7. How do TTL and dead-letter exchanges fit a retry design?

**Interviewer intent:** A concrete failure path.

**Strong answer:** TTL expires a message or the contents of a queue. Expired, rejected, or overflowed messages can be published to a dead-letter exchange with a reason header. I bind a retry queue whose messages expire and route back to the main work queue, and after N deaths I route to a parking queue that alerts. The main consumer nacks without requeue so the hot path stays clean. I can see death counts in headers instead of hiding retries in the worker.

**Follow-up:** Why not sleep inside the consumer to delay a retry?

**Weak answer to avoid:** "Dead letter means the message is deleted."

## 8. How is RabbitMQ's delivery model different from Kafka's?

**Interviewer intent:** The honest contrast, briefly.

**Strong answer:** RabbitMQ pushes to consumers and deletes on ack. Kafka retains a log and consumers pull at an offset, so other groups can replay. RabbitMQ's strength is routing and work distribution. Kafka's strength is the retained, partitioned log. I do not use RabbitMQ as an event store, and I do not use Kafka only because a task queue was fashionable. Handlers on both must be idempotent.

**Follow-up:** Name one requirement that would make you migrate a flow from RabbitMQ to Kafka.

**Weak answer to avoid:** "They are the same broker with different clients."

## 9. A queue depth is growing. What do you look at?

**Interviewer intent:** Operate, do not guess.

**Strong answer:** I separate ready messages from unacked. Unacked with idle consumers means workers are stuck or prefetch is hoarding. Ready and climbing means too few consumers or a slow handler. I check consumer count, prefetch, downstream latency, and alarm state on memory or disk. I do not purge a payments queue to make the graph pretty. If one poison message is blocking a single consumer, I dead-letter it.

**Follow-up:** Consumers are at 100% and unacked equals prefetch times consumers. What does that imply?

**Weak answer to avoid:** "I restart the broker to clear the backlog."

## 10. What do you require in a design that publishes to RabbitMQ?

**Interviewer intent:** A bar you can enforce in review.

**Strong answer:** A named exchange type and binding, publisher confirms, persistent messages on a durable replicated queue if loss matters, manual ack after the side effect, a finite prefetch, and a dead-letter path with a retry limit. The payload has an idempotency key. I write down that order is not guaranteed once we scale consumers. I reject a design that auto-acks and "will add dead-letter later."

**Follow-up:** Which of those can you relax for a metrics firehose, and why?

**Weak answer to avoid:** "We publish and assume the broker will do the reliable thing by default."
