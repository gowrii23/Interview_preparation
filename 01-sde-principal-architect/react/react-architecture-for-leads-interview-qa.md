# React architecture for leads — interview questions

## 1. How would you structure a React codebase owned by several teams?

**Interviewer intent:** Boundary design, mapped to how you already split backend services.

**Strong answer:** I slice by business capability, aligned with bounded contexts: each feature folder owns its pages, components, and API adapter. A design system and a shared HTTP client are the only wide dependencies. Features do not import each other's internals. The route table is the public API, and shareable filters live in the query string so a refresh and a pasted link match. Ownership of a folder is explicit.

**Follow-up:** How do two features share a customer summary without coupling their folders?

**Weak answer to avoid:** "One `components` directory and one `services` directory for the whole company."

## 2. What do you put in an API adapter versus a view component?

**Interviewer intent:** See a testable seam, not a rigid container pattern.

**Strong answer:** The adapter speaks HTTP and returns typed domain objects or typed errors. The page decides which queries and commands to run. Presentational components take data and callbacks and do not call fetch. I map problem+json to a small error type once, so JSX never branches on an Axios shape. I do not wrap every button in three files; the seam exists where the network crosses.

**Follow-up:** How do you test the view without a backend?

**Weak answer to avoid:** "Each component calls fetch so it stays independent."

## 3. How do you handle a command that must not run twice?

**Interviewer intent:** Connect UI behavior to idempotency, a backend strength.

**Strong answer:** I create an idempotency key when the user starts the action and reuse it on retry. I disable the control while the mutation is in flight. The server still treats that key as the consistency boundary, because the browser will retry, crash, or time out. A new key per HTTP attempt is how you double-charge. The error path tells the user whether it is safe to retry.

**Follow-up:** The response is lost after the server committed. What does the user see?

**Weak answer to avoid:** "The button spinner is enough; the API can insert again."

## 4. How should authentication work between the SPA and your APIs?

**Interviewer intent:** You must not trust the browser as a security boundary.

**Strong answer:** The SPA proves the user through an authorization-code flow, preferably via a BFF that holds the refresh token in an `HttpOnly` cookie. Route guards call a session endpoint; they do not treat a decoded JWT as authorization. Every API enforces its own checks. Roles hidden in the UI are a convenience. XSS and a stolen local token are the threats I design against first.

**Follow-up:** How does CSRF show up in this design, and what do you do about it?

**Weak answer to avoid:** "I decode the JWT in React and hide routes the token does not allow."

## 5. What is your testing strategy for a feature slice?

**Interviewer intent:** Investment judgment. They do not want a snapshot religion.

**Strong answer:** Adapter tests assert the contract against OpenAPI examples. Component tests drive behavior: filter, request, empty state, error alert. A few end-to-end journeys cover the happy path and an auth failure. I do not keep large snapshots; they fail on harmless markup and miss broken requests. The correlation id is asserted present on outbound calls so production debugging is not an afterthought.

**Follow-up:** What would you refuse to mock?

**Weak answer to avoid:** "Snapshot every component on each commit."

## 6. How do you release a SPA safely?

**Interviewer intent:** Cache and rollback, which backend candidates often ignore.

**Strong answer:** Hashed JS and CSS are immutable and cached long. `index.html` is not, or clients keep a shell that points at deleted chunks. The ingress or CDN enforces that split. A feature flag at the BFF can dark-launch a route. Rollback is "serve the previous index and assets," coordinated with any API that the new UI required. I do not bake the API host into a rebuild per environment if configuration can come from the edge.

**Follow-up:** The API deployed a breaking change first. What fails, and how do you stage it?

**Weak answer to avoid:** "We cache the whole site for a week, including HTML."

## 7. How do you stop one failing widget from taking down the shell?

**Interviewer intent:** Bulkheads, applied to UI.

**Strong answer:** I place error boundaries around regions, not only at the root. Recommendations can fail while checkout still works. Each remote region has a timeout budget at the BFF that is shorter than the user will wait and consistent with downstream deadlines. The fallback includes a reference id. I do not share one giant try/catch across the page, and I do not let a slow optional call block the primary command.

**Follow-up:** Where do you draw the line between a region failure and a full-page failure?

**Weak answer to avoid:** "One error boundary at the root is the architecture."

## 8. How would you introduce a design system without freezing product work?

**Interviewer intent:** Lead judgment: adoption path, not a component-library lecture.

**Strong answer:** I start with the primitives new screens need: button, input, dialog, focus behavior. Old screens migrate when they are touched. Tokens for color and spacing land first so new CSS stops inventing palettes. I do not rewrite the app to adopt a library. Contribution rules are small: accessibility is required, and feature folders do not fork the button.

**Follow-up:** A team wants a one-off button. How do you decide?

**Weak answer to avoid:** "Stop feature work for a quarter and rebuild every screen on the new kit."

## 9. What belongs in the URL, and what must never be there?

**Interviewer intent:** State placement with security mixed in.

**Strong answer:** Shareable UI state belongs in the query string: tab, filter, page cursor if it is not sensitive. A refresh should restore it. Tokens, national ids, and anything you would not put in an access log do not belong in the URL, because URLs leak via history, referrers, and logs. Ephemeral drafts can stay in memory. Secrets never belong in the route or the bundle.

**Follow-up:** How do you handle a cursor that is opaque but not secret?

**Weak answer to avoid:** "I put the access token in the query so deep links stay authenticated."

## 10. What do you reject in a React pull request as the tech lead?

**Interviewer intent:** A concrete quality bar.

**Strong answer:** I reject fetch during render, index keys on reorderable lists, refresh tokens in `localStorage`, hand-copied DTOs beside a generated client, and screens with no empty or error state. I require labels on controls and the OpenAPI change with the UI change. I ask who owns the folder after the author leaves the project. Taste comments on naming come second; contract and failure come first.

**Follow-up:** The author says the error state is a fast-follow. What do you do?

**Weak answer to avoid:** "I approve if the happy path demo looks right."
