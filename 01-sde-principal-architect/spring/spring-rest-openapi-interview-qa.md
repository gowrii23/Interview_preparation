# REST and OpenAPI — interview questions

## Q1. Design POST /orders/{id}/tenders so a store retry cannot double-charge.

**What the interviewer is probing:** Idempotency as a design, not a header name.

**Sample answer:** The client sends an `Idempotency-Key` it generated, a UUID scoped to that authorization attempt. The order service inserts a row keyed by that value, with a hash of the body and the eventual response, under a unique constraint in Oracle or Postgres. The insert winner calls the payment gateway. A retry with the same key and the same body returns the stored receipt and does not call the gateway again. The same key with a different amount or tender type returns `409`, because replaying a different request under a key the client claims is identical is a bug or an attack. I do not store this only in a JVM map or in a Redis key that can evict under pressure; eviction turns a retry into a second charge.

The first response is `201` when a tender resource was created, or `200` when I am returning a replay, and I document which one clients must handle. A gateway timeout stores `UNKNOWN` and reconciliation resolves it; the HTTP result to the client is not a fake success. The key expires on a documented window. After expiry, reuse is a new attempt. Apigee may forward the header and retry on a connection reset; the service still enforces the constraint because batch jobs bypass the gateway. I mention that the unique constraint is the real lock, and Feign between services must send the same key if order and payment are separate.

**Follow-up:** What do you store if the process crashes after the gateway returns and before the receipt row is updated?

**Weak answer:** "PUT is idempotent, so I tell the client to retry the POST freely."

## Q2. Which status codes do you return for checkout, and which do you refuse?

**What the interviewer is probing:** Contract clarity and the "200 with success false" anti-pattern.

**Sample answer:** `201` and a `Location` when an order is created. `202` when capture is accepted and not settled, so the client does not print a settled receipt. `400` for a malformed body or a Bean Validation failure, with field errors. `422` or a documented `400` with a problem `code` when the body is well formed and the domain rejects it, such as a zero quantity; I pick one and keep it stable. `409` for a version conflict or an idempotency key reused with a different body. `404` when the order id does not exist for this store. `401` and `403` stay distinct: not authenticated versus not allowed at this store. `429` when the client must back off. `503` when inventory or the gateway is unavailable and a retry may help.

I refuse `200` with `{ "success": false }`, because caches, Apigee policies, and client code treat `200` as success. I refuse leaking SQL text in a `500`. The body is a problem document: `type`, `title`, `status`, `detail`, a stable `code`, and a correlation id the store can read to me while I search Kibana. In a principal interview I also say which codes are safe for the client to retry automatically. `503` and `429` might be. `409` is not a blind retry. A declined tender is a `200` or `201` with a declined receipt, not a `500`, because the system worked and the card did not.

**Follow-up:** When is a declined card a 402, and why might you still avoid it?

**Weak answer:** "I return 200 for everything so the app does not throw."

## Q3. How do you paginate order history for a store?

**What the interviewer is probing:** Offset versus keyset, and authorization.

**Sample answer:** Offset pagination (`page` and `size`) is easy and wrong once new orders insert at the top: the client skips a row or sees a duplicate across pages. I use keyset pagination on a stable order, typically `createdAt` plus `id`, and the client passes the last seen pair. The query is `where store_id = :store and (created_at, id) < (:ts, :id) order by created_at desc, id desc` with a bound `limit`. That needs a matching index. I cap `size` so a device cannot ask for a year of tickets in one call.

The store id comes from the token, not only from a query parameter the device can edit. I do not return the entity graph; I return a projection DTO. Filtering by status is an additional predicate, not a second endpoint per status, unless the authorization differs. I document the cursor as opaque if I might change the sort, and as an explicit pair if partners must debug it. Total count is expensive and usually unnecessary for an infinite scroll; I do not run a count query on every page unless a screen truly needs it. The principal point is stability under inserts, plus authorization, not the query parameter names.

**Follow-up:** How do you expose this cursor in OpenAPI without pretending it is a page number?

**Weak answer:** "I return all orders for the store and let the UI page them."

## Q4. Code-first or contract-first OpenAPI for this service?

