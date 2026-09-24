# React fundamentals — interview questions

Audience: a senior backend lead who can read JavaScript and is not a React specialist. Sample answers are what you should be able to say out loud.

## 1. What is a React component, and how is JSX different from HTML?

**Interviewer intent:** Check that you see React as a rendering model, not as "HTML in JavaScript."

**Strong answer:** A component is a function that takes props and returns a description of UI. JSX compiles to `createElement` calls; it is not a string of HTML. Attributes differ (`className`, `htmlFor`), events are camelCase, and a capitalised name means a component while a lowercase name means a DOM node. The function should be pure with respect to props and state: it returns elements and does not touch the network or the DOM during render.

**Follow-up:** Why must component names start with a capital letter?

**Weak answer to avoid:** "JSX is HTML that React pastes into the page."

## 2. How do props and state differ, and when do you lift state?

**Interviewer intent:** See whether you can place data the way you place ownership in a service.

**Strong answer:** Props are read-only inputs from the parent. State is owned by the component and updated through a setter, which schedules a new render. I lift state when two children must share a fact, and I leave it local when only one subtree needs it. If a value is fully determined by props or state, I compute it during render instead of copying it into another `useState` inside an effect.

**Follow-up:** What goes wrong if you set state from props with `useEffect`?

**Weak answer to avoid:** "State is for variables and props are for constants."

## 3. What does reconciliation do, and why do list keys matter?

**Interviewer intent:** Test identity and update semantics, a common production bug.

**Strong answer:** Each render produces an element tree. React walks it against the previous tree. Same type in the same position means update in place; a different type means unmount and mount, which drops state. In a list, `key` tells React which child is which across renders. I use a stable business id. An index key attaches state to the position, so reordering or inserting rows mixes input values and component state.

**Follow-up:** What happens to state if you change a component's `key`?

**Weak answer to avoid:** "Keys are for CSS, and the index is always fine because React is fast."

## 4. When do you use `useEffect`, and what belongs in the dependency list?

**Interviewer intent:** Separate side effects from render, and catch stale-closure mistakes.

**Strong answer:** `useEffect` synchronizes React with something outside it after paint: fetch, subscription, timer, or a non-React library. I return a cleanup that aborts the request or unsubscribes. Every reactive value the effect reads goes in the dependency array. An empty array means mount and unmount only. I do not use an effect to derive state I can compute while rendering, and I do not fetch directly in the component body, because that runs during render and can loop.

**Follow-up:** How do you stop a slow response from overwriting a newer search?

**Weak answer to avoid:** "I put all my logic in `useEffect` so the component stays clean."

## 5. What are `useMemo`, `useCallback`, and `useRef` for?

**Interviewer intent:** See restraint. Principals who memo everything signal inexperience.

**Strong answer:** `useMemo` caches a computed value. `useCallback` caches a function identity so a memoized child or an effect dependency does not change every render. Both are useless if the dependencies change every time, and they are noise if nothing expensive depends on them. `useRef` is a mutable box that does not rerender when you write `.current`. I use it for DOM nodes, timers, and abort controllers, not as a secret state store for data the UI must show.

**Follow-up:** When is `React.memo` actually worth it?

**Weak answer to avoid:** "Wrap every handler in `useCallback` or the app will be slow."

## 6. Explain a controlled input.

**Interviewer intent:** Confirm you can talk about forms without a tutorial script.

**Strong answer:** A controlled input's `value` comes from React state and its `onChange` writes that state. React is the source of truth, so I can validate, disable submit, or mirror the value into the query string. An uncontrolled input keeps the value in the DOM until I read a ref. I default to controlled fields for anything the user edits that the application must inspect before submit, and I always attach a label.

**Follow-up:** How would you prevent a double submit?

**Weak answer to avoid:** "I read `document.getElementById` on submit because it is simpler."

## 7. When is context the right tool, and when is it the wrong one?

**Interviewer intent:** Stop you from treating context as a global database.

**Strong answer:** Context passes a value to a subtree without prop threading. It fits theme, locale, and the current user object. It is the wrong cache for server data, because any change rerenders all consumers. I split fast-changing values from stable ones, and I memoize object values I pass to the provider. Server payloads belong in a query cache keyed by the request, not in a single app-wide context.

**Follow-up:** What happens if the provider value is an object literal created during render?

**Weak answer to avoid:** "Context replaces Redux and props, so I put the whole API response there."

## 8. What does an error boundary catch, and what does it miss?

**Interviewer intent:** Check failure handling, which is where backend leads should be strong.

**Strong answer:** An error boundary is a class component with `getDerivedStateFromError` that renders a fallback when a child throws during render. It does not catch event-handler errors, async errors, or errors in the boundary itself. I still try/catch those paths, show an alert with a correlation id, and keep the boundary small so one widget cannot blank the shell. Async failures set error state; they do not magically hit the boundary.

**Follow-up:** How would you log this so it joins a Spring trace?

**Weak answer to avoid:** "A try/catch around JSX catches all errors, including fetch."

## 9. What accessibility basics do you insist on in review?

**Interviewer intent:** See if UI quality includes people who do not use a mouse.

**Strong answer:** Native `button` and `a` elements, a label on every control, status not indicated by color alone, and focus moved into a dialog and back to the trigger. I reject a clickable `div` unless there is a strong reason and full keyboard support. The empty and error regions should be in the accessibility tree, for example with `role="alert"` on a failure. This is part of the definition of done, not a later polish pass.

**Follow-up:** Why is "click here" a bad link name?

**Weak answer to avoid:** "Accessibility is a specialist pass after we ship."

## 10. How would you review a React pull request as the backend owner of the API?

**Interviewer intent:** Principal signal: contract, failure, and security, not pixel taste.

**Strong answer:** I check that the client matches the published contract: paths, enums, and error body. I look for stable keys, effect cleanup, and loading, empty, and error UI. I look for a double-submit on commands. I look for tokens or secrets in the bundle. I ask what a 409 and an empty page do. I do not approve a screen that only renders the happy path, even if the JSX looks tidy.

**Follow-up:** What would you require before the frontend and backend merge independently?

**Weak answer to avoid:** "I trust the UI developer on the contract and only review the Java side."
