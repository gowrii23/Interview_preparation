# Spring Boot core — interview questions

## Q1. Why did the @Transactional annotation on reserve not roll back?

**What the interviewer is probing:** Proxy self-invocation and rollback rules, together.

**Sample answer:** I ask two questions before I look at the database. First, was `reserve` called through the Spring proxy or through `this`? `@Transactional` is advice on a proxy. A call from `checkout` to `this.reserve()` runs the target method directly, so no transaction starts and no advice runs. The same is true for `@Cacheable` and `@Async`, and for private or final methods the proxy cannot override. The fix I prefer is to move `reserve` onto another bean so the call crosses the proxy. Self-injection and `AopContext.currentProxy()` work and leak the container into the service.

Second, if the proxy was entered, what was thrown? The default rollback rule rolls back on `RuntimeException` and `Error` only. A checked `InsufficientStockException` commits unless `rollbackFor` includes it. Catching the exception inside the method and returning normally also commits, because the proxy sees success. I would look for both bugs; they stack. I would not "fix" it by annotating the private method and hoping. In a principal interview I draw the proxy and the target, and I mention that `REQUIRES_NEW` has the same self-invocation trap and, when it does run, borrows a second Hikari connection.

**Follow-up:** How does OSIV confuse this diagnosis?

**Weak answer:** "Transactional means every method in the class is atomic, including private ones."

## Q2. Where do you draw the transaction boundary around payment authorization?

**What the interviewer is probing:** Connection holding, and honesty about atomicity.

**Sample answer:** I do not put the Feign or gateway call inside `@Transactional`. The proxy binds a JDBC connection to the thread for the whole method. A two-second gateway timeout is then a two-second occupied session, multiplied by concurrent cashiers, and Hikari pending time climbs while the database does little work. I load the order in a short transaction, call the gateway outside, then open a second short transaction to record the authorized or declined result. If the process dies between those steps, the outcome is unknown and reconciliation owns it. I do not pretend one annotation makes a remote system and a local commit atomic.

`REQUIRES_NEW` is how I persist the attempt row even if the later order update rolls back, and I account for the extra connection. Read-only on the load path is a hint to skip dirty checking, not a security control. Isolation stays at the database default unless I have a lost-update story, which I usually solve with `@Version` or a conditional update rather than `SERIALIZABLE` on every checkout. The sentence I want the interviewer to hear: the transaction boundary is the consistency boundary, and the gateway is outside it on purpose.

**Follow-up:** What isolation do you rely on for the conditional on-hand update?

**Weak answer:** "I annotate the controller method and the whole request is one transaction."

## Q3. Explain singleton scope and a bug it caused.

**What the interviewer is probing:** Shared mutable state in the container.

**Sample answer:** A Spring singleton is one instance per application context, shared by all request threads. That is correct for a stateless `CheckoutService` whose dependencies are thread-safe. It is a race if the service has a `HashMap` field of carts, a `SimpleDateFormat`, or a mutable request DTO stored on `this`. Two stores then see each other's lines, or a formatter throws, and the bug is load-dependent. I keep per-order state on the aggregate loaded inside the request, and I keep the service free of request fields.

Prototype scope does not fix this when a singleton is injected with one prototype at construction; it keeps that one instance. I would use `ObjectProvider` only if I truly need a new instance per use. Request scope for a cart duplicates the database and surprises tests that have no request. I also mention that two Spring test contexts mean two singletons, so a static cache "shared across the app" is not shared across those tests. The principal point is that thread safety is still our job: the container does not wrap singleton fields in locks.

**Follow-up:** What does `@Scope("prototype")` on a `@Bean` method do when another singleton depends on it?

**Weak answer:** "Singleton means one instance per HTTP session."

## Q4. A bean failed at startup after a harmless library upgrade. How do you trace it?

**What the interviewer is probing:** Auto-configuration and conditionals.

**Sample answer:** Boot's auto-configuration is conditional on classes, properties, and missing beans. A new jar on the classpath can satisfy `@ConditionalOnClass` and start a `DataSource`, a security filter, or a second JSON mapper I did not ask for. I enable the condition evaluation report (`debug` or the conditions endpoint in a lower environment) and read which auto-configuration matched. I also read the cause chain: `BeanCreationException` is usually a wrapper around a missing property, a cycle, or a factory method that threw.

