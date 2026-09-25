# Spring Data JPA and Hibernate — interview questions

## Q1. Explain the persistence context while adding a line to an order.

**What the interviewer is probing:** Dirty checking, flush, and what `save` is for.

**Sample answer:** Inside a transaction the `EntityManager` is a unit of work. `find` loads the order and takes a snapshot. Because I call `order.addLine(...)` on that managed instance, I do not need `repository.save` for Hibernate to notice. At flush, before commit, dirty checking compares the snapshot and writes the insert or update. Flush also runs before a query when flush mode is AUTO, so a following query sees the new line. If I return the entity and the transaction commits, the write is already ordered.

Outside the transaction the instance is detached. Mutating it does nothing until a merge, and merging a graph the client built is how over-posting writes a price. I map a command onto the managed aggregate instead. The first-level cache means a second `find` by id in the same context returns the same Java instance and does not hit the database. It does not survive the transaction, and it is not shared across pods. I keep the transaction short so the snapshot and the connection are not held across a payment call. In a principal interview I say this sequence out loud: load, mutate, flush, commit. I do not say "save persists the object" as if the context did not exist. Spring Data's `save` still matters for a new entity and for a detached one, and I know which case I am in.

**Follow-up:** What does `save` do if the entity is detached and the version column is stale?

**Weak answer:** "I call save after every setter or the changes are lost."

## Q2. A screen that lists orders suddenly runs hundreds of queries. What do you do?

**What the interviewer is probing:** N+1 diagnosis and a fix that still paginates.

**Sample answer:** I turn on SQL logging in a lower environment or look at the trace: one select of orders, then one select of lines per order. That is lazy `@OneToMany` touched in a loop, often by Jackson after the service returned, with OSIV keeping the session open so it "works" and hides the cost. I disable reliance on OSIV for the fix. I do not mark the association EAGER, because every future query would load lines, including a status poll that needed one column.

`join fetch` or an entity graph loads the lines for a known set of parents. Combined with page size, a join multiplies rows and Hibernate may apply the limit in memory (the HHH000104 warning). I page parent ids first, then `where orderId in :ids` to load lines, or I use a DTO projection so the list screen never materializes entities. I add an index that matches the list predicate. I assert the query count in a `@DataJpaTest` so the N+1 cannot return quietly. If the spike is in production, the APM's query count on that transaction is the evidence, and the mitigation is a fix or a temporary cap on page size, not a larger connection pool that lets us run more N+1s in parallel.

**Follow-up:** When is a projection a worse choice than an entity graph?

**Weak answer:** "I set everything to EAGER and the problem goes away."

## Q3. Two cashiers sell the last unit. How do you prevent oversell?

**What the interviewer is probing:** Optimistic versus conditional update versus pessimistic lock.

**Sample answer:** Under Read Committed, both transactions can read `on_hand = 1` and both write `0`. That is a lost update. `@Version` makes the second update match zero rows, so one cashier gets `409` and retries. On a hot SKU, retries are constant and optimistic locking is the wrong tool. `SELECT FOR UPDATE` serializes the row and works if the transaction is tiny. It is a disaster if I call the payment gateway while holding the lock.

The pattern I write on the board is one statement: `update inventory set on_hand = on_hand - :qty where sku = :sku and on_hand >= :qty`, then check the update count. Zero means not enough stock. There is no read-modify-write in Java. A reservation row with an expiry covers the case where I must hold stock between scan and pay; the expiry releases it if the device disappears. The unique key on the reservation id makes the reserve idempotent. I do not raise isolation to serializable for the whole service to avoid writing this statement. Oracle and Postgres both do this. The principal close: name the lock duration and the idempotency key, not just the isolation level.

**Follow-up:** How do you index and lock a hot SKU without pinning one row for the whole chain?

**Weak answer:** "I use a synchronized method in the inventory service."

## Q4. Why do you turn spring.jpa.open-in-view off?

**What the interviewer is probing:** Session and connection lifetime.

**Sample answer:** OSIV opens the persistence context for the whole web request. Lazy associations work in the controller and in Jackson, so the code appears correct while it issues queries at render time and holds a Hikari connection until the response is written. That hides N+1 until load, and it couples the view to the schema. I turn it off, accept `LazyInitializationException` as a signal, and load what the endpoint needs inside the service transaction: entity graph or projection. The connection returns to the pool at the end of the service method, before serialization.

