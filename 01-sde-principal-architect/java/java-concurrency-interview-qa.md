# Java concurrency — interview questions

## Q1. What does happens-before mean on a tender path?

**What the interviewer is probing:** The memory model, not the words "thread-safe."

**Sample answer:** A write in thread A is not guaranteed visible to thread B unless a happens-before edge orders them. The edges I rely on are program order within one thread, monitor unlock before a later lock of the same monitor, a volatile write before a later volatile read of that field, `start` before the new thread runs, and the actions of a thread before `join` returns. The relation is transitive. Without an edge, the JIT and the CPU may reorder or keep a flag in a register, so a cashier thread can spin forever on a `done` boolean that another thread set.

`volatile` publishes that flag and orders surrounding writes only in the limited way the JMM specifies. It does not make `count++` atomic. For a one-way "shutdown" flag, volatile is enough. For an idempotency map, volatile on the map reference is enough only if I publish a new immutable map with one swing. In-place mutation needs a concurrent map or a lock. Safe publication also comes from `final` fields once the constructor finishes, which is why an immutable `Money` can be shared with no lock. I do not start a thread or register a listener inside a constructor; that publishes `this` too early. In a principal interview I name the edge for the specific field, and I refuse to say "volatile makes it thread-safe" as a complete answer.

**Follow-up:** Does a volatile write of a reference guarantee the fields inside the object are visible?

**Weak answer:** "The JVM flushes the cache when you use volatile."

## Q2. Where do you put the lock around payment authorization?

**What the interviewer is probing:** Critical-section length and double-charge risk.

**Sample answer:** I do not hold a lock, a transaction, or a Hikari connection across the gateway call. The critical section is the local decision: have we already accepted this idempotency key, and can I mark it in progress. That is a unique insert in Postgres or Oracle, or `ConcurrentHashMap.compute` only as a single-JVM optimization in front of that insert. The winner calls the gateway with a timeout. The loser returns the stored receipt and does not authorize again.

If the gateway times out, the outcome is unknown, not declined. I persist `UNKNOWN` and let reconciliation ask the gateway by its key. A blind retry of capture is how a POS double-charges. `synchronized` on the service instance would also be wrong: it serializes every store on one monitor, and locking `this` lets a caller outside the class participate in the lock order. I use a private final lock only if the state is truly local. Deadlock shows up when tender code locks the order and then the customer while another path locks the customer and then the order. I fix the order of locks or I stop taking two.

A thread dump during an incident should show threads blocked in HTTP, not piled up on one monitor. If they are piled up, the lock is around I/O.

**Follow-up:** How do you make the database insert and the "in progress" state one step?

**Weak answer:** "I synchronize the authorize method."

## Q3. How does ConcurrentHashMap lock, and when is that not enough?

**What the interviewer is probing:** Java 8 bin locking versus a synchronized wrapper, and scope.

**Sample answer:** On Java 8 and 17, `ConcurrentHashMap` installs a node into an empty bin with CAS. Updates to a non-empty bin synchronize on the head node of that bin, not on the whole map. Reads are volatile reads of the table and do not take that lock. `compute` and `merge` are atomic for one key and hold the bin lock while the function runs, so the function must not block and must not update other keys in the same map. Size is maintained with striped counters, similar in spirit to `LongAdder`, so writers do not all bounce one cache line. Iterators are weakly consistent: no `ConcurrentModificationException`, and a concurrent insert may or may not appear.

`Collections.synchronizedMap` is one mutex around every call and still does not make two calls atomic. I choose CHM for a shared in-process structure with atomic single-key updates. I do not choose it as the inventory authority across pods. Two `get` calls and a `put` of a decremented quantity are not atomic even on a CHM; I need `compute` or, for real stock, a conditional SQL update. I also mention that null is illegal, which breaks code migrated from `HashMap` that used null as a sentinel for "price missing."

**Follow-up:** What goes wrong if the `compute` function calls another service?

**Weak answer:** "It uses segments, so there are 16 locks."

## Q4. Design the executor for quote fan-out.

**What the interviewer is probing:** Pool bounds, `CompletableFuture`, and timeouts.

