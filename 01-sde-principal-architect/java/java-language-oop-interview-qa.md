# Java language and OOP — interview questions

## Q1. How do you model money and tender outcomes in a POS order?

**What the interviewer is probing:** Whether domain types protect invariants, or the design is a bag of setters and booleans.

**Sample answer:** I treat money as an immutable value: a `long` in minor units plus an ISO currency code, with `plus` implemented using `Math.addExact` so overflow fails instead of wrapping. The type is `final`, fields are `final`, and arithmetic returns a new instance. I never use `double` for a charged amount. An order line does not expose setters for quantity and extended price separately; `changeQuantity` recomputes the extension so those fields cannot drift.

Tender outcomes are a sealed interface, finalized in Java 17: `Authorized`, `Declined`, and `RetryableFailure` as records. Callers switch on the type. A boolean `success` plus a nullable reason lets a declined card look like a captured payment if one branch forgets a check. Sealed types make a new outcome a compile error at every switch, which is what I want when wallet tenders are added later.

I keep the aggregate's consistency boundary on lines, totals, and tender state. Loyalty accrual is a side effect after commit, not a subclass of `Order`. In a principal discussion I also say which exceptions are checked at the port (`ReservationRejected`) because Spring's default transaction advice commits on checked exceptions unless `rollbackFor` is set. That is part of the domain model, not an afterthought in the service class.

**Follow-up:** How does this change if the same SKU can be priced in two currencies on one order?

**Weak answer:** "I use BigDecimal everywhere and an enum with SUCCESS and FAILURE."

## Q2. Explain equals and hashCode for an order identity.

**What the interviewer is probing:** Contract knowledge and the damage a broken implementation does inside maps and JPA sets.

**Sample answer:** If `a.equals(b)`, then `a.hashCode()` must equal `b.hashCode()`. Equality is reflexive, symmetric, transitive, consistent, and false for null. I implement them together, usually with `Objects.equals` and a stable hash of the same fields. For a value such as `Money`, that is amount and currency. For an order that already has a business key, I use `storeId` and `orderNumber`, not a mutable status field. If status is inside `hashCode` and I insert the order into a `HashSet` before payment, then mark it authorized, `contains` looks in the wrong bucket and the order is "lost" in memory.

I do not use `==` for value equality. I do not base entity equality on a generated id that is still null before `persist`, because two transient orders would be unequal to themselves after save, and a `Set` would hold duplicates. I also do not walk lazy JPA associations inside `equals`; that loads the graph and can recurse through `Order` and `OrderLine`. `compareTo` must agree with `equals` if these objects enter a `TreeMap`. Symmetry matters: `order.equals("123")` returning true is a contract break. I would rather have a small value type for the key than overload equality on the aggregate.

**Follow-up:** What happens if two objects have the same hash and are not equal?

**Weak answer:** "IDE generated equals is always fine, including on every field of the entity."

## Q3. When do you choose inheritance over composition?

**What the interviewer is probing:** Whether you can defend LSP with a real POS example, not a slogan.

**Sample answer:** I default to composition. Pricing, tax, and loyalty change on different schedules, so they are policies an `Order` holds, not subclasses such as `TaxFreeOrder extends Order`. Inheritance is justified when there is a true is-a relationship and the subclass honors the superclass contract, including preconditions and exceptions. If `RefundTender.capture()` throws `UnsupportedOperationException`, it is not a `Tender`. I model that as a different type or a sealed result, not a subclass that surprises callers.

The fragile base class shows up when a parent constructor calls an overridable method. Subclass fields are still default, and a later parent change reorders calls the child depended on. I keep superclass state private, expose protected hooks rarely, and prefer a template method only at a framework extension point. Fields are not polymorphic; if both classes declare `status`, the field read depends on the compile-time type. Overload resolution is also compile-time, so a method set `apply(Payment)` and `apply(CardPayment)` will not dispatch on the runtime card type if the reference is `Payment`. I mention that bug because it looks like polymorphism and is not. Sealed interfaces give me a closed hierarchy without a deep class tree.

**Follow-up:** How would you unit test a pricing policy without Spring?

**Weak answer:** "I always use interfaces, so I never use inheritance."

## Q4. A cart total is wrong only when a subclass discount is applied. How do you debug it?

**What the interviewer is probing:** Dispatch rules, override bugs, and a disciplined debug story.

