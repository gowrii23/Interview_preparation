# React state and performance — interview questions

## 1. What causes a React component to rerender?

**Interviewer intent:** See if you can debug a slow screen from first principles.

**Strong answer:** A component rerenders when its state updates, when a context it consumes changes, or when its parent renders it and it does not bail out. A rerender reruns the function; it does not automatically rewrite the whole DOM. The usual waste is state lifted too high, a context value that is a new object every time, or an effect that sets state and retriggers itself. I colocate state and then profile before I add memo.

**Follow-up:** Does a parent rerender force the child's DOM to be recreated?

**Weak answer to avoid:** "Any state change rerenders the entire application."

## 2. How do you separate server state from client state?

**Interviewer intent:** This is the design question. They want a cache policy, not a library slogan.

**Strong answer:** Client state is UI-owned: open panels, unsent drafts, selected tabs. Server state is a cache of orders, prices, or entitlements. I use a query library such as TanStack Query so a query key addresses the cache, in-flight calls dedupe, and a mutation invalidates the keys it changes. The server stays the source of truth. I do not copy the payload into context so "everyone can read it," because invalidation then becomes folklore.

**Follow-up:** What do you do with the cache when a POST returns 409?

**Weak answer to avoid:** "I load everything in a top-level `useEffect` and pass it down."

## 3. When would you lift state, and when is prop drilling acceptable?

**Interviewer intent:** Judge taste. Over-globalizing is the failure mode of senior people too.

**Strong answer:** I lift client state to the nearest parent that has two readers. Passing an id through two layers is fine. I reach for context when many distant readers need a stable value such as locale. I do not introduce a global store to avoid typing props. Server data is not lifted; it is queried by key where it is used, so a refetch updates every subscriber.

**Follow-up:** How would two components on different routes share a draft?

**Weak answer to avoid:** "All state goes in one store so we never drill props."

## 4. Why does React favor composition over inheritance?

**Interviewer intent:** Connect your Angular or OOP background to the React way without nostalgia.

**Strong answer:** A base component class hides data flow and fights hooks. I share behavior with a hook and share structure by composing children, slots, or a layout route with an outlet. A specialized page wraps small pieces instead of overriding protected methods. The result is a dependency list I can read, which is what I want in review.

**Follow-up:** How would you reuse a paged-table behavior across three screens?

**Weak answer to avoid:** "I create `BasePageComponent` and extend it for every screen."

## 5. When do you code-split, and when do you virtualize a list?

**Interviewer intent:** See if you apply techniques only when the constraint exists.

**Strong answer:** I split at routes and at heavy panels that most users never open, with `lazy` and `Suspense`, and I keep `index.html` uncached so clients do not pin a dead shell. I do not split every component. I virtualize when a screen truly mounts hundreds or thousands of rows. Before that I paginate or cursor the API. Virtualizing a page of twenty rows adds accessibility cost for no win.

**Follow-up:** What breaks if the HTML shell is cached for a year?

**Weak answer to avoid:** "Virtualize every table on day one so we are future-proof."

## 6. Why is premature memo harmful?

**Interviewer intent:** You should sound like someone who has cleaned up a memo-covered codebase.

**Strong answer:** `memo`, `useMemo`, and `useCallback` pay off when a profile shows a costly child or when identity must stay stable for a dependency. Applied everywhere, they obscure flow, and a wrong dependency list freezes a stale closure. A new object prop still defeats `memo`. I measure, then memoize the hot boundary, and I remove memo that does not change the profile.

**Follow-up:** A memoized child still rerenders. What do you check first?

**Weak answer to avoid:** "Memo makes renders free, so it is always an improvement."

## 7. What is a BFF, and why would you put one in front of React?

**Interviewer intent:** Principal scope: the UI is not allowed to become a distributed-systems client.

**Strong answer:** A backend-for-frontend is a server edge shaped for one client. It aggregates internal services, enforces the user's authorization, sets timeouts, and returns a DTO the page can render. The browser does not fan out to five internal hosts or learn their network. I would rather evolve the BFF contract than teach the SPA each microservice's error dialect.

**Follow-up:** How do you stop the BFF from becoming a second monolith with business rules copied from the domain services?

**Weak answer to avoid:** "The React app calls each microservice directly with a service account."

## 8. How should OpenAPI govern the React client?

**Interviewer intent:** Contract discipline, the same standard you already apply to backend consumers.

**Strong answer:** The spec is the source of truth. CI generates the TypeScript client, and the UI build fails on a breaking change. Changes are additive, with explicit deprecation. Hand-written interfaces next to a generated client will drift and show up as `undefined` in production. The same pull request, or a contract PR that merges first, carries the spec and the screen.

**Follow-up:** A field changes type from string to object. What do you do for clients already shipped?

**Weak answer to avoid:** "We keep a shared wiki of the JSON and update it when someone notices."

## 9. Where should tokens live in a browser app?

**Interviewer intent:** Security. They will fail you for secrets in the bundle.

**Strong answer:** The bundle is public. No client secret, no signing key, no long-lived token in source or in an env file that the bundler inlines. My preferred shape is authorization code with PKCE, refresh token in an `HttpOnly`, `Secure`, `SameSite` cookie set by the BFF, and the access token kept server-side or briefly in memory. `localStorage` is readable from any XSS. Hiding a button is not authorization; the API checks again.

**Follow-up:** What threat remains if the session is a cookie?

**Weak answer to avoid:** "I put the JWT in `localStorage` so refresh is easy."

## 10. A list page is slow. What is your order of investigation?

**Interviewer intent:** A structured performance story, not a pile of tricks.

**Strong answer:** I confirm the API: payload size, pagination, and latency. Then I look at whether one keystroke rerenders the whole page, whether a context provider invalidates everyone, and whether a request waterfall exists. Only then do I open the profiler and consider memo, splitting, or virtualization. I also check that we are not downloading the full ledger to filter it in the browser. Most "React is slow" reports I expect are data-fetch or state-placement bugs.

**Follow-up:** The profiler says the row component is cheap, but p95 is two seconds. Where do you look?

**Weak answer to avoid:** "I add `useCallback` everywhere and ship."