The migration is not free. Endpoints that accidentally lazy-loaded will fail, and I would rather fail a test than a Saturday peak. I add slice tests that serialize the DTO without a session. I do not "fix" the exception by annotating the controller with `@Transactional`. That reopens the same long session and can make the controller the transaction boundary, which is how payment calls end up inside it. In a principal interview I say OSIV is a convenience that taxes the pool, and I accept the fetch-plan work as the cost of a service that has a real concurrency budget.

**Follow-up:** What breaks in a test that used to rely on OSIV if the controller returns an entity?

**Weak answer:** "Open-in-view is required for Hibernate to function."

## Q5. How do you map money, enums, and identity in JPA?

**What the interviewer is probing:** Schema mistakes that survive code review.

**Sample answer:** Money is a `long` minor units column or a precise decimal type, never `double`. If I embed a `Money` value type, both amount and currency are mapped, and equality does not use a floating value. Enums use `@Enumerated(EnumType.STRING)`. `ORDINAL` breaks when someone inserts a constant at the top of the file and rewrites every row's meaning. For a status that lives a long time I still prefer a string code the database can check with a constraint.

Identity: a surrogate key is fine, and a business key (`store_id`, `order_number`) has a unique constraint so the surrogate is not the only guard. `equals` and `hashCode` do not use a lazy collection and do not use a null generated id as if it were stable. I implement equality on the business key once it is assigned, or I keep entities out of hash-based sets until they are persistent, and I document which. Bidirectional associations have one owning side, and helper methods on `Order` maintain both sides so a line is not linked in memory and missing a foreign key. `ddl-auto` is `validate` in shared environments. Flyway or Liquibase owns change. I mention one Oracle or Postgres detail: sequence allocation size must match the generator, or we either hammer the sequence or collide.

**Follow-up:** What does `cascade = ALL` on a `@ManyToOne` to Sku do if you delete an order?

**Weak answer:** "Hibernate chooses the SQL types, so the schema is an implementation detail."

## Q6. When do you use the second-level cache, and when Redis?

**What the interviewer is probing:** Cache coherence versus a cache you can explain.

**Sample answer:** The first-level cache is the persistence context and is not optional while the session is open. The second-level cache stores entity or collection state across sessions, per JVM, unless I add a clustered provider and invalidation. It is attractive for a read-mostly reference entity and easy to get wrong for collections and for data that must not be stale. I usually do not put inventory there.

Redis is the shared cache I can draw: cache-aside for the price book, TTL, and an explicit rule that a miss loads from Oracle or Postgres. All pods see the same entry. I can flush a key when pricing publishes a change. I accept staleness up to the TTL for price only if the business accepts it. I do not cache on-hand with a TTL and then decrement the cache as if it were the ledger. The ledger remains the conditional update. Hibernate's second-level cache does not replace that design and it does not span a process crash the way a database does. In the interview I pick Redis for price, database for stock, and I say what I would measure: hit ratio and how often a cashier sees a stale price. I do not enable both caches "for performance" without an owner for invalidation.

**Follow-up:** How do you stop a cache stampede on a hot SKU key?

**Weak answer:** "I turn on the second-level cache for all entities."

## Q7. A transactional service calls a repository and commits partial work. Debug it.

**What the interviewer is probing:** Proxy boundaries, propagation, and caught exceptions.

**Sample answer:** I look at where `@Transactional` actually is. If it sits only on the repository, each `save` commits on its own when the service method is not transactional, so a later failure leaves the earlier line inserted. If it sits on a private or self-invoked method, there is no transaction at all and autocommit mode depends on the connection. If the service method catches a checked exception, the default advice commits. If it catches a runtime exception and does not rethrow or mark rollback-only, it also commits.

`REQUIRES_NEW` on a line-item save commits that line even when the outer method rolls back, which is partial work by design and often accidental. I want one transaction on the application service around the aggregate mutation, and I want domain failures to propagate as runtime exceptions or as checked exceptions listed in `rollbackFor`. I confirm with a test that throws after the second insert and asserts the first insert is gone. Logging SQL with transaction ids, or simply asserting row counts in Testcontainers, is enough. I do not debug this by adding `@Transactional` to every interface in the hierarchy without checking propagation. One boundary, one rollback rule, one test that proves the rollback.