**Sample answer:** I start from the number, not the framework. I write down the line inputs, the expected minor units, and the actual total. Then I check whether the discount method that ran is the override I think ran. Overload selection uses the compile-time type, so a caller holding `Discount` may never enter `EmployeeDiscount`'s overload. I log the runtime class once, or I set a breakpoint on both methods. I also check construction order: if the superclass constructor called `apply()`, the subclass rate field was still zero.

I look for integer division truncating a percent, for a sign error on a return line, and for `double` rounding that only appears on some prices. I check whether the subclass strengthened a precondition, violating LSP, so some carts skip the discount and some throw. If the bug appears only after the object was copied into a DTO, I compare mapping code; a subclass field is easy to drop.

The fix is usually to replace the hierarchy with a strategy that takes an explicit `DiscountContext`, covered by a table test of rates and quantities. I would not patch it by adding another `instanceof` without deleting the old path. In production I want the pricing inputs and the policy name on one structured log line so the next bad total is searchable by order id in Kibana.

**Follow-up:** How do you keep a promotion table from becoming a second, untested pricing engine?

**Weak answer:** "I would add logs everywhere and reproduce it in production."

## Q5. What does encapsulation mean past private fields?

**What the interviewer is probing:** Invariants, API surface, and defensive copies.

**Sample answer:** Private fields with public setters do not encapsulate anything; they move the invariant into every caller. Encapsulation means the only way to change an `OrderLine` is a method that leaves quantity, price, and tax consistent. The class does not return its internal `ArrayList`; it returns `List.copyOf` or an unmodifiable view, because a caller who clears the list corrupts the aggregate with no method on `Order` ever running.

I use package-private constructors when a factory in the same package is the only legal way to build a tender, and I do not assume Java modules unless the service is actually modularized. Validation sits at the boundary (`@Valid` DTO) and again in the domain constructor, because not every caller is the controller. Batch jobs and message consumers skip Bean Validation.

I keep Spring annotations off the domain type when I can, so the same `Order` is testable without a container. The application service is the transaction boundary; the entity does not inject a repository. That split is what I defend in a principal interview: the domain enforces rules, the service enforces atomicity and I/O. An anemic `Order` plus a 2,000-line service duplicates those rules the first time a second entry point appears.

**Follow-up:** Where would you put a rule that spans order and inventory?

**Weak answer:** "Encapsulation means getters and setters and a DAO layer."

## Q6. How do records and sealed classes change a Java 17 service?

**What the interviewer is probing:** Practical use of 16/17 language features, and where they do not belong.

**Sample answer:** Records, standard since 16, are my default for DTOs, events, and value carriers. I get a constructor, accessors, `equals`, `hashCode`, and `toString` that match the components. A compact constructor validates the currency code or the quantity. They are shallowly immutable, so if a component is a list I copy it in the constructor. I do not use a record as a JPA entity. Entities have identity and a persistence lifecycle; record equality is structural and there is no room for a lazy proxy to behave like a normal mutable entity.

Sealed interfaces, finalized in 17, close a hierarchy: `TenderResult` permits only the outcomes I ship. Adding `WalletAuthorized` fails the build at every exhaustive switch. On Java 17 I use pattern matching for `instanceof` (`if (r instanceof Authorized a)`) which is standard, and I treat pattern switches as a later-JDK convenience if the preview flag is off. Default methods on the sealed interface can still share a small behavior, but I do not put state there.

I would not seal a type that downstream stores implement, and I would not record-ify a 40-field order document just to avoid writing a class. The feature is for a small, honest carrier.

**Follow-up:** How do you evolve a sealed interface without breaking a Feign client generated from OpenAPI?

**Weak answer:** "Records replace all classes and sealed means abstract."

## Q7. Design the order aggregate. What is inside it?

**What the interviewer is probing:** Consistency boundaries, not UML decoration.

**Sample answer:** The aggregate is the cluster of objects that must change together in one transaction. For checkout I include the order header, its lines, the computed totals, and the list of tender attempts with their states. I do not include the SKU master, the full inventory position, or the customer's loyalty ledger. Those are other aggregates, referenced by id. A line holds sku id, quantity, unit minor units, and tax, not a live `@ManyToOne` product graph that lazy-loads on every screen.

