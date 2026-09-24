# Java 8–17 — interview questions

## Q1. When do you refuse a stream on the order path?

**What the interviewer is probing:** Judgment about laziness, exceptions, and parallel streams.

**Sample answer:** I use a stream when a pipeline of filter and map makes the transformation clearer and the data is already in memory: summing line extensions with `mapToLong` so I do not box, or grouping tenders. I refuse it when the body calls JPA or HTTP, when I need a checked exception to propagate cleanly, or when the logic is one conditional that a `for` loop shows more honestly to the next reader. Intermediate operations are lazy and may be skipped or fused. A side effect inside `map` is a bug, and `parallel()` makes the bug worse because order and thread are no longer the caller's.

`parallel()` on the common `ForkJoinPool` can stall other parallel work in the JVM. I will not parallelize a 20-line cart. I might parallelize a CPU-bound catalog reprice with an explicit pool and a proven gain. `peek` is not a business step. `orElse` on an `Optional` is a different trap, but the same theme: know what runs eagerly. I also refuse a stream that mutates the list it is reading. The principal line is that streams are a readability and fusion tool, not a performance default and not a place to hide I/O.

**Follow-up:** How do you unit test a collector that groups tenders by type?

**Weak answer:** "Streams are always faster and more modern, so I convert every loop."

## Q2. How should Optional be used in a Spring service?

**What the interviewer is probing:** API taste and the NPE modes Optional does not fix.

**Sample answer:** `Optional` is a return type for a query that may legitimately find nothing, such as `findPrice(sku)`. I do not use it as a field on a JPA entity, I do not take it as a method parameter, and I do not store it in a session. `get` without a presence check is an NPE with extra ceremony. On the success path I use `map` and `flatMap`. For a missing price that is a bug in the catalog, `orElseThrow` with a domain exception is clearer than an empty optional leaking into checkout. `orElse(fallback())` evaluates the fallback even when a value is present; `orElseGet` does not. I have seen `orElse` build a decline object and write a log line on every successful tender.

Returning null from a method typed as `Optional` is a contract break. Returning `Optional` of a collection is usually noise; an empty list already means none. At the REST boundary I turn absence into a `404` or a problem document, not into a JSON null that every client interprets differently. In a principal interview I also say where I still use null: a JSON field that is optional in a DTO, handled at the edge, not pushed into the domain as a nullable money amount.

**Follow-up:** What does `Optional` do to a Spring Data derived query that finds no row?

**Weak answer:** "I wrap every field in Optional to eliminate nulls."

## Q3. Why does a POS "business date" need java.time, not a timestamp alone?

**What the interviewer is probing:** Zone handling, a real source of production defects.

**Sample answer:** `Instant` is a point on the timeline, good for "when the tender was recorded." The store's business date is a `LocalDate` in the store's zone. If I derive "today" from UTC, a sale just after local midnight books on the wrong day and the end-of-day report disagrees with the register. I store the zone id on the store, convert at the edge, and persist the business date as a date, not as a truncated timestamp. `LocalDateTime` without a zone is not a moment and I do not use it as one.

`DateTimeFormatter` is immutable and shared. `SimpleDateFormat` is not thread-safe and does not belong in a singleton service. I do not introduce `java.util.Date` in new code. JDBC mapping should be explicit: `OffsetDateTime` or a database `timestamptz` for the audit instant, `LocalDate` for the business date. I also ban "store local time saved as if it were UTC," which cannot be fixed later without a convention. In the interview I give one example: two stores, one zone east of UTC, a batch that runs at 00:30 UTC and mis-files the evening's sales. The fix is the type, not a comment.

**Follow-up:** How do you test a cutoff that happens at local 02:00 during a daylight-saving gap?

**Weak answer:** "I save `new Date()` and format it in the UI."

## Q4. What do you use from Java 17 in new order-service code?

**What the interviewer is probing:** A working set, not a release-note recital.

