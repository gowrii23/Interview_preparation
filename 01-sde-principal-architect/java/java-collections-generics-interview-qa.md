# Collections and generics — interview questions

## Q1. Walk through HashMap get and put on Java 8 or 17.

**What the interviewer is probing:** Cost model, tree bins, and resize behavior, not "it's O(1)."

**Sample answer:** A `HashMap` is a table whose length is a power of two. The index is the key's hash, mixed with `h ^ (h >>> 16)`, then masked with `length - 1`. That mix stops a hash that only varies in the high bits from stacking in one bin. An empty bin stores the node directly. A short collision chain is a linked list. When a bin reaches length 8 and the table length is at least 64, the bin becomes a red-black tree. If the table is still small, the map resizes instead of treeifying, because the problem is spread, not a true hot bin. Untreeify happens around length 6.

Put that exceeds `capacity * loadFactor` (default 0.75) doubles the table and rehashes nodes. `get` is amortized constant time when the hash spreads, and degrades if `hashCode` is constant. One null key is allowed. `HashMap` is not thread-safe; two puts can lose entries or loop a chain on older JDKs. I do not share one across cashier requests. For a price-book key I care that `equals` and `hashCode` use the same fields and that the key is immutable. I would mention tree bins in a principal interview because "buckets" alone sounds like Java 7.

**Follow-up:** Why not set the load factor to 1.0 to save memory?

**Weak answer:** "HashMap is an array of linked lists and lookup is always O(1)."

## Q2. ConcurrentHashMap versus Collections.synchronizedMap for an in-process idempotency table.

**What the interviewer is probing:** Locking granularity and compound actions.

**Sample answer:** `synchronizedMap` locks the entire map on every operation. Check-then-act (`containsKey` then `put`) is still a race unless the caller holds the same mutex around both calls. Iteration also needs that mutex. `ConcurrentHashMap` does not lock the whole table. A new bin is installed with CAS. An update of an existing bin synchronizes on that bin's head node. Readers use volatile reads and do not block writers. `putIfAbsent`, `compute`, and `merge` are the atomic operations I actually want: record a tender key only if absent, and do not replace a stored receipt.

I keep the mapping function short and free of HTTP or JDBC. The bin lock is held for the duration of `compute`, and calling out to the payment gateway there stalls every other update in that bin and risks deadlock if the function touches the same map. Iterators are weakly consistent and do not throw `ConcurrentModificationException`. Null keys and values are forbidden. `size` is a striped counter, not a locked snapshot.

This structure is one JVM. Two POS pods still need a unique constraint in Oracle or Postgres. I say that before I finish the answer, so the interviewer does not think the CHM is the system of record.

**Follow-up:** What does weakly consistent iteration guarantee during a resize?

**Weak answer:** "ConcurrentHashMap is fully lock-free and synchronizedMap is deprecated."

## Q3. ArrayList or LinkedList for order lines?

**What the interviewer is probing:** Memory layout and why the textbook answer is usually wrong.

**Sample answer:** I use `ArrayList`. A cart has tens of lines, not millions. Index access is constant time, insertion at the end is amortized constant time, and the lines sit in a contiguous array of references, which is friendly to the CPU cache. `LinkedList` uses a node object per line, with a header and two pointers, so it costs more memory and scatters across the heap. Reaching index i is linear. The only operational advantage is splicing at an iterator you already hold, which a cart UI does not need.

I size the list if I know the line count, to avoid the 1.5x copy. I do not return the live list from the aggregate; I return an unmodifiable copy so a controller cannot `clear` the order. `Arrays.asList` is fixed-size and backed by the array; `List.of` is immutable and rejects nulls. I have seen a defect where a signature took `List` and a caller passed `List.of`, then the domain called `add` and threw. The API should document mutability. `ArrayDeque` is the queue or stack I would pick, not `Stack` and not `LinkedList`. For a principal role I also mention fail-fast iterators: structural changes during for-each throw `ConcurrentModificationException`, and that check is not a lock.

**Follow-up:** How does `removeIf` avoid the fail-fast problem?

**Weak answer:** "LinkedList is faster for inserts, so I use it for carts."

## Q4. Explain PECS with a pricing API.

**What the interviewer is probing:** Whether wildcards are something you can apply, not recite.

