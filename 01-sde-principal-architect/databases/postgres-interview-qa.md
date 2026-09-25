# PostgreSQL interview Q&A

## 1. How does Postgres MVCC differ from "update in place," and what does that cost you?

**Interviewer intent.** Dead tuples, not a definition of ACID.

**Strong sample answer.** An update writes a new tuple and leaves the old one dead for later snapshots. `xmin` and `xmax` plus the snapshot decide visibility. Indexes point at tuple versions, so indexed updates cost index maintenance too. The cost is bloat until vacuum reclaims space, and a long-running transaction holds back that cleanup. Readers do not block writers for ordinary snapshots. Writers still block other writers on the same row.

**Follow-up.** Heap-only tuples? If no indexed column changes and fillfactor leaves room, Postgres can avoid an index update. I would set that only on a table we measured as update-heavy.

**Weak answer to avoid.** "Postgres locks the row for readers the same way a spreadsheet does" or ignoring vacuum entirely.

## 2. Which isolation level do you use for checkout, and what can still go wrong?

**Interviewer intent.** Default behavior and retryable serialization failures.

**Strong sample answer.** Read committed for short transactions, with the invariant in a conditional update or a unique constraint: reserve only if quantity remains, insert order with a unique idempotency key. Repeatable read holds one snapshot. Serializable will abort one of two conflicting transactions and the application must retry. I do not hold any isolation level across a payment HTTP call. Lost update on a cart is handled with a version column, not by hoping the isolation level notices a read-modify-write in the app.

**Follow-up.** Phantom rows? A unique constraint is the reliable guard for "this payment key exists once." Isolation level alone is the wrong tool if the app reads, decides in Java, and writes later.

**Weak answer to avoid.** "We use serializable everywhere so we do not need constraints."

## 3. When do you choose GIN or GiST over a btree?

**Interviewer intent.** Operator and workload match.

**Strong sample answer.** Btree for equality and range on scalars: SKU, order id, time. GIN when the predicate is containment inside JSONB, an array, or full text. GIN is more expensive on write, so I do not put it on a hot checkout row for a field I could have extracted into a column. GiST for ranges and exclusion constraints, for example two promotions that must not overlap for the same SKU and interval. BRIN only for large, physically ordered append-only data.

**Follow-up.** You always filter `payload->>'sku'`? Promote `sku` to a column with a btree. JSONB remains the long tail.

**Weak answer to avoid.** "GIN is the JSON index, so I add it and then use whatever operator I like."

## 4. What do you look at in `EXPLAIN (ANALYZE, BUFFERS)`?

**Interviewer intent.** Actuals versus estimates, and buffer reality.

**Strong sample answer.** Estimated rows against actual rows first. A large gap means stats, correlation, or a black-box function. Then time per node, and shared hits versus reads. An index-only scan that still heap-fetches is a visibility-map problem, which is a vacuum problem. External merge or hash batches mean `work_mem` or, more often, a plan that should not be sorting that much. I will not `ANALYZE` a destructive statement on production to satisfy curiosity. JIT time dominating a point lookup means JIT should be off for that role.

**Follow-up.** A sequential scan on a small table? Leave it. The planner is allowed to be right.

**Weak answer to avoid.** "I only look at whether it says Seq Scan and then I add an index."

## 5. Autovacuum is running. Why is the table still in trouble?

**Interviewer intent.** Long transactions, wraparound, and per-table settings.

**Strong sample answer.** A snapshot held by an idle transaction, a migration session, or a report on the primary prevents removal of dead tuples. A high-churn cart table needs a tighter autovacuum scale factor than the default. Vacuum reclaims space for reuse; it does not always shrink the file, so disk can stay high after a cleanup. Separately I watch xid freeze age. If freezing falls behind, Postgres will stop writes to protect wraparound. That is an incident, not a tuning nicety.

**Follow-up.** `VACUUM FULL`? It rewrites and locks heavily. I would rather `pg_repack` in a planned window, and I would rather not need either.