Invariants I enforce inside `Order`: no line with zero quantity, total equals the sum of extensions plus tax, and you cannot add a line after the order is authorized unless a defined amend flow says so. The application service loads the order, calls `addLine`, and lets dirty checking persist the change. Inventory is a conditional update in the inventory service, or a reservation id stored on the order, not a foreign key the order service updates directly.

I keep the aggregate small so the row lock or the version check is short. A god `Order` that cascades to audit history, shipments, and payments makes every read expensive and every write contentious. Audit is an outbox event. That is the design I would draw on the board, including what is deliberately outside the boundary.

**Follow-up:** How do you handle a price change between scan and pay?

**Weak answer:** "One Order class with JPA relations to every table."

## Q8. Why is constructor injection your default in a Spring service that is still "just Java"?

**What the interviewer is probing:** Testability and immutability of dependencies, and whether you understand the container at all.

**Sample answer:** Constructor injection makes dependencies explicit and `final`. A unit test calls `new CheckoutService(gateway, orders)` and passes fakes. Field injection forces reflection or a Spring test for every case, and it hides cycles until runtime. I do not use setter injection for required ports; a setter means the service can exist half-built.

The class stays a plain Java object. Annotations may mark the constructor, but the domain rules do not need the container. That is what I want when a batch job reuses pricing. Circular constructors fail fast at startup, which is the right failure: `OrderService` and `InventoryService` depending on each other means the boundary is wrong. I break the cycle with an event or a third orchestrator, not with `@Lazy`.

I also talk about object publication. Dependencies that are immutable or thread-safe can be shared by the singleton service. A mutable `HashMap` field on that singleton is a race across cashier requests. I keep per-order state on the aggregate, not on the service. If a dependency is optional, `ObjectProvider` or a small strategy is clearer than a null field that NPEs on the first wallet tender.

**Follow-up:** What does prototype scope actually do when injected into a singleton?

**Weak answer:** "I always use `@Autowired` on fields because it is less code."

## Q9. A developer adds `equals` that compares only order id, and a `HashMap` keyed by order misbehaves before save. What happened?

**What the interviewer is probing:** A concrete failure mode you have thought through.

**Sample answer:** Before `persist`, the generated id is null. Every transient order has the same hash bucket and `equals` returns true for any two of them if equality is "both ids null" or, worse, `equals` returns false for two unsaved orders and true after save against a different instance that now shares an id. A `HashSet` of new orders collapses them or fails to find them after the id is assigned, because the hash changed while the object sat in the set. That is the mutable-key failure.

I reproduce it with two new orders, insert both in a set, assign ids, and assert `contains`. The fix is a stable business key assigned before the object is used as a map key, or not using the entity as a key at all. I use `Map<OrderId, Order>` where `OrderId` is assigned by the application (store plus sequence) before insertion. I also check symmetry and a null-safe `equals`. This bug rarely shows in a test that only loads managed entities from a repository, so I keep one pure unit test that never starts Hibernate. In review I reject `equals` implementations generated across every column, especially associations.

**Follow-up:** How would you implement equality for a JPA entity that truly has no business key until insert?

**Weak answer:** "Hibernate handles equals, so application code should not define it."

## Q10. What do you say about SOLID in a principal interview without reciting the letters?

**What the interviewer is probing:** Judgment. They have heard the acronym. They want a design consequence.

**Sample answer:** I pick one change and show the blast radius. A new tax rule should add a `TaxCalculator` implementation and a registration, not an edit in `CheckoutService` (that is the open-closed idea, stated as a consequence). A `PaymentGateway` port that also has refund, void, and gift-card balance methods forces every fake and every adapter to stub methods it does not support, so I split the ports the callers actually use. A domain exception that is checked versus unchecked is a dependency-inversion detail: the service depends on the port, and the transaction advice depends on the exception type.

I explicitly say which letter I will violate on purpose. A small service does not need a strategy per line of code. Speculative extension points are how POS code becomes unreadable. I also tie Liskov to the tender example rather than to a square-and-rectangle puzzle: if the substitute cannot authorize, it does not implement `Authorizable`. The interviewer hears that I can use the vocabulary and also stop. If they want the five names, I can list them in one sentence after the example, not before.

**Follow-up:** Show me a place where adding an interface made the code worse.

**Weak answer:** Reciting five definitions with no code and no tradeoff.