**What the interviewer is probing:** A decision with a drift story.

**Sample answer:** I use contract-first when Apigee and several clients share the spec: the YAML is reviewed, published, and the server stubs or the client are generated. Drift then shows up as a failed contract test, not as a store release that cannot parse a field. I use code-first, springdoc on the controllers, when the only client is our own and speed of change matters, and I still publish the document and run a contract check in CI so a DTO rename fails the build. I do not maintain a wiki that restates the spec.

Either way I document the idempotency header, the problem schema, and the auth scope inside the spec. Enums are a compatibility promise: adding a tender type can break a generated client that switches exhaustively, so I set `additionalProperties` and an explicit unknown-value policy. Removing a field or changing a type is a new major version (`/v2`), not a silent edit. I keep `/v1` until devices have moved. The service enforces the contract even when the caller bypasses Apigee. In a principal interview I name who owns the spec and what merges if the gateway policy and the service disagree. The spec wins for the external contract; the service still defends invariants the spec cannot fully express, such as "do not capture twice."

**Follow-up:** How do you review a "compatible" change that adds a required field?

**Weak answer:** "Swagger annotations are the documentation, so we do not need a process."

## Q5. A client can set the order status by sending a JSON field. How did that happen?

**What the interviewer is probing:** Over-posting and entity binding.

**Sample answer:** The controller accepted a JPA entity, or a DTO that was mapped field-for-field including `status`, `version`, and `price`. Jackson set what the client sent, and merge or dirty checking wrote it. The fix is a command DTO with only the fields the action allows, such as sku and quantity for add-line, and a method on the aggregate that is the only way to change status. I ignore unknown JSON properties or I reject them, as a conscious choice: ignoring hides typos, rejecting is stricter for a partner API. I never bind the entity directly.

I add a test that posts `status: AUTHORIZED` and asserts the stored status did not change. Mass assignment is a security defect, and SonarQube or a review checklist should catch entity parameters on controllers, but the test is the proof. The same bug exists on `PATCH` if the merge function copies every incoming key. I define the patch document. For price, the client may send a scanned price as a claim; the server recomputes from the price book and treats a mismatch as a domain error, not as a column update. That is the design I describe so the interviewer hears both the binding bug and the pricing invariant.

**Follow-up:** How do you allow an admin override of price without opening the same hole?

**Weak answer:** "We use `@JsonIgnore` on a few fields and bind the entity."

## Q6. How do you validate input without mixing it with domain rules?

**What the interviewer is probing:** Boundary versus invariant.

**Sample answer:** Bean Validation on the DTO answers "is this request well formed?": present sku, positive quantity, currency of length three, idempotency key not blank. `@Valid` on the argument triggers it, and a `@ControllerAdvice` turns `MethodArgumentNotValidException` into a `400` with field errors. Method-level validation needs `@Validated` on the class. These checks do not hit the database.

Domain rules run inside the aggregate or the service after a consistent read: enough on-hand, order still open, tender type allowed for this store. Those return a sealed result or a domain exception mapped to `409` or `422` with a stable `code` such as `SKU_NOT_ON_HAND`. I do not encode stock levels as a Bean Validation constraint that queries a repository; that hides I/O inside validation and is painful to test. I still defend the constructor of `Money` so a batch importer cannot bypass the controller and build a bad value. Two layers is deliberate. The weak design is one giant validator class that does HTTP, SQL, and pricing. I say where each rule lives and how a unit test reaches it without Tomcat.

**Follow-up:** What is the failure mode of validating in the controller and nowhere else?

**Weak answer:** "I validate everything in the database with triggers only."

## Q7. Apigee sits in front of the order API. What do you still enforce in the service?

**What the interviewer is probing:** Defense in depth, not gateway worship.

**Sample answer:** Apigee is useful for TLS edge, OAuth or API keys, quotas, and routing. I still authenticate and authorize in the service or I validate the JWT the gateway forwards, and I bind the store id to the token. A header `X-Store-Id` the device can set is not authorization. I still enforce idempotency, request validation, and the domain rules, because other microservices and jobs call the pod directly inside the network. I still apply timeouts and a bulkhead on outbound calls; the gateway does not protect me from a slow inventory service unless every hop is designed to.