**Follow-up:** How many connections does an outer transaction plus one `REQUIRES_NEW` need from Hikari?

**Weak answer:** "Spring Data methods are always joined to the controller transaction."

## Q8. How do you batch inserts of a large order import?

**What the interviewer is probing:** JDBC batching limits, identity strategies.

**Sample answer:** `saveAll` does not magically batch. I set `hibernate.jdbc.batch_size`, order inserts, and I avoid a generator that disables batching. Identity columns that require the key immediately can force a round trip per insert on some databases; a sequence with an allocation size lets Hibernate assign ids in blocks and batch the inserts. I flush and clear the persistence context every batch window so the first-level cache does not hold the entire file. I keep one transaction per chunk so a bad row does not roll back a million good ones and so a single transaction does not hold locks and undo for the whole import.

I do not do this inside the cashier request path. An import is a job with its own pool budget so it cannot consume every connection the API needs. I would rather bulk-load with a database tool when the file is huge and the domain rules can be applied in a staging table. If domain rules must run per order, I still chunk. I measure statements in the SQL log: one multi-value or batched prepare, not a million single inserts. The principal point is that the persistence context is a unit of work, not a pile, and the id strategy decides whether batching is even possible.

**Follow-up:** What goes wrong if you clear the context while another object still holds a reference to a managed line?

**Weak answer:** "I increase the heap and call saveAll on the whole file."

## Q9. Write the repository query you would accept in review.

**What the interviewer is probing:** Derived queries versus JPQL versus projections.

**Sample answer:** `findByStoreIdAndStatus` is fine. A derived name with five joins and a sort is not; I ask for `@Query` or a specification with a name that a human can read. The query selects a projection record when the caller will not mutate: id, total, status, business date. It fetches an entity when the caller will add a line. I parameterize every predicate. String concatenation in a native query is a SQL injection defect and a SonarQube failure, even if "the input is internal."

Pagination is keyset or a page of ids, not a join fetch plus pageable that warns HHH000104 and then loads the world. I include the store predicate so one store cannot read another's orders by guessing a page. I add the index in the same migration as the query, and a test against Postgres or Oracle in Testcontainers, not only H2, if I used a function H2 treats differently. I lock with `@Lock` or a `@Query` that includes the conditional update only on the inventory repository, with a comment that says why. The review standard is: I can predict the SQL, the index, and the transaction, or the query does not merge.

**Follow-up:** When do you drop to a native query, and what do you lose?

**Weak answer:** "Derived query names are always cleaner than JPQL, however long they get."

## Q10. How do you test a conditional inventory update?

**What the interviewer is probing:** A test that would fail if the where-clause disappeared.

**Sample answer:** I do not mock the `EntityManager` to "verify" a string of SQL I wrote in the test. I run `@DataJpaTest` or a short Spring test against Testcontainers Postgres, or Oracle if that is what production is and the image is allowed. I seed `on_hand = 5`, run the update for 2, assert the count is 1 and the row is 3. I run it again for 4, assert success and 0 remaining would be the next case: from 3, a qty of 4 affects zero rows and leaves 3. Two threads or two sequential transactions that both try to take the last unit assert one winner. The unique constraint on an idempotency key gets a second insert that must fail.

H2 is acceptable for a simple derived query and a liar for `FOR UPDATE`, partial indexes, and some date functions. I tag the Testcontainers test so CI runs it, and I keep the dataset tiny. I assert outcomes, not the exact SQL text, so a rewrite to a bulk statement still passes. I add one failure test that removes the `on_hand >= :qty` predicate in my mind: if someone deletes it, the second assertion (`on_hand` would go negative) fails. That is the test I describe in the interview. A Mockito test of the service still fakes the port and checks that a zero update count becomes `SKU_NOT_ON_HAND` for the HTTP layer.

**Follow-up:** How do you keep Testcontainers fast enough for every pull request?

**Weak answer:** "I verify that the repository method was called. The SQL is Hibernate's problem."