**Sample answer:** I write records for request and response carriers and for value objects that are not entities. A compact constructor rejects a bad currency or a negative quantity. I copy lists in that constructor because a record is only as immutable as its components. I use sealed interfaces for tender and quote results so a new outcome breaks the build instead of falling through a boolean. Pattern matching for `instanceof` keeps the cast and the test together. Switch expressions replace fall-through switches on status codes. Text blocks hold SQL and JSON fixtures. `var` is for locals when the right-hand side already names the type; I do not use it on numeric primitives where the width matters, and never on fields.

I do not turn on preview features in production code without a team decision. I do not plan a JPMS migration as part of a feature story; I only deal with `opens` when a library's reflection breaks on 17. Helpful NPE messages are worth enabling if the runtime supports them and they are not already on. The thread I pull is illegal states: records and sealed types remove null auth codes and boolean pairs. I mention virtual threads only as a Java 21 follow-on, not as something I would backport.

**Follow-up:** Which of these features would you forbid on a JPA entity class?

**Weak answer:** "Java 17 is just Java 8 with modules, so I write the same code."

## Q5. A parallel stream in pricing caused a production stall. What happened?

**What the interviewer is probing:** The common pool, blocking, and how you would prove it.

**Sample answer:** `Collection.parallelStream` uses the common `ForkJoinPool`. If the lambda blocks on JDBC, Feign, or a lock, those carrier threads of the common pool sit idle in a queue they cannot steal useful work from, and every other parallel stream in the JVM stalls with them. A quote that "only" parallelized tax lookups can freeze an unrelated batch reprice. The dump shows `ForkJoinPool.commonPool` workers parked in socket reads, and application threads waiting to submit work.

The fix is to remove `parallel()` from any I/O pipeline. If the work is CPU-bound and large enough to care, I use an explicit pool whose bound I can explain, or I keep the stream sequential and fan out with `CompletableFuture` on a named I/O executor that has a timeout. I add a review rule: no `parallelStream` in the order module without a comment that states the pool and why the lambda cannot block. I would reproduce with a test that blocks inside the lambda and asserts the call does not use the common pool, if we keep parallelism at all. I would not "fix" it by raising `parallelism` on the common pool, because that multiplies the damage.

**Follow-up:** How does this change on virtual threads if the blocking call stays in a parallel stream?

**Weak answer:** "Parallel streams use all cores, so they are always a win."

## Q6. How do you handle a checked exception inside a lambda?

**What the interviewer is probing:** A daily Java 8 pain, and whether you hide failures.

**Sample answer:** `Function` does not throw checked exceptions. Wrapping every call in `RuntimeException` lets the stream compile and turns a declared `ReservationException` into an unchecked failure that Spring will roll back, which might be what I want, or might skip a `rollbackFor` policy someone set for the checked type. I do not do that inside a stream that also has business logic after the failure. For a single call I keep the loop or the method ordinary so the `throws` clause stays honest.

If I truly want a pipeline, I write a small adapter that returns a sealed success-or-failure and I handle it at the terminal operation, not in `peek`. I never catch `Exception` inside `map` and substitute a zero price. That is a successful-looking quote with a wrong total. The same rule applies to `CompletableFuture`: an exception must be handled with `handle`, not dropped. In review I ask what the cashier sees and what the database sees. If those two answers differ, the lambda is hiding a partial failure. Principal-level is preserving the type of the failure until the boundary that maps it to a problem document.

**Follow-up:** How does this interact with a transactional proxy around the method that started the stream?

**Weak answer:** "I catch Exception and return null so the stream continues."

## Q7. Compare a record and a class for a TenderRequest DTO.

**What the interviewer is probing:** Immutability, validation, Jackson, and JPA boundaries.

**Sample answer:** A record gives me an immutable carrier with generated equality and a clear component list, which is what I want for a request DTO that should not be mutated by a filter after validation. I validate in the compact constructor or with Bean Validation annotations on the components, and I keep the domain `Order` separate so Jackson cannot bind a client field onto a version column. Equality on a record is useful in tests: the receipt I built equals the receipt I expected, component by component.