I do not trust the gateway to strip sensitive fields from logs that the service itself writes. I do not put the only copy of the OpenAPI contract in the gateway and let the service drift. Rate limits at Apigee protect the edge; a hot SKU lock is still a database problem the gateway cannot see. If the gateway retries POSTs, those retries must carry the idempotency key or the gateway is the thing that double-charges. I would draw the trusted boundary in the interview: which hop is TLS, which identity reaches the pod, and what a stolen pod-to-pod call can do. That is the principal answer, not the product name alone.

**Follow-up:** Where do you terminate TLS, and what does that imply for the hop to the pod?

**Weak answer:** "The gateway handles security, so the service can be open."

## Q8. How do you version and evolve the tender enum?

**What the interviewer is probing:** Compatibility practice.

**Sample answer:** I treat the external enum as a compatible-evolution problem. Adding a value is safe for a client that ignores unknowns and unsafe for a generated client that switches exhaustively and fails closed. I document the policy: servers may add values, clients must tolerate an unknown value by showing "other" or by failing a single tender rather than the whole history download. Removing or renaming a value is a breaking change and goes to `/v2` while `/v1` still emits the old value for old apps. I do not reuse a numeric code for a new meaning.

Inside the Java service, a sealed type or a Java enum can be stricter than the wire, with an explicit `Unknown` mapping at the edge so a new database value does not throw in Jackson and turn a GET into a `500`. The OpenAPI document is updated in the same change as the code, and a contract test loads a fixture that includes the new value and one fixture from the previous version. Feature flags can hide the new tender type per store without a second API. The principal point is that the enum is a promise to devices that do not ship on my schedule.

**Follow-up:** How does this interact with `@Enumerated(EnumType.ORDINAL)` if someone mapped the same type in JPA?

**Weak answer:** "I change the enum and everyone regenerates their client the same day."

## Q9. Debug a client that sometimes creates two orders for one tap.

**What the interviewer is probing:** A debugging narrative for lost responses.

**Sample answer:** I ask whether the client generated one idempotency key per tap and reused it on timeout, or generated a new key per HTTP attempt. A new key on every retry is two creates, and the server is behaving. I look at access logs for two POSTs with different keys and the same client timestamp and basket hash, and at one POST that received `201` after the client had already given up. That is a classic lost response: the server committed, the device timed out, the retry used a new key.

The fix is client reuse of the key for that gesture, and server storage of the response so the retry returns the original order id. I also check Apigee and the load balancer for a retry that is invisible to the device. I check that create is not behind a GET cache. I look at the unique constraint violations; if I see none, the keys were different. I do not "fix" this by making create return `200` faster without durability. If the order insert and the outbox commit are slow, I still return only after the local commit, and I set the client timeout above that budget. The evidence I want is two log lines, the keys, and the order ids, not a guess about Java GC.

**Follow-up:** How do you clean up the duplicate orders that already exist?

**Weak answer:** "Tell the stores not to double-tap."

## Q10. What does a principal-level API review comment look like?

**What the interviewer is probing:** Taste and priorities under time.

**Sample answer:** I comment on behavior a device will hit, not on annotation style. I ask for an idempotency story on every non-safe POST that moves money or stock. I ask which status a retry sees, and where that is written in OpenAPI. I reject entity binding, unbounded lists, and store id taken only from the query string. I ask how a timeout is distinguished from a decline in the body, because the UI prints different words and the reconciliation job branches on it. I check that error `code` values are stable and that `detail` is safe to show.

I ask what happens to version `/v1` when this change ships. I ask which fields are optional and what the server does when an old client omits them. I do not block the review for a missing description string on an obvious field, and I do block it for a missing timeout on the Feign client that this endpoint calls. The comment references the cashier gesture: scan, pay, retry on a bad radio. If the design works on a perfect network and fails on a store network, it is not done. That is the bar I state in the interview, with one concrete example rather than a checklist of twenty items.

**Follow-up:** Which comment would you waive for an internal-only endpoint?

**Weak answer:** "LGTM if the controller has `@RestController` and Swagger loads."