**Sample answer:** PECS means producer-extends, consumer-super. A method that reads prices from a list and does not add to it takes `List<? extends LinePriced>`. The list produces items that are at least `LinePriced`. The compiler rejects `add` because the actual element type might be a subtype, and a plain `LinePriced` would not fit. A method that only writes an `OrderLine` into a sink takes `List<? super OrderLine>`. I can `add` an `OrderLine`. When I `get`, the static type is `Object`, because the list might be `List<Object>`.

I put wildcards on input parameters, not on return types, unless the caller is genuinely not supposed to know the exact type. Returning `List<? extends Price>` sticks the wildcard to every caller. Invariants: generics are invariant, so `List<OrderLine>` is not a `List<Object>`, unlike arrays, which are covariant and throw `ArrayStoreException`. I use a bound such as `<T extends Price & Comparable<? super T>>` only when the method both uses price behavior and sorts. Heap pollution from a raw list is a review comment I treat as a defect, because the `ClassCastException` appears far from the bad `add`.

**Follow-up:** Why is `new T()` illegal inside a generic class?

**Weak answer:** "I use `List<Object>` and cast when I need it."

## Q5. Your local price cache grows until the pod OOMs. What do you change?

**What the interviewer is probing:** Eviction, scope, and the difference between a map and a cache.

**Sample answer:** An unbounded `HashMap` held by a singleton is a leak with a respectable name. I first confirm it with a heap dominator: one map retaining price entries keyed by SKU and store. Then I stop treating it as the source of truth. The price book lives in Oracle or Postgres, and a shared Redis cache with a TTL serves the pods. Inside one JVM I only keep a bounded cache: `LinkedHashMap` in access order with `removeEldestEntry`, or Caffeine if we already depend on it, with a maximum weight and a TTL. I do not synchronize a `LinkedHashMap` by hand if multiple request threads share it; I use a concurrent cache.

I also check key cardinality. Caching per store, per SKU, per customer segment can exceed the catalog. The key design is part of the fix. Eviction policy is LRU for a hot seller and a poor fit if the working set is larger than the cap; then the cache thrashes and I would rather hit Redis. I set a metric on hit ratio and size. I do not "fix" this by raising `-Xmx`. Inventory counts do not go in this cache with a long TTL. Price can be slightly stale by policy; on-hand cannot.

**Follow-up:** How do you prevent a stampede when the TTL expires on a hot SKU?

**Weak answer:** "I would make the map static and synchronized."

## Q6. What is type erasure, and where does it leak?

**What the interviewer is probing:** Practical limits of generics in APIs and reflection.

**Sample answer:** The compiler checks generic types and then erases them to the bound, usually `Object`, or inserts casts at use sites. At runtime there is one `List` class. That is why `instanceof List<String>` does not compile, why you cannot allocate `new T()`, and why a reflective framework sees a raw list unless it also reads generic signatures from methods and fields via `ParameterizedType`. Those signatures exist on classes, methods, and fields, which is how Jackson and Spring bind `List<OrderLine>`. They do not exist on a local variable after erasure, and they are lost on a raw type.

Arrays are reifiable, so `String[]` knows its component type and a bad store throws `ArrayStoreException`. That is also why you cannot cleanly allocate a generic array. Overloads that differ only by erasure (`void m(List<String>)` and `void m(List<Integer>)`) do not compile. I have seen a bug where a raw `List` was stuffed with a `String` and a later cast to `Sku` failed in the pricing loop, far from the insert. SonarQube's unchecked warnings on the order module are a quality-gate item for me. I do not suppress them on a public API.

**Follow-up:** How does Spring read the generic type of a `RestTemplate` exchange call?

**Weak answer:** "Erasure means generics are optional and only for documentation."

## Q7. How would you count tenders by type for a batch of orders?

**What the interviewer is probing:** Choice of map, collector, and enum structures.

**Sample answer:** If the set of tender types is a closed enum, I use `EnumMap`. It is an array indexed by ordinal, dense and iteration-stable, and it rejects a null key. For a one-off aggregation over a list I already hold, `Collectors.groupingBy(Tender::type, () -> new EnumMap<>(TenderType.class), counting())` states the intent. I do not pull a million orders into a list to do this; that aggregation belongs in SQL (`group by tender_type`) where the database can use an index. The in-memory collector is for a cart or a page of results.