**Sample answer:** Price, tax, and loyalty can run together only if they are independent and each has a timeout. I use `CompletableFuture.supplyAsync` with an explicit executor, not the common pool by accident. `thenApply` may run on the completing thread; if that thread is an HTTP callback I do not do heavy CPU work there. `thenApplyAsync` takes the executor I mean. I set `orTimeout` (Java 9+) so a stuck tax service completes the future exceptionally inside the cashier's budget. I always attach `handle` or `exceptionally`. An unhandled exceptional future is a swallowed failure or a later `join` that surprises the caller.

The pool for blocking Feign calls is bounded. An unbounded cached pool creates a thread per stuck call until the pod dies. The queue in front of a small pool is also bounded, or memory grows while latency looks fine. I name threads `pricing-io-n` so a dump in New Relic or a jstack is readable. Cancellation of the future does not abort a JDBC call already inside the driver; I say that out loud. Loyalty is optional: its failure becomes a sealed `Skipped`, not a failed quote. Tax is required: its timeout fails the quote. That split is the design, not a retry loop around everything.

**Follow-up:** How do you propagate a correlation id onto those threads?

**Weak answer:** "I use parallelStream for price, tax, and loyalty."

## Q5. A counter of scans per minute is wrong under load. Why?

**What the interviewer is probing:** Atomicity versus visibility, and `LongAdder`.

**Sample answer:** A `long` updated with `count++` loses increments because the operation is read, add, write, and two threads read the same value. Marking the field `volatile` makes each write visible and still loses updates, because volatile is not a lock around the compound action. `AtomicLong.incrementAndGet` is correct and will contend on one cache line if every scan in the store hits it. `LongAdder` stripes the counter and sums on read, which is the right shape for a metric that is incremented on every request and read occasionally. It is a poor choice if I need a single strongly ordered value to decide a business rule at each increment.

If the number must be exact across pods, neither atomic is enough; I would emit a metric to New Relic or aggregate in the database, and I would not pretend a JVM counter is the chain-wide total. I also check that the increment is not inside a `synchronized` block that also does I/O, which would be correct and slow. For a principal answer I show the lost-update interleaving on the board in three lines, then name the class I would use and why its read path is more expensive.

**Follow-up:** When is a synchronized block clearer than an atomic?

**Weak answer:** "Add volatile and the increments become atomic."

## Q6. What changes if the service moves from Java 17 to virtual threads?

**What the interviewer is probing:** Forward-looking judgment without pretending the resume is Java 21.

**Sample answer:** Virtual threads, a platform feature in Java 21, make a blocked thread cheap. The JVM mounts many virtual threads on a small set of carrier threads. For a POS service whose threads mostly wait on JDBC, Redis, or Feign, I could replace a large platform pool with `Executors.newVirtualThreadPerTaskExecutor` and stop sizing the pool to the concurrency of slow I/O. I would not pool virtual threads. I would not expect a CPU-bound pricing loop to get faster; carriers are still limited by cores.

The database does not grow. HikariCP `maximumPoolSize` remains the real cap, and a virtual thread blocked in `getConnection` is still a waiting request. On Java 21, a virtual thread can pin a carrier inside `synchronized` or a native frame, so I would review hot `synchronized` blocks and prefer locks that do not pin, or keep those sections tiny. Later JDKs improved the `synchronized` case; I would name the JDK I was actually on. Thread locals that assumed a small pool, including some transaction or MDC patterns, can become a memory problem if every request creates a virtual thread and a heavy thread local. Structured concurrency is how I would group the price and tax fan-out so one failure cancels the sibling. I would re-test timeouts and dumps; I would not rewrite JPA.

**Follow-up:** How would you detect pinning in a load test?

**Weak answer:** "Virtual threads remove the need for a connection pool."

## Q7. Debug a deadlock reported during checkout.

**What the interviewer is probing:** Evidence, lock order, and a fix that is not "add a timeout and hope."

**Sample answer:** I take a thread dump from the affected pod, not from my laptop. I look for threads in `BLOCKED` whose lock owners form a cycle: tender holds the order monitor and wants the customer monitor, while a profile update holds the customer and wants the order. Jstack or the dump in the APM shows the monitors and the owning thread ids. I also check for a thread stuck in `compute` on a `ConcurrentHashMap` that called back into the same map.

