# Oracle interview Q&A

Ten questions a principal loop can ask. Sample answers are the shape of a strong reply, not a script to memorize word for word.

## 1. What lives in the SGA versus the PGA, and why should a service owner care?

**Interviewer intent.** Check that memory architecture is tied to latency and pooling, not recited as a diagram.

**Strong sample answer.** The SGA is shared: buffer cache for blocks, shared pool for cursors and plans, redo buffer for change vectors. The PGA is private to each server process and holds sorts, hashes, and session state. I care because hard parses and literal SQL burn the shared pool, a bad plan spills in the PGA and temp, and every pooled connection is a server process with its own PGA. Commit waits on LGWR flushing redo, so checkout latency is also storage and redo, not only Java. If we double pod count we can hit the process limit before CPU.

**Follow-up.** What do you do when you see `ORA-01555`? The query's consistent read outlived undo. Shorten the query or size undo retention to the real report. Do not hammer retry from the client.

**Weak answer to avoid.** "SGA is System Global Area and PGA is Program Global Area" and then stopping. Or claiming you should put business pricing logic in the shared pool.

## 2. Why do readers not block writers in Oracle, and when does that stop being true?

**Interviewer intent.** Read consistency versus locking, and whether you have been burned by the exceptions.

**Strong sample answer.** Default read committed is statement-level. The server rebuilds blocks as of the statement SCN from undo, so a normal `SELECT` does not take a lock that blocks DML, and DML does not block that `SELECT`. It stops being true for `SELECT FOR UPDATE`, for a missing index on a foreign-key child that forces parent-key locking, and for bitmap indexes, which lock bitmap pieces and can serialize unrelated rows. Serializable isolation can also fail the transaction with `ORA-08177` instead of blocking.

**Follow-up.** Would you run the catalog extract on the primary? Only if it is short and indexed. A long extract belongs on a replica or a reporting copy so undo and buffer cache stay with checkout.

**Weak answer to avoid.** "Oracle locks the whole table on every update" or "MVCC means we never need transaction boundaries."

## 3. When is a bitmap index the wrong index?

**Interviewer intent.** Cardinality versus concurrency. A common trap is "low cardinality, so bitmap."

**Strong sample answer.** Bitmap fits low-cardinality columns in a warehouse load, where DML is batched. On OLTP, DML locks a range of rowids in the bitmap, so concurrent checkout updates collide even on different rows. Order status, store id, or channel on a live order table should be a B-tree if it is selective enough to seek, or it should not be indexed that way at all. I would also question a B-tree nobody's predicate can enter, but the bitmap failure mode is locking, not just a bad plan.

**Follow-up.** Reverse-key index? It spreads a sequence-like insert hotspot and kills range scans. I would partition or change the key before I reverse it.

**Weak answer to avoid.** "Bitmap is always faster for flags" with no mention of DML locking.

## 4. Bind variables versus literals in a POS service?

**Interviewer intent.** Cursor sharing, and whether you understand skew.

**Strong sample answer.** Bind. The library cache should hold one cursor for "order by id" regardless of which store is calling. Concatenating SKU or store id into SQL is how a sale event hard-parses the shared pool. I allow literals only when a value is so skewed that one plan is wrong for both, and even then I look at adaptive cursor sharing and histograms before I give up on binds. Evidence is `DISPLAY_CURSOR` actual rows versus estimates, not a habit.

**Follow-up.** How do you confirm a bad plan in production? `DBMS_XPLAN.DISPLAY_CURSOR` with last-run statistics, under the bind that the app uses, not a literal you typed in a console.

**Weak answer to avoid.** "Always use literals so the optimizer knows the value" as a blanket rule for OLTP.

## 5. How should Hibernate use Oracle sequences?

**Interviewer intent.** Duplicate keys and pool allocation. This shows up the moment there is more than one app node.

**Strong sample answer.** A named sequence, pooled optimizer, `allocationSize` equal to `INCREMENT BY`. If the sequence increments by 1 and Hibernate assumes 50, nodes will collide or leave huge gaps that still collide depending on the optimizer. Sequence cache gaps after a crash are acceptable for surrogate keys. They are not acceptable for a customer-visible gap-free invoice number, and that requirement is a hotspot I would challenge. I avoid identity if we need JDBC batching, because the key fetch serializes the insert.

