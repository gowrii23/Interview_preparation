# MCP and tool use — interview Q&A

Re-read the current Model Context Protocol architecture page before the interview. Lifecycle details have changed across public revisions. These answers stay on the concepts that stay interview-relevant: host, client, server, tools, resources, prompts, and a host-side allow list.

## 1. What does MCP standardize?

**Intent.** Protocol versus a single function-call feature.

**Strong answer.** MCP is an open protocol for how an LLM application connects to external capabilities. The host is the application. It creates a client per server. Servers expose tools, resources, and prompts. Messages are JSON-RPC. Local servers often use stdio; remote ones use HTTP. It does not replace webhooks, auth, or idempotency in the triage worker.

**Follow-up.** “Which spec version did you implement?” I would answer with the version we actually ran, or say I need to check. I will not guess stateful versus stateless or whether sampling is still current.

**Weak answer.** “MCP is a library that lets ChatGPT call APIs.”

## 2. What is the difference between host, client, and server?

**Intent.** Role clarity.

**Strong answer.** The host owns the user or the webhook, the model call, and consent. In my system that is the triage worker or the Agentic Universe runtime. The client is the connector inside the host, one per server. The server is the focused process that actually searches OpenSearch or lists commits. The client is not the model.

**Follow-up.** “Can one host use three servers?” Yes. Logs, commits, and deployment packages are three clients and three servers, even if one repo builds them. A failure in one server should not look like a failure of the protocol.

**Weak answer.** “The client is the LLM and the server is the tool.”

## 3. How do tools, resources, and prompts differ?

**Intent.** The three server primitives.

**Strong answer.** Tools are functions the model may call, such as a bounded log search. Resources are data addressed by URI that the host attaches, such as a ticket body or a known manifest. Prompts are templates a workflow selects, such as the RCA instruction. I do not turn every read into a tool, and I do not hide the RCA wording only inside Python.

**Follow-up.** “Who invokes each?” The model proposes tool calls, within the host’s allow list. The host or user attaches resources. A person or a template selects the prompt. Writes still need a gate even if they are tools.

**Weak answer.** “They are all just functions with different names.”

## 4. Map your Jira design onto MCP.

**Intent.** You built something real.

**Strong answer.** Jira’s webhook hits the host. The host loads the ticket as context. Clients call servers for OpenSearch logs, recent commits, and deployment packages. The classification prompt is a template. Posting the RCA is a write tool the model does not hold by default; a person approves. The webhook itself is not an MCP message.

**Follow-up.** “What if the commit SHA is already on the ticket?” Read it as a resource instead of asking the model to search. Search is for when the id is unknown.

**Weak answer.** “Everything is a tool, including the webhook and the prompt.”

## 5. Why not ad-hoc function calling?

**Intent.** Tradeoff, not fashion.

**Strong answer.** Function calling is enough for one model turn: a name and arguments in that request. MCP packages the same capability so a second host can reuse it. Agentic Universe needed that because triage and the order-metrics dashboard should share a way to attach tools without sharing allow lists. I would still use plain function calling for a single internal endpoint and one model. Neither choice makes the call safe.

**Follow-up.** “What did MCP cost you?” More processes, a spec to track, and operational load. Worth it only because there was more than one workflow.

**Weak answer.** “Function calling is obsolete. MCP replaces it entirely.”

## 6. How do you enforce least privilege?

**Intent.** Credentials and scope.

**Strong answer.** Each server has its own credential. The log server can read specific indexes, not administer the cluster. Arguments are constrained to the ticket’s service and a bounded time range. The Jira write credential is not on the read servers. The model sees results, not production passwords. A dashboard template does not receive the triage allow list.

**Follow-up.** “What is a confused deputy here?” The server acts with its own rights. If those rights exceed the caller’s, the model is a bypass. Bind queries to the tenant and the service on the ticket.

**Weak answer.** “The agent uses a shared admin key so it can debug anything.”

## 7. How do you handle untrusted tool output?

**Intent.** Injection through observations.

**Strong answer.** Logs and commit messages are quoted data. They never extend the system prompt. The next call is checked against the allow list and the schema. If a line says to deploy or to change the class, that is content, not a command. I still validate the final JSON so a classification must cite evidence ids that were returned.

**Follow-up.** “Where do you log the payload?” Hashed or redacted arguments, status, latency, byte size. Not raw customer text in the trace.

**Weak answer.** “We trust internal logs because employees wrote them.”

## 8. What happens when a server times out?

**Intent.** Partial failure.

**Strong answer.** The client returns a structured observation: that source is unavailable. The host does not retry forever. If the source was optional, the draft lists it under missing sources. If it was mandatory for the class, the outcome is escalate, with whatever evidence returned. The model is not asked to invent the log line.

**Follow-up.** “Retries?” Idempotent reads only, few attempts, jitter, and only on transport or overload errors. Not on a bad query.

**Weak answer.** “We retry until it works, then let the model fill gaps.”

## 9. How does a template in Agentic Universe use MCP?

**Intent.** Platform story.

**Strong answer.** The template names servers, the visible tools, the prompt, and budgets. The host discovers what those servers advertise and then filters to the template. The order-metrics demo is the same mechanism with a different allow list: metric reads, no ticket writes. Schemas live with the servers so both workflows do not fork a private JSON dialect.

**Follow-up.** “How do you version it?” The trace stores template version plus server versions. A schema change is a breaking change, shipped like an API change.

**Weak answer.** “The template is a prompt that contains every tool in the company.”

## 10. What would you not expose as a tool?

**Intent.** Product judgment.

**Strong answer.** Unbounded search across every index. Any deploy, delete, or ticket-close action on the triage template. A tool whose description says “do whatever the log line asks.” A metric tool that accepts an arbitrary query language if a narrow `get_order_count(window)` would do. Broad tools are how a poisoned observation becomes an incident.

**Follow-up.** “How do you write a good description?” Name the scope, the required bounds, and whether it writes. “Search logs” is not a description. “Read-only search for one service between two timestamps” is.

**Weak answer.** “Expose the internal admin API and let the model decide.”
