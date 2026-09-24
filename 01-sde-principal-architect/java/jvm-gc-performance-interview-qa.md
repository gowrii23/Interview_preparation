# JVM, GC, and performance — interview questions

## Q1. A POS quote endpoint's p99 jumped after a deploy. Where do you look first?

**What the interviewer is probing:** A method, not a collector slogan.

**Sample answer:** I do not start by changing GC flags. I compare the deploy to the latency breakdown in New Relic or whatever APM we run: is the time in the order service, in JDBC, in Feign to pricing, or in Redis? If the database time moved, this is a query or a lock, and a GC discussion is a distraction. If the time is inside the JVM and CPU is high with a high allocation rate, I look for a new per-SKU allocation, a log line that concatenates strings at INFO, or a JSON serializer walking a lazy Hibernate graph.

If GC logs show pauses that match the p99 spikes, I read occupancy and pause time before I touch `MaxGCPauseMillis`. G1 is already the default on Java 17. Raising the heap is reasonable when occupancy is legitimately near the limit and the working set is real, and it is the wrong fix for a leak or a hot allocation. I check Hikari pending connections, because pool wait looks like a latency spike and is not GC. I check whether the JVM is in warmup (JIT) on a fresh pod. I would bring one GC log, one transaction trace, and one pool metric to the review. I would not bring a suitcase of experimental flags.

**Follow-up:** The trace shows 40 ms in `ObjectMapper`. What do you change?

**Weak answer:** "Switch to ZGC and double the heap."

## Q2. Explain G1 as you would to a principal panel.

**What the interviewer is probing:** Practical G1, humongous objects, pause goals.

**Sample answer:** G1 is the default collector on Java 17. It splits the heap into regions that it uses as Eden, survivor, old, or humongous. New objects allocate in Eden. A young collection copies live objects into survivor regions or promotes them. Concurrent marking finds live data without a full stop, then mixed collections reclaim old regions that contain mostly garbage, together with a young collection. `-XX:MaxGCPauseMillis` is a goal, default 200 ms, not a hard deadline. G1 may miss it when the live set is large or allocation is extreme.

Objects larger than half a region are humongous. A receipt image or a giant `byte[]` on the request path occupies humongous regions and is awkward to reclaim. I would rather stream the image or keep it out of the heap than tune region size on day one. I set `-Xms` equal to `-Xmx` in a container so the heap does not resize, and I turn on GC logging. I do not claim G1 has no pauses. It is the right default when the service can tolerate short pauses and needs throughput. If the SLO is single-digit-millisecond pauses on a very large heap, that is a ZGC conversation, after allocation rate is sane.

**Follow-up:** What does "to-space exhausted" tell you?

**Weak answer:** "G1 never stops the world, so pauses are not from GC."

## Q3. When would you choose ZGC on a Java 17 service?

**What the interviewer is probing:** Tradeoffs, and the generational gap on 17.

**Sample answer:** I choose ZGC when pause time is the SLO and the heap is large enough that G1 mixed collections miss the goal: a multi-gigabyte in-memory catalog, a long-lived order cache, a tail latency budget that G1 cannot hold. ZGC uses colored pointers and load barriers so most work is concurrent and pauses stay very small. On Java 17 that ZGC is single-generation. A high allocation rate, which a POS quote can produce if it builds large short-lived graphs, hurts more than it does under generational ZGC, which is the mode I would plan for on a later JDK. ZGC also wants heap headroom. If I size the heap to 95 percent occupancy, I trade long pauses for allocation stalls.

I do not enable it because a blog promised "no GC." I need CPU for the barriers, and I need a load test that shows the pause histogram, not a screenshot of flags. Shenandoah is the other low-pause option; I would not pretend I have tuned both. If the p99 is a slow Oracle query, neither collector will save the cashier. The principal answer is the condition, the Java 17 limitation, and the measurement.

**Follow-up:** What would you re-measure after moving that service to generational ZGC on a newer JDK?

**Weak answer:** "ZGC is always faster than G1."

## Q4. The pod is OOMKilled but the heap dump shows free heap. What is going on?

**What the interviewer is probing:** Native memory versus heap.

