# Kubernetes core — interview questions

## 1. What is a Pod, and why do you usually not create one directly?

**Interviewer intent:** Desired state versus a single process.

**Strong answer:** A Pod is one or more containers that share network and volumes and are scheduled together. It is the unit the kubelet runs. I declare a Deployment, which owns a ReplicaSet, which owns Pods, so a crash or a node loss is replaced to the replica count. A bare Pod is not rescheduled by a Deployment controller if the node disappears. Sidecars are extra containers in the same Pod, not a second Deployment hiding beside the app.

**Follow-up:** When would you choose a StatefulSet instead?

**Weak answer to avoid:** "A Pod is a Docker container, and I SSH in to restart it."

## 2. How does a Service find Pods?

**Interviewer intent:** Labels, endpoints, and stable DNS.

**Strong answer:** A Service selects Pods by labels and exposes a stable DNS name and virtual IP. Ready Pods are the endpoints. Callers inside the cluster use that DNS name, never a Pod IP. If the selector does not match the Deployment's labels, the Service has no endpoints and the app looks "up" while every call fails. Readiness, not liveness, decides membership.

**Follow-up:** What is the DNS name of a Service in another namespace?

**Weak answer to avoid:** "The Service load-balances all containers on the node."

## 3. How should a Spring Boot process use liveness, readiness, and startup probes?

**Interviewer intent:** The classic Java-on-Kubernetes failure: probing the database for liveness.

**Strong answer:** I enable actuator health probes. Startup covers a slow JVM boot so liveness does not kill it. Liveness hits a cheap local endpoint and means "restart this process"; it must not call the database or Kafka. Readiness means "send traffic," and it can fail when a mandatory dependency is down. Confusing them either drops traffic forever or restarts every Pod when a dependency blips.

**Follow-up:** Readiness fails on every Pod because the database is down. What does the user see, and why is that better than liveness?

**Weak answer to avoid:** "I point all three probes at a health check that pings every downstream."

## 4. What is the difference between a resource request and a limit?

**Interviewer intent:** Scheduling versus enforcement, CPU versus memory.

**Strong answer:** The request is what the scheduler reserves and, for CPU, the baseline share under contention. The limit is the ceiling. CPU limit throttles via the CFS quota and shows up as latency. Memory limit does not throttle; crossing it OOMKills the container. I set requests from measured steady state so the node is not overpacked and the HPA is not lying. A Pod with no requests is the first candidate for eviction under pressure.

**Follow-up:** CPU is flat at the limit and p99 is high. What do you change?

**Weak answer to avoid:** "Request and limit should always be equal, and equal to `-Xmx`."

## 5. How do you size memory for a JVM in a container?

**Interviewer intent:** Heap is not the container.

**Strong answer:** The limit must cover heap, metaspace, thread stacks, code cache, direct buffers, and native memory. Setting `-Xmx` equal to the memory limit is a reliable way to get exit 137. I leave headroom, or I use the JVM's cgroup awareness with `MaxRAMPercentage` low enough that non-heap fits. I watch the container working set, not only the heap gauge. After an OOMKilled I read the previous termination reason before I only raise the number.

**Follow-up:** Metaspace grew after a deploy that added libraries. Where does that show up?

**Weak answer to avoid:** "Set `-Xmx` to the limit so we use all the memory we pay for."

## 6. How does a Deployment rollout avoid dropping traffic?

**Interviewer intent:** Surge, readiness, and rollback.

**Strong answer:** `RollingUpdate` starts new Pods within `maxSurge` and only then terminates old ones, respecting `maxUnavailable`. New Pods must become Ready before they take traffic. If they never do, the rollout sticks, which is safer than shifting users. I keep `terminationGracePeriodSeconds` long enough for Spring's graceful shutdown, and I can `rollout undo` to the previous revision. `Recreate` is for the rare case where two versions must not overlap.

**Follow-up:** What race does a short `preStop` sleep address?

**Weak answer to avoid:** "Kubernetes waits until no users are online, then restarts."

## 7. How should Spring Boot shut down on Kubernetes?

**Interviewer intent:** Drain behavior, which separates people who have been paged.

**Strong answer:** On SIGTERM I need `server.shutdown=graceful` so the server stops accepting and finishes in-flight requests, with the lifecycle timeout under the Pod grace period. Readiness should fail first so the Service stops routing. A `preStop` delay covers the endpoint-propagation race. Shutdown hooks must leave the Kafka group or close RabbitMQ channels. If the grace period is too short, SIGKILL cuts live requests.

**Follow-up:** A consumer is mid-transaction when SIGTERM arrives. What do you want to happen?

**Weak answer to avoid:** "The JVM exits immediately and the load balancer will retry."

## 8. Where does configuration live, and what does a Secret actually protect?

**Interviewer intent:** Twelve-factor plus a realistic view of Secrets.

**Strong answer:** One image is promoted; environment-specific config comes from ConfigMaps and Secrets, read as env or files, which Spring already binds over `application.yml`. A Secret is base64 in the API, not encryption. Protection is RBAC, etcd encryption at rest, and preferably an external secret manager. I do not commit secrets or bake them into the image. Changing an env-based ConfigMap does not refresh a running JVM; I roll the Deployment.

**Follow-up:** How do you rotate a database password without a long double-write window?

**Weak answer to avoid:** "Kubernetes encrypts Secrets, so they are safe in git if the file is a Secret manifest."

## 9. What does an HPA scale on, and what can it not fix?

**Interviewer intent:** Autoscaling limits.

**Strong answer:** The HPA sets replica count from a metric, often CPU versus request, or a custom metric such as lag or in-flight requests. It needs honest requests and a service that is stateless enough to add Pods. It does not fix a memory leak, a hot lock, or a database pool that is already at its max. Scaling out a broken query multiplies load on the dependency. I pair it with a max replica cap and a metric that matches the bottleneck.

**Follow-up:** Why is CPU a poor signal for a Kafka consumer?

**Weak answer to avoid:** "HPA will keep the service fast no matter what the code does."

## 10. What is a namespace for, and what is it not?

**Interviewer intent:** Isolation claims you must not oversell.

**Strong answer:** A namespace scopes names, RBAC, quotas, and network policy. It is the right boundary between teams or environments on one cluster. It is not a security boundary against cluster-admin, and it does not cap CPU unless I add quotas and requests. I still want default-deny network policy between namespaces. I do not treat namespace separation as equivalent to separate accounts for hostile tenants.

**Follow-up:** Two teams share a cluster. What do you put in place on day one?

**Weak answer to avoid:** "Namespaces are separate clusters, so they cannot talk."