I use a class when I need inheritance beyond a sealed interface, a framework that demands a no-arg constructor and setters, or a mutable builder for a large optional payload. Many JSON libraries on the Java 17 line bind records; I confirm the one we ship rather than assuming. I do not use a record as an entity. I do not put a live `InputStream` or an `EntityManager` in a record. If the DTO includes a list, I copy it on the way in so the caller cannot change the request after the fact. The interview point is fit: records for transparent values, classes for lifecycle and frameworks that still need beans.

**Follow-up:** How do you evolve the record when a client omits a new component?

**Weak answer:** "Records are just syntactic sugar and behave exactly like JPA entities."

## Q8. What would you re-evaluate on an upgrade from 17 to 21, and what would you leave alone?

**What the interviewer is probing:** Forward judgment tied to this service, not a feature list.

**Sample answer:** I would leave JPA transaction boundaries, idempotency keys, and the sealed tender model alone. Those do not get better because the JDK moved. I would re-evaluate thread pools: virtual threads make blocking Feign and JDBC cheap to schedule, but Hikari size, Redis timeouts, and gateway timeouts are still the caps. I would look for `synchronized` pinning on the 21 line and for thread locals that inflate with one virtual thread per request. I would consider structured concurrency for the price-and-tax fan-out so cancellation is a scope, not a scattered `orTimeout`.

Generational ZGC becomes the low-pause collector I would actually test if G1 pauses were the SLO problem. Pattern matching on switch completes the sealed `TenderResult` story. Sequenced collections matter only if we depended on iteration-order quirks. I would run the Testcontainers suite and a JMeter scenario on the new JDK before any flag change, and I would watch for libraries that open JDK internals and fail closed. The answer I will not give is a rewrite of the service in a reactive style because virtual threads removed the reason we might have done that for I/O.

**Follow-up:** Which library upgrade would you insist on before flipping the runtime to 21?

**Weak answer:** "Upgrade and turn on every preview feature in the same release."

## Q9. Debug a quote total that changes when someone adds parallel().

**What the interviewer is probing:** State, reduction, and non-associative bugs.

**Sample answer:** A sequential total and a parallel total differ when the lambda has side effects or the reduction is not associative. I look for a lambda that writes a shared `ArrayList` or a field on the service, for a `reduce` that starts from a mutable identity, and for a floating `double` add that is not associative in the way the business expects. Minor-unit `long` addition is associative; a running average written into one object is not. I also look for `forEach` that inserts into a non-concurrent map.

I force the test to run the pipeline both ways on a fixed list of lines with a known sum. If they differ, I delete the shared mutation and use `mapToLong(...).sum()` or a concurrent collector deliberately. If they match and production still mismatches, the input list is being mutated on another thread, which is not a stream bug. I check `peek` that was doing a required tax rounding; `peek` is not a place for that, sequential or not. The fix is a pure function of the line list. I keep `parallel()` out unless the purity and the gain are both there.

**Follow-up:** Why can `reduce(0L, Long::sum)` still be slower than `mapToLong` even when the result matches?

**Weak answer:** "Parallel streams are non-deterministic, so totals are allowed to change."

## Q10. How do default methods and sealed types interact when you evolve a port?

**What the interviewer is probing:** API evolution on Java 8 interfaces and Java 17 sealing.

**Sample answer:** A default method lets me add a behavior to an interface without breaking existing implementors. That is how Java 8 grew the collections API. The risk is conflict: two defaults with the same signature force the implementor to override, and a class method wins over a default. I do not put required new behavior in a default that silently does the wrong thing, such as `refund` defaulting to success. A default that throws `UnsupportedOperationException` recreates the LSP bug I was trying to avoid.

Sealed types are the opposite tool: I control every implementor, so I can add a method without a default and fix each permitted type in the same change. That is appropriate for `TenderResult` inside one service. It is hostile for a port that another team implements in another repository, unless we release together. For a Feign API I evolve with an optional JSON field, not a sealed Java type the client must share as a binary. Inside the service, I seal the domain result. On the wire, I keep the contract compatible. I state that split in a principal interview so I do not sound like every abstraction should be sealed.

**Follow-up:** When would you still ship an abstract class instead of a sealed interface with defaults?

**Weak answer:** "Default methods are multiple inheritance of state, so I avoid interfaces."