**Sample answer:** `-Xmx` is not the process RSS. The cgroup kills the pod when RSS exceeds the limit, which includes the heap, metaspace, thread stacks, code cache, direct buffers, and native memory from the JDBC driver or Netty. If a heap histogram is modest, I look at native memory. Direct `ByteBuffer`s used by the HTTP client do not live in the Java heap. A leak of those buffers shows up as rising RSS with a flat heap. Thread stacks multiply with a cached thread pool that grew without bound during a gateway stall. Metaspace grows if something generates classes or leaks classloaders; that is less common on a single Boot process that does not hot-redeploy, and it is real if a library spins proxies.

I compare the container limit with what the JVM actually observed. Modern JDKs honor cgroup limits; an older JVM that sized the heap from the node can overshoot the pod limit. I would set the memory limit with room above `-Xmx` for native usage, and I would graph RSS next to heap. Taking a heap dump that is mostly empty does not explain an OOMK. NMT (Native Memory Tracking) on a canary is the tool I would use with a clear head, not on every pod by default.

**Follow-up:** How do virtual threads on Java 21 change the stack part of this story?

**Weak answer:** "Increase Xmx to the size of the container limit."

## Q5. What would you stop allocating on a hot SKU loop?

**What the interviewer is probing:** Concrete allocation, not "make it faster."

**Sample answer:** I look at a profiler sample of the quote path before I rewrite it. The usual waste is `new BigDecimal(String)` inside a loop over lines, `SimpleDateFormat` constructed per call (also not thread-safe), string concatenation in a log that is disabled, and boxing a `Long` key for every cache get. I price in `long` minor units so a line extension is a multiply and an exact add, not a decimal allocation. If the business calculation truly needs decimal quantities, I reuse a scale-aware `BigDecimal` built once, not parsed from a string per SKU.

`DateTimeFormatter` is immutable and can be a constant; `SimpleDateFormat` cannot. Parameterized logging avoids the message build when the level is off. I do not micro-optimize a path that runs once per order. I do refuse a per-line allocation that shows up as the top allocator when a store quotes a large catalog batch. Escape analysis may remove a non-escaping object, and I do not rely on it once the object is stored on the order. I would show a before-and-after allocation rate from the same JMeter scenario, not a claim that the code "looks lighter."

**Follow-up:** When is BigDecimal still the right public API?

**Weak answer:** "Allocation is cheap on modern JVMs, so ignore it."

## Q6. How do you read a GC log during an incident?

**What the interviewer is probing:** Whether you have actually looked at one.

**Sample answer:** I want the log on the pod (`-Xlog:gc*`) correlated with the latency chart. I look at the pause duration, the heap occupancy before and after, and whether the collection was young, mixed, or a full collection that should be rare on G1. A pause that matches the user-facing spike is a GC problem. A log full of short young collections and a high allocation rate is a code problem: we are creating garbage faster than necessary, and lengthening the pause goal will not remove the work. A heap that climbs to the limit and never returns is a leak or a cache with no bound, and I take one histogram or a dump from a canary, not from every pod, because the dump itself pauses and fills disk.

I check humongous allocations if I see regions dedicated to large arrays. I check whether the event started at deploy (warmup, cold cache) or grew over days (leak). I do not tune ten flags in the incident channel. I mitigate by scaling or rolling back, then I fix the allocation or the leak with evidence. Principal-level is naming what I would ignore: a single full GC at startup, or a pause well under the SLO.

**Follow-up:** What is the difference between a safepoint pause and a GC pause in that log?

**Weak answer:** "I search for the word ERROR in the GC log."

## Q7. HikariCP wait times are high and GC looks quiet. How do you size the pool?

**What the interviewer is probing:** Pool math and the "GC is not everything" judgment.

**Sample answer:** Pending connections with quiet GC means threads are waiting for a session, not for the collector. `maximumPoolSize` is per pod. I multiply by replica count and compare to the database's session limit, leaving room for batch and admin. Sizing the pool to the number of Tomcat threads just moves the queue into Oracle or Postgres, where each extra session costs memory and may slow the others. A smaller pool with a short `connectionTimeout` fails fast and keeps the database healthy. I set `maxLifetime` below the firewall or database idle cut so Hikari retires connections first. `leakDetectionThreshold` tells me when a transaction is held across a Feign call.