`TreeMap` is justified when I need a range, such as totals by business date, not for an enum. `HashMap` is fine for an open string key and wastes the enum's ordinal structure. I avoid `Hashtable`. If this runs on request threads and updates a shared map, I use `ConcurrentHashMap.merge` or I aggregate locally and publish an immutable snapshot. A parallel stream into a non-concurrent map is a race. I also define what happens with an unknown type coming from an old payload: fail the batch record, do not drop it into a null key.

**Follow-up:** When would `groupingByConcurrent` be the wrong collector even on a parallel stream?

**Weak answer:** "I always use HashMap and parallelStream for speed."

## Q8. A ConcurrentModificationException appears while building a receipt. What do you check?

**What the interviewer is probing:** Fail-fast versus concurrent modification, and a real debug path.

**Sample answer:** `ConcurrentModificationException` from an `ArrayList` or `HashMap` iterator means `modCount` changed during iteration. It is often single-threaded: a for-each that calls `lines.remove(line)` or that adds a fee line inside the loop. The fix is `removeIf`, an explicit iterator's `remove`, or collecting the changes and applying them after the loop. I do not catch the exception and continue; the iterator is invalid.

If the stack is on a request thread and another thread mutates a shared list, the exception is a race detector, not a lock. It can also fail to throw. The fix is not to share a mutable cart, or to mutate a concurrent structure whose iterators are weakly consistent and will not throw this exception. I look for a singleton bean holding a list, a static cache, or a parallel stream sharing an `ArrayList`. Logging the line list with a side effect inside `map` can modify it too.

I add a unit test that mutates during iteration the way the receipt builder did, so the regression is local. If the production log shows it only under load, I still fix the shared mutation; I do not blame JMeter.

**Follow-up:** Why can this exception be absent even when a race exists?

**Weak answer:** "It means the JVM ran out of threads."

## Q9. Design a generic repository method that does not paint you into a corner.

**What the interviewer is probing:** API design with bounds, wildcards, and honesty about Spring Data.

**Sample answer:** I keep persistence out of the domain. A generic method I would write is `<T> T require(Optional<T> found, String id)` or a small `<ID, E extends Identified<ID>> E load(Class<E> type, ID id)` at the edge of a hand-rolled store. The bound is the behavior I use. I do not declare `<T extends Object>` as decoration. If the method only consumes entities to write them, the parameter is `Collection<? extends E>` so a `List<OrderLine>` can be passed to a writer of a supertype. If the method produces them into a caller-owned list, the parameter is `List<? super E>`.

I return a concrete type or `Optional<E>`, not `Optional<? extends E>`, unless I am forwarding an unknown producer. I avoid raw types at the boundary. In a Spring Data service I usually do not invent a generic DAO at all. `JpaRepository<Order, String>` is already that abstraction, and a second generic layer hides queries. I drop to `@Query` or a projection record when the derived name gets long. The principal point is that generics express producer and consumer roles, and they are not an excuse to build a framework inside one order service.

**Follow-up:** What breaks if this method catches `Exception` and returns null?

**Weak answer:** "Make every method `<T> T` and cast inside."

## Q10. How do you choose among HashMap, TreeMap, and EnumMap in a design review?

**What the interviewer is probing:** A decision you can repeat under time pressure.

**Sample answer:** I ask what operation is hot and what the key is. Equality lookup on an arbitrary immutable key is `HashMap`, with an explicit capacity if the size is known, and with the understanding that iteration order is undefined. If two requests must see a stable order, I use `LinkedHashMap` or I sort at the edge. A range query, a nearest price band, or a chronological map is `TreeMap`, O(log n), and the comparator must match equality or keys disappear. A closed enum key is `EnumMap`. I do not use `ConcurrentHashMap` unless the map is actually shared across threads; the extra complexity and the ban on nulls surprise callers who were copying a request-local structure.

I reject a map used as a disguised domain object (`Map<String, Object>` for an order). That pushes every invariant to string keys and shows up as a `ClassCastException` in production. I also reject synchronization wrapped around a `HashMap` when the real need is cross-pod consistency; that is a database constraint. In review comments I ask for the eviction story if the map outlives the request. Those three questions—key type, sharing, lifetime—catch almost every collection defect I see on a POS service.

**Follow-up:** When is a sorted array and binary search a better price-band structure than a TreeMap?

**Weak answer:** "TreeMap is faster than HashMap because it is sorted."