**Weak answer to avoid.** "Turn autovacuum off because it uses CPU."

## 6. Why put PgBouncer in front, and what breaks?

**Interviewer intent.** Connection multiplication from pods, and transaction pooling limits.

**Strong sample answer.** Each Postgres connection is a backend with memory and snapshot cost. `pods * pool_size` will exhaust the database before CPU does. PgBouncer in transaction mode checks out a server connection only for a transaction, which is what a stateless JDBC service usually needs. It breaks session `SET`s, temp tables, advisory locks held outside a transaction, and prepared statements that outlive the transaction unless the pooler is configured for them. I keep the app pool small, set a real backend budget, and put timeouts on both. RDS Proxy is the same tradeoff if we are on RDS and do not want to run the pooler.

**Follow-up.** Session mode? It fixes session features and does not reduce backends. I would not choose it as the scale plan.

**Weak answer to avoid.** "We set Hikari max to 200 per pod so we have headroom."

## 7. How do you migrate a live catalog table?

**Interviewer intent.** Expand/contract and lock levels.

**Strong sample answer.** Expand then contract. Add a nullable column or a new table, deploy code that writes both and reads the new shape with a fallback, backfill in batches, then enforce the constraint, then delete the old path in a later release. `CREATE INDEX CONCURRENTLY` cannot run inside a transaction block. I state the lock in the review: a rewrite `ALTER` is a window, not a surprise. Two app versions run during rollout, so the database must be valid for both.

**Follow-up.** A `NOT NULL` on a big table with a default? Know the version behavior. On versions that still rewrite the table, I add nullable, backfill, then set not-null.

**Weak answer to avoid.** "Flyway runs it in one transaction in production and we hope the lock is short."

## 8. Where does JSONB help a plan catalog, and where does it hurt?

**Interviewer intent.** Discipline about predicates.

**Strong sample answer.** Plan configuration that varies by product line can sit in JSONB beside real columns for id, price, status, and foreign keys. I index the containment operators I actually run. If every request filters a key, that key becomes a column. JSONB does not remove migrations, and a GIN index on a row we update constantly will dominate write time. I do not store the allocatable inventory count in a JSON blob.

**Follow-up.** Constraints? A check on a column, or an exclusion constraint on a range, still beats a JSON key nobody can see in the catalog.

**Weak answer to avoid.** "We will store the whole order as JSON so we never migrate."

## 9. How do you lock a work queue without blocking the world?

**Interviewer intent.** `SKIP LOCKED` and the limits of advisory locks.

**Strong sample answer.** Outbox or job rows: `SELECT ... FOR UPDATE SKIP LOCKED` inside a short transaction, with a lease column so a crashed worker's row becomes visible again. That lets multiple workers claim different rows. Advisory locks fit a singleton scheduler. I do not hold either lock across HTTP. Inventory correctness is a conditional update in the row, not an advisory lock around a remote call.

**Follow-up.** Deadlock? Two transactions update the same rows in opposite order. Fix the order, keep transactions short, and retry a deadlock. It is not the same error as a serialization failure, but both can be retried if the transaction is idempotent.

**Weak answer to avoid.** "We `LOCK TABLE` the outbox every poll."

## 10. When does Postgres beat Oracle for you?

**Interviewer intent.** A decision rule, not a brand preference.

**Strong sample answer.** New service, relational invariants, some semi-structured attributes, team can operate vacuum and pooling, and we want managed multi-AZ without an Oracle license decision. I stay on Oracle when the schema, packages, and DBAs already own that data and a rewrite is the larger risk. I will not split one order across both. Replicas are for read scale and lag; checkout read-your-writes stays on the primary.

**Follow-up.** Logical replication? Fan-out and migrations. Not multi-master checkout.

**Weak answer to avoid.** "Postgres does not need tuning" or "they are identical, so pick either and dual-write."