If two `DataSource` beans appear, I name the one the order module must use and exclude the other, or I mark one `@Primary` only when that is honest. I do not exclude random auto-configurations until the report says they matched. Circular injection fails fast; I break the cycle instead of `@Lazy` unless I can explain the lifecycle. `@ConfigurationProperties` validation (`@Validated`) turns a missing timeout into a startup failure, which I prefer to a null on the first tender. After the fix I keep the dependency that triggered the auto-config explicit in the build file so the next upgrade is not a surprise. The interview line is: classpath is configuration.

**Follow-up:** How do you stop a test slice from starting the real DataSource?

**Weak answer:** "I would comment out beans until it starts."

## Q5. How do JDK proxies and CGLIB change what gets advised?

**What the interviewer is probing:** Proxy mechanics at the level of a debugging session.

**Sample answer:** If the bean is injected as an interface, Spring typically uses a JDK dynamic proxy that implements that interface. Only interface methods are advised. If there is no interface, or if proxy-target-class is on, CGLIB subclasses the concrete class and cannot override `final` or `private` methods. In both cases the object in the context is the proxy and the target is inside it. Self-invocation skips advice either way. A class-level `@Transactional` does not advise a `final` method under CGLIB, so a write method someone marked final "for safety" silently has no transaction.

I inject interfaces for ports (`PaymentGateway`) so the service depends on the role and tests pass a fake without CGLIB. I do not mark transactional business methods `final`. I know that `@Transactional` on a class applies to public methods that the proxy can see; a public method called only from inside still needs an external entry. When a transaction "does not start," I check the runtime type in the debugger. If I see the target class and not a proxy, I am on the wrong object, often because `new CheckoutService(...)` bypassed the container in a test that then talked to a real database.

**Follow-up:** What does `exposeProxy` change, and why is it a last resort?

**Weak answer:** "Spring always uses reflection, so annotations work on private methods."

## Q6. How do you configure HikariCP for an order service on Postgres or Oracle?

**What the interviewer is probing:** Pool parameters you can defend.

**Sample answer:** I size `maximumPoolSize` per pod and multiply by replicas, then compare with the database session limit, leaving room for jobs and administrators. A pool as large as the Tomcat thread pool just queues inside the database. `connectionTimeout` is the wait I am willing to add to a cashier request before failing; it should be visible as an error, not a two-minute hang. `maxLifetime` sits below the network or database idle timeout so the pool closes sockets before a firewall does, which avoids "connection reset" on the next tender. `leakDetectionThreshold` logs when a connection is held too long; that is how I catch a transaction wrapped around a remote call.

I keep `minimumIdle` close to the max for a steady API so I do not pay connection setup on the first spike. I set the JDBC socket timeout so a stuck Oracle session is not immortal. I do not copy a blog's pool of 100 onto 30 pods. Validation is on borrow as Hikari does by default with a lightweight check, and I still rely on `maxLifetime` for dead connections the check cannot see. Metrics I want in New Relic or the actuator: active, idle, pending, and timeout count. The principal close: the pool is a budget shared with every other replica, not a local performance knob.

**Follow-up:** Why can a long `maxLifetime` still be wrong if it is under the database timeout?

**Weak answer:** "Set maximumPoolSize to 1000 so we never wait."

## Q7. Constructor injection versus field injection in a review.

**What the interviewer is probing:** Testability and cycles.

**Sample answer:** I require constructor injection for required dependencies. The fields are `final`, the object cannot be built half-configured, and a unit test calls `new` with fakes and no Spring. Field injection hides the dependency list, needs reflection in tests, and lets a cycle exist until a method runs. Setter injection is for a truly optional collaborator, which is rare; an `ObjectProvider` or a no-op strategy is clearer than a null I must remember to check.