**Follow-up.** What about `CACHE` on the sequence? It reduces dictionary contention. Gaps are fine. Order of allocation across RAC nodes is not a business order unless we asked for `ORDER`, which costs more.

**Weak answer to avoid.** "Sequences never gap" or "we use `new Date()` plus random for primary keys."

## 6. How do you read an explain plan on a slow order query?

**Interviewer intent.** You can navigate a plan and you know estimates lie.

**Strong sample answer.** I start from the driving access path. I want to see whether we range-scan an index that matches the predicate or full-scan a table, and whether the join method matches the cardinality. Then I compare estimated rows to actuals. A nested loop over a bad estimate, or a hash join that spills, is a stats or predicate problem. A full scan of one pruned partition can be the right plan. I check that the partition key is in the predicate if the table is partitioned. I do not add indexes until the plan says which predicate is selective.

**Follow-up.** Stale stats after a load? The plan can flip under you. Gather stats as part of the load, or lock stats only if you have a reason and a test.

**Weak answer to avoid.** "Full table scan is always bad, so I add an index on every column."

## 7. What is your rule for PL/SQL in a microservice estate?

**Interviewer intent.** Whether logic will be buried where it cannot be traced or scaled.

**Strong sample answer.** Constraints and a short procedure when the invariant must commit with the data: unique payment attempt, reservation plus outbox. Not a pricing engine, not plan-compatibility rules that change every promotion. Those stay in services we can test and see in New Relic. Set-based SQL for maintenance; cursor loops only when the work is genuinely row-wise, and then bulk bind. The database is an API of constraints, not a second application server.

**Follow-up.** Trigger for audit? A trigger that writes an audit row can be right. A trigger that calls another system or updates three aggregates is an incident.

**Weak answer to avoid.** "All business logic in packages so we do not depend on Java" or "no logic in the database at all, not even unique constraints."

## 8. How do you partition an orders table, and what does it not fix?

**Interviewer intent.** Lifecycle and pruning, not partitioning as a slogan.

**Strong sample answer.** Range on order time if retention and pruning are by age. The query must include the partition key or we scan every partition. Local indexes for the common access path. Dropping or exchanging a month beats a delete that generates undo. Partitioning does not make a missing SKU predicate fast, and global indexes need maintenance across the drop. Hash partitioning spreads a hot key and then you cannot prune by range. I would not partition a small table to look enterprise.

**Follow-up.** A query by order id only? That is a global index or a design that includes time in the lookup. I would rather keep the id lookup as a unique index the optimizer can use than force every reader to know the month.

**Weak answer to avoid.** "Partition every table by default."

## 9. N+1 and locking with Hibernate on a cart.

**Interviewer intent.** Practical ORM failure, and lock scope across payment.

**Strong sample answer.** N+1 is loading the parent then initializing each collection. I fix the use case with a join fetch or an entity graph, or `@BatchSize`, and I do not mark every association eager. Cart line updates use a version column so a stale write fails and the client reloads. Inventory decrement can take a pessimistic lock for a few statements. I will not hold that lock, or an open persistence context, across the payment HTTP call. The connection pool would stall on a popular SKU.

**Follow-up.** Open session in view? It hides lazy-load bugs and holds connections for the whole render. Map a DTO inside the transaction.

**Weak answer to avoid.** "We set everything to EAGER and the N+1 went away."

## 10. Why might a new service use Postgres instead of the existing Oracle?

**Interviewer intent.** Judgment, not loyalty to one engine. You should not dual-write casually.

**Strong sample answer.** If the data and the PL/SQL already live in Oracle and the invariant is there, I keep the write in Oracle. A new service with relational OLTP and some JSON, no existing packages, and a cloud operational model fits Postgres: no per-core license, JSONB, managed failover. I do not run both as the system of record for the same order. One writer, and a projection if we need the other shape. The MVCC difference I actually operate is undo versus heap versions and vacuum.

**Follow-up.** How would you move a slice? Expand a new owner, backfill, cut reads, cut writes, reconcile counts. Not a weekend of dual-write with no check.

**Weak answer to avoid.** "Postgres is always better" or "we write both and they will stay in sync."
