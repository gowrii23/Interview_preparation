# Redis interview Q&A

## 1. Which structure do you use for a device page, a cart counter, and a top-ten list?

**Interviewer intent.** Structure follows the command, not habit.

**Strong sample answer.** Device page: a string, JSON, with a TTL, because we get and set the whole document. A field we increment, such as a session's item count, can be a hash so we do not rewrite the blob. Top-ten devices in a store: a sorted set scored by a count, trimmed to a window. I would not pull a huge hash into the service and sort it in Java. Sets fit bounded membership, such as a small compatibility list, and they stop being a good idea when the set grows without a cap. Streams if we truly need consumer groups; otherwise a real queue owns durable work.

**Follow-up.** Why not a list for the top ten? A list does not rank by score cheaply. We would be rebuilding it.

**Weak answer to avoid.** "We put everything in strings because Redis is a cache."

## 2. Cache-aside versus write-through for price display?

**Interviewer intent.** Source of truth and race on invalidation.

**Strong sample answer.** Cache-aside. Database is the price of record. The service reads Redis, and on a miss reads the database and sets a TTL. Publish deletes the key after the database commit, or bumps a version in the key. I do not write-through the display cache if that puts Redis on the critical write of merchandising. I do not write-behind price at all. Checkout reprices from the primary so a stale display cannot charge the wrong amount. The race where a reader repopulates an old value between commit and delete is bounded by TTL or by a versioned key.

**Follow-up.** Negative caching? A missing SKU should be cached briefly so a bad client cannot stampede the database. The TTL is short so a newly created SKU appears.

**Weak answer to avoid.** "We update the cache and the database and assume they match."

## 3. How do you stop a stampede when a hero device key expires?

**Interviewer intent.** Cross-pod single flight, not a local `synchronized`.

**Strong sample answer.** One rebuild, not one per pod. A short Redis lock so a single populator refills the key, plus a stale copy we can serve while that happens, or probabilistic early refresh before the TTL. A JVM lock only collapses traffic inside one process, and we have many. The database still has to survive a miss storm if the lock fails, so I size that fallback and I test it by expiring the key under load. JMeter against a warm cache will not show this.

**Follow-up.** What if the lock holder dies? The lock has a TTL, so it expires. The next caller rebuilds. That is why the lock is not correctness for inventory.

**Weak answer to avoid.** "We set a long TTL so it never expires during the day."

## 4. Why is a Redis lock not enough to prevent oversell?

**Interviewer intent.** Redlock critique and fencing, at a practical level.

**Strong sample answer.** `SET NX PX` with a token, released by compare-and-delete, is a lease. Pauses, clock jumps, and failover can let two holders believe they own it. Redlock does not remove that class of bug. Oversell is a conditional update in the transactional database, or a reservation insert with a unique constraint. If I use Redis to shed duplicate refreshers, the token does not authorize the stock change. I never hold the lock across a payment call, because the TTL will expire under a live request.

**Follow-up.** Fencing? The system of record stores a monotonic token and rejects a late writer with an older one. Without that check, the lock is advisory.

**Weak answer to avoid.** "Redlock makes it strongly consistent, so we decrement stock in Redis."

## 5. RDB versus AOF for a session cluster?

**Interviewer intent.** Loss window, and cache versus durable store.

**Strong sample answer.** If the session can be rebuilt or the user can sign in again, I might run with persistence off or with infrequent RDB and accept a cold start. If session loss is a real incident, AOF with `appendfsync everysec` caps the loss near a second and costs less latency than fsync on every write. RDB alone loses everything since the snapshot. Neither makes Redis the order ledger. Replication is asynchronous, so a promoted replica can miss the last writes. I still keep the order in Postgres or Oracle.

**Follow-up.** AOF rewrite? It compacts in the background. I watch disk and rewrite duration so it does not coincide with a traffic spike we already cannot afford.

**Weak answer to avoid.** "AOF means we cannot lose data, so payment state can live only in Redis."

## 6. How do hash slots change key design?

**Interviewer intent.** Cluster mode and hot keys.

**Strong sample answer.** Cluster has 16384 slots. The key, or the `{tag}` inside it, picks the slot. Multi-key commands must share a tag or they fail with a cross-slot error. I tag by a real locality key, such as `{cartId}`, not by a constant that piles the world onto one node. A hero SKU as a single key still hot-spots one shard, because slots do not split one key. Clients must honor `MOVED` and `ASK`. I use a cluster-aware Spring client in every environment, not only in production.

**Follow-up.** Logical databases as isolation? They share one thread and one memory budget. Different failure domains get different clusters.

**Weak answer to avoid.** "We use Redis transactions across the cart key and the inventory key on a cluster."

## 7. Spring Cache or RedisTemplate?

**Interviewer intent.** Abstraction leak.

**Strong sample answer.** `@Cacheable` for cache-aside on a method whose inputs form a stable key. I include every argument that changes the result, I set TTL in `RedisCacheConfiguration`, and I choose JSON serialization so an incident is readable. `sync=true` only single-flights inside that process. `RedisTemplate` or `StringRedisTemplate` when I need a sorted set, a lock, a stream, or an explicit delete. I do not hide those behind the cache annotation. Timeouts are set. A slow Redis call must not pin the servlet thread until the pool is exhausted.

**Follow-up.** Caching null? For a stampede of unknown SKUs, yes, with a short TTL. For a value that can legitimately appear a moment later, the TTL has to be short enough for the business.

**Weak answer to avoid.** "Java serialization is fine because it is the default."

## 8. How do you evict without deleting the lock keys?

**Interviewer intent.** Policy and workload split.

**Strong sample answer.** `allkeys-lru` will evict a key with no TTL, including a lock, if memory is full. Catalog uses `volatile-lru` or `allkeys-lru` on a cluster that holds only cache entries, all with TTLs. Locks and queues, if they exist, sit on a separate instance with `noeviction` so a full memory condition errors visibly. I alarm on evictions and on memory. A national launch that fills the cache should drop cold device pages, not the session of the person checking out, which is why those workloads are split.

**Follow-up.** `maxmemory` policy change in production? It is a behavior change, not a harmless toggle. I would rather size and split first.

**Weak answer to avoid.** "Eviction means least recently used, so important keys are safe."

## 9. What do you invalidate when a plan matrix is republished?

**Interviewer intent.** Graphs and partial deletes.

**Strong sample answer.** A compatibility matrix is a graph. Deleting one SKU's key and hoping is how stale edges survive. I put the publish version in the key and treat the new version as a new namespace. Old keys expire by TTL. The database commit happens first. Checkout does not trust the cached matrix for the price or the reservation. Display can be stale within the version window. If the matrix is small, the version can be one key the service reads before composing other keys.

**Follow-up.** Who deletes? The publisher, after commit, or the version bump makes delete optional. I do not let every channel delete keys ad hoc.

**Weak answer to avoid.** "We flush the whole database on every price change."

## 10. Session and cart: what is Redis allowed to own?

**Interviewer intent.** Ownership, PCI hygiene, loss on failover.

**Strong sample answer.** Opaque server-side session id, TTL refreshed on activity, no card data, no full payment instrument. Anonymous cart can live in Redis if losing it on a failover is acceptable before checkout. At login or submit, the relational cart or order becomes the owner. I do not let the store channel and the digital channel both write the same key without a version. Rate limits can live here and can fail open or closed as a product decision, stated explicitly.

**Follow-up.** Cookie contents? An opaque id. The session payload stays on the server.

**Weak answer to avoid.** "We cache the authorized payment result in Redis and skip the provider on retry" without an idempotency design in the system of record.