I also reject a constructor with twelve services. That is not a style nit; the class is an orchestrator of the whole company and will be the merge-conflict magnet in checkout. Split by port: pricing, inventory, payment. Circular constructors should fail startup. `@Lazy` on one side hides the cycle and creates a proxy that surprises equality and transactions. I allow `@Autowired` on a single constructor only as noise; on modern Boot a single constructor is injected without it. The review comment I write is short: make the dependency list true, and make the class small enough that the list fits in my head.

**Follow-up:** How do you inject a configuration value without scattering `@Value`?

**Weak answer:** "Field injection is the Spring standard and easier for the container."

## Q8. A read-only transaction still issued an UPDATE. Why?

**What the interviewer is probing:** What `readOnly` does and does not do.

**Sample answer:** `readOnly = true` is a hint. Hibernate can skip dirty checking and some drivers can choose a read connection. It is not a guard that rejects writes, and it is not applied if the method ran without the proxy, or if a different transaction manager is in effect, or if an outer `REQUIRED` transaction was already read-write and this method joined it. Joining an existing read-write transaction means the inner `readOnly` flag does not start a new read-only scope.

An update can also be a flush of a dirty entity loaded earlier in the same transaction and mutated by a mapper that "only reads." I look for a DTO mapper that touches setters on the entity, and for OSIV keeping that entity open until Jackson runs. The fix is a projection or a DTO loaded by query, and a transaction boundary that does not include the serializer. If I need a guarantee, I use a database user that lacks update grants for the reporting path, not an annotation. I mention that in a principal interview because people treat `readOnly` as a security control. It will not stop a bug, and it will not stop a determined query.

**Follow-up:** What does Hibernate skip when the hint is actually honored?

**Weak answer:** "readOnly starts a transaction that the database cannot write in, always."

## Q9. How do profiles and configuration properties fail a POS service?

**What the interviewer is probing:** Environment drift and startup validation.

**Sample answer:** I bind configuration with `@ConfigurationProperties` and validate it, so a missing `pos.pricing.quote-timeout` fails startup instead of NPEing on the first quote. Profiles select files and beans: local, test, prod. The failure mode I have seen is a profile that changes behavior, not just endpoints: local uses a no-op payment client and someone deploys with the local profile, so tenders "succeed" without the gateway. Production must not contain that bean. I use the profile to set URLs and pool sizes, and I use types, not booleans named `mock`, to select adapters, with the mock adapter absent from the prod classpath if I can manage it.

Secrets do not live in `application.yml` in git. They come from the environment or a secret store. A refresh of `@RefreshScope` config must not assume in-flight authorizations see a single consistent snapshot; I roll endpoint changes deliberately. I also watch for two property sources overriding the pool size in a way that only appears in Kubernetes, because the env var name did not match relaxed binding and the default stayed in place. The principal point is that configuration is part of the binary's behavior and belongs in review and in the startup log as non-secret values.

**Follow-up:** How does relaxed binding turn an environment variable into a property?

**Weak answer:** "We use one application.yml and edit it on the server."

## Q10. Design the checkout service object graph you would actually defend.

**What the interviewer is probing:** A coherent Spring design, not annotation trivia.

**Sample answer:** I draw four objects. `CheckoutApplicationService` is the transaction and orchestration boundary, constructor-injected with ports. `Order` is the aggregate, plain Java, no Spring annotations. `InventoryPort` and `PaymentPort` are interfaces. Adapters are `@Component` types that call Feign or the gateway and map failures to domain results. A small `IdempotencyStore` port hides the unique insert. The service method loads or creates state in a short transaction, calls ports outside that transaction when they do network I/O, and writes the outcome in a second short transaction.

I do not put `@Transactional` on the port implementation and also on a private method of the service. One boundary is understandable. Events for loyalty go through an outbox written in the same transaction as the order state change. The controller depends on the application service, validates the DTO, and maps sealed results to HTTP. Nothing in this graph is a singleton with mutable carts. I can unit test the aggregate and the service with fakes, and I need Spring only for a slice test of the controller and a Testcontainers test of the idempotency insert. That split is the answer I want a principal panel to remember.

**Follow-up:** Where does `@TransactionalEventListener` fit, and what is its failure mode?

**Weak answer:** "One `@Service` class with all the annotations on the public methods is enough."