The fix is a global lock order, or one lock, or removing the need by making one of the updates asynchronous. `tryLock` with a timeout is a recovery tactic so a cashier gets an error instead of a hung thread; it is not the design. I add a test only if I can force the order; many deadlocks will not reproduce in unit tests, so the dump is the real artifact. I look for locks taken in a filter plus a service plus a listener on the same request. I check that we do not synchronize on an interned String or a boxed integer shared across the JVM. After the fix I confirm pool and thread metrics return, and I write down the lock order next to the code so the next feature does not reintroduce the cycle.

**Follow-up:** How does a database deadlock differ from this Java deadlock in the dump?

**Weak answer:** "Restart the pod on a schedule."

## Q8. How do you publish a new price snapshot to request threads?

**What the interviewer is probing:** Safe publication and copy-on-write versus in-place mutation.

**Sample answer:** I build the new map off to the side, wrap it so callers cannot mutate it, and then publish it with a volatile write of the reference or an `AtomicReference.set`. Readers who loaded the old reference keep a consistent snapshot. Readers who load after the write see the new map, including all entries, because the volatile or atomic publication happens-before the reader's read and the writes that filled the map happen-before that publication in the writer thread. I never mutate the map after publication. `HashMap` is not safe to update in place while others read it.

If prices arrive as a stream of single-SKU updates, a `ConcurrentHashMap` updated with `compute` is simpler than a full copy. I choose copy-on-write when reads dominate and the book is replaced as a batch. I choose CHM when updates are fine-grained. Either way, a local snapshot can be stale relative to Postgres. The TTL or the version I publish is a product decision. I do not use this pattern for on-hand counts that authorize a sale. I also avoid double-checked locking unless the field is volatile; the broken form can publish an object before its constructor's writes are visible.

**Follow-up:** What does the reader do if it needs a price that is missing from the snapshot it loaded?

**Weak answer:** "Put volatile on every field of every Price object and share one HashMap."

## Q9. InterruptedException is swallowed in the gateway client. Why does that matter?

**What the interviewer is probing:** Cancellation and shutdown behavior.

**Sample answer:** An interrupt means a caller asked the thread to stop: a future was cancelled, a pool is shutting down, or a deadline fired. `sleep`, `wait`, and blocking queues throw `InterruptedException` and clear the interrupt status. If I catch it and log it without restoring the status (`Thread.currentThread().interrupt()`) or without aborting, the thread continues the tender as if nobody asked it to stop. The pool cannot shut down promptly. A higher layer's timeout thinks the call was cancelled while the code still charges the card.

I restore the interrupt if I cannot throw, and I let the method fail so the order stays in an unknown or cancelled state that reconciliation understands. I do not convert it to a successful empty receipt. On a `CompletableFuture`, cancel is cooperative; the task must observe the interrupt or check `isCancelled`. A JDBC driver may not abort until the socket timeout. I set that socket timeout so "cancel" has a backstop. In review, an empty catch of `InterruptedException` is a defect, the same way an empty catch of a payment exception is a defect. The principal point is that interrupts are part of the control plane of the service, not a checked inconvenience.

**Follow-up:** How does this interact with a virtual-thread cancellation on Java 21?

**Weak answer:** "InterruptedException is only thrown when the process is killed, so log it and continue."

## Q10. How would you explain a race that only fails in production?

**What the interviewer is probing:** How you reason when a unit test is green.

**Sample answer:** I start from the invariant that broke: two authorizations, a lost line, a counter that went backwards. I identify the shared mutable state and the two operations that should have been one. A unit test that runs on one thread will not catch it. I look at production evidence: two log lines with the same idempotency key and two gateway refs, or a thread dump that shows overlapping calls. I reproduce with a stress test that uses real threads and a barrier so both enter the check before either writes. Flaky tests that `Thread.sleep` are not that test.

Then I name the missing atomicity: unique constraint, `compute`, or a conditional update. I also check publication: a field written by the startup thread and read by request threads without a safe publish. Load makes the window visible; it does not create the bug. I would rather add the constraint and a test that fires two inserts than add a retry that hides the duplicate. In the interview I say what I would not do: sprinkle `synchronized` on the controller and declare it fixed without a dump or a failing test. JMeter can widen the window in a pre-prod environment; it is not a proof by itself if the assertion is only "no HTTP 500."

**Follow-up:** What assertion would you put in that stress test?

**Weak answer:** "Races are theoretical if the code uses Spring, because the container is thread-safe."