I also check whether OSIV is keeping a connection checked out for view rendering, and whether a `REQUIRES_NEW` loop is burning sessions. The fix might be fewer concurrent checkouts, a shorter transaction, or an index that makes each borrow brief, not a pool of 200. I would graph active, idle, and pending, plus database CPU. In the interview I refuse a single magic pool size; I state the formula: replicas times pool, versus sessions the engine can actually serve, given the query time.

**Follow-up:** What happens to in-flight tenders if you drop maximumPoolSize during a rolling restart?

**Weak answer:** "Set the pool equal to maxConnections and maximum Tomcat threads."

## Q8. How do compressed oops and object headers show up in a design?

**What the interviewer is probing:** Whether low-level layout ever changes a decision.

**Sample answer:** With compressed ordinary object pointers, on by default under roughly a 32 GB heap, references are 4 bytes. Crossing that threshold widens them and can make a heap just above the cutoff use more memory and suffer more cache misses than a heap just below it. I mention it if someone proposes a 40 GB heap "for the cache" without measuring. I do not lead with it for a 2 GB order service.

Object headers dominate when the payload is small. A `HashMap` of boxed `Integer` quantities is an entry object plus an `Integer` object per SKU, which is a lot of header for four bytes of data. On a hot in-process structure I care; on a request DTO I do not. Primitive collections are a dependency I would justify with a profile, not a default. I also separate this from escape analysis: a short-lived object that does not escape can be scalar-replaced, so a theoretical allocation may not appear. Once the price is stored, it is real. The principal framing is that layout is a tool after measurement, and the first win is usually fewer objects, not a flag.

**Follow-up:** Why might a 34 GB heap be slower than a 30 GB heap on the same data?

**Weak answer:** "Objects are 8 bytes, so a million of them is 8 MB."

## Q9. Design a performance test that would satisfy you before a peak event.

**What the interviewer is probing:** JMeter used honestly, without fake numbers.

**Sample answer:** I write a scenario that matches the cashier, not a single URL in a loop with a tiny payload. Example: quote a basket of a realistic line count, then reserve one hot SKU concurrently, with the payment gateway stubbed at a latency we have measured, against a database loaded with a catalog, not ten rows. JMeter or an equivalent drives it from a machine that is not the server. I record error rate, a high percentile, pool pending, GC pause, and downstream time. I set the threshold from the SLO the business already has. I do not invent a throughput trophy number in the interview, and I do not accept a laptop run against shared dev as evidence.

I run it long enough to pass JIT warmup, and I run a case where Redis is down and a case where inventory is slow, because peak failure is usually a dependency, not the happy path. The output I want is the bottleneck: lock on the hot SKU, pool, or GC. A chart with no bottleneck named is not a result. I also make sure the test data's idempotency keys are unique per attempt so I am not accidentally measuring duplicate-key failures and calling them capacity.

**Follow-up:** How do you stop the test from becoming a production incident against a shared environment?

**Weak answer:** "We ran JMeter and it was fast."

## Q10. A heap dump shows one HashMap retaining most of the heap. What do you do?

**What the interviewer is probing:** Leak versus cache, and a safe operational response.

**Sample answer:** I identify the owner of the map from the path: a static field, a singleton Spring bean, a `ThreadLocal`, or a session that never closes. If it is a price cache with no eviction, that is a bounded-cache fix, not a collector flag. If the keys include a request id or a timestamp that never repeats, it is a leak, and I find the insert without a remove. If the map is the catalog and the business asked to pin it, the heap is too small for the working set, or we should move it to Redis so every pod does not hold a full copy.

Operationally I do not take dumps from every pod in a cluster during the incident; one pod is enough and the dump can pause the process and fill the disk. I mitigate by a restart if the leak is slow, with a ticket to fix the bound, and I add a metric on the map's size so it is visible before RSS alarms. I check whether a `ClassLoader` is retained only if the dominator is metadata; a plain `HashMap` of prices is not a metaspace issue. The review comment after the fix is: every structure that outlives the request needs a maximum size and a reason.

**Follow-up:** How would you confirm the fix without waiting for the next OOM?

**Weak answer:** "Restart more often and increase the heap limit."
