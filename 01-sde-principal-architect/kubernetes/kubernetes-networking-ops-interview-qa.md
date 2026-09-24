# Kubernetes networking and operations — interview questions

## 1. When do you use ClusterIP, NodePort, and LoadBalancer?

**Interviewer intent:** Exposure should be deliberate.

**Strong answer:** ClusterIP is the default for service-to-service calls: a virtual IP and DNS inside the cluster. NodePort publishes a port on every node and is mainly for labs or constrained networks. LoadBalancer asks the platform for an external balancer and should be reserved for a few edges, usually the ingress controller, not one per microservice. Internal traffic stays on ClusterIP even if the product is public.

**Follow-up:** Why is one LoadBalancer per service expensive operationally?

**Weak answer to avoid:** "Every Deployment needs a LoadBalancer so users can reach it."

## 2. What is an Ingress, and what has to be running for it to work?

**Interviewer intent:** The object versus the controller.

**Strong answer:** An Ingress is a set of host and path rules, plus TLS configuration, that points at Services. It does nothing until an ingress controller is deployed and watching those objects. I terminate TLS there unless I have a reason to re-encrypt, and I align proxy timeouts with the application's deadlines. A 60-second proxy in front of a 10-second server produces opaque client errors. The Gateway API is the successor shape on platforms that have moved; the idea is the same: routing is data, a controller implements it.

**Follow-up:** A new host rule returns 404 from the controller. What do you check?

**Weak answer to avoid:** "Ingress is a load balancer you get automatically with every Service."

## 3. How would you restrict which Pods can call the payments Service?

**Interviewer intent:** NetworkPolicy as part of the design, not a later hardening project.

**Strong answer:** I start from default-deny in the namespace and add policies that allow the gateway or BFF labels to reach payments on the right port. Selection is by labels, so the labels must be stable. I do not rely on "nobody knows the DNS name." Without a policy the cluster network is flat. I test the deny path, not only the allow path.

**Follow-up:** A policy allows the namespace but you wanted only one Deployment. What selector did you get wrong?

**Weak answer to avoid:** "Security groups outside the cluster are enough; Pods are trusted."

## 4. Explain PV and PVC for a stateless Java service that sometimes wants a disk.

**Interviewer intent:** You should usually decline the disk.

**Strong answer:** A claim asks for size and access mode; a volume is the bound storage, often dynamically provisioned by a StorageClass. `ReadWriteOnce` attaches to one node and does not serve two replicas writing the same files. Most Spring services should keep state in a database or broker and use no PVC. `emptyDir` is scratch and dies with the Pod. If I truly need a disk per instance, I use a StatefulSet and a claim template, plus a backup story.

**Follow-up:** Two replicas, one PVC, ReadWriteOnce. What fails?

**Weak answer to avoid:** "I mount a persistent disk on a Deployment so logs survive."

## 5. The Pod is in CrashLoopBackOff. What do you do?

**Interviewer intent:** A real debug sequence.

**Strong answer:** The process is exiting, and the kubelet is backing off. I `describe` the Pod and read Events, then `logs --previous` for the crashed instance. Exit 137 often means OOM. A Spring failure on startup is often config, secrets, or a migration. I separate this from ImagePullBackOff and from a live process whose liveness probe is misconfigured. I do not raise replicas to "get ahead of the crash."

**Follow-up:** Logs are empty and the previous container lasted 200 milliseconds. What next?

**Weak answer to avoid:** "Delete the Pod until it stays up."

## 6. How do CPU throttling and OOMKilled show up differently?

**Interviewer intent:** You must not treat both as "raise the limit."

**Strong answer:** A CPU limit throttles. Latency climbs, the process stays up, and threads are not necessarily blocked on locks. If the node has spare CPU, I raise or drop the limit and keep an honest request. Memory does not throttle. Past the limit the container is OOMKilled. I check working set versus heap so I do not miss native memory. Raising memory without a profile only delays a leak.

**Follow-up:** How does a missing startup probe imitate a crash loop?

**Weak answer to avoid:** "Throttling and OOM are the same thing: the Pod ran out of resources."

## 7. How do you apply twelve-factor config on Kubernetes for Spring Boot?

**Interviewer intent:** One image, external config, predictable reload.

**Strong answer:** The image contains the app and non-secret defaults. URLs, pool sizes, and credentials come from the environment or mounted files via ConfigMaps and Secrets. Spring binds environment variables over packaged yaml. I promote the same image. Env changes apply on rollout, because the JVM will not see a replaced environment. I log to stdout and keep probes cheap and local.

**Follow-up:** Someone wants a separate image per environment. What do you push back with?

**Weak answer to avoid:** "We rebuild the image with the prod properties baked in."

## 8. Readiness is failing, liveness is fine. How do you explain the incident?

**Interviewer intent:** You should welcome this outcome in some failures.

**Strong answer:** The process is alive and the kubelet is not restarting it, but the Service has removed it, so callers see connection errors or land on other Ready Pods. That is correct when a mandatory dependency is down or warm-up is incomplete. If every replica is unready, the user-facing error is at the ingress, and restarting Pods would make it worse. I fix the dependency or the probe that is too strict.

**Follow-up:** When would a dependency failure belong in liveness instead?

**Weak answer to avoid:** "Unready means Kubernetes is broken and I should kill the node."

## 9. How do you watch a rollout that might be bad?

**Interviewer intent:** Operational control.

**Strong answer:** I watch `rollout status`, Ready counts, and error rate or a synthetic check before the old ReplicaSet is fully gone. `maxUnavailable` keeps capacity. If new Pods never go Ready, traffic stays on the old ones and I undo. I do not delete the Deployment to "restart." After undo I confirm the previous Pod template is the one serving. Database migrations that are incompatible with the old code block this strategy, so migrations have to be backward compatible.

**Follow-up:** Why does a backward-incompatible migration break a rolling deploy?

**Weak answer to avoid:** "I wait for user complaints, then rebuild."

## 10. What do you require before a Java service is allowed in production on the cluster?

**Interviewer intent:** A bar, not a tool list.

**Strong answer:** Requests and a memory limit with heap headroom, startup plus separate liveness and readiness on actuator groups, graceful shutdown inside the grace period, config and secrets outside the image, stdout logs, and a Service that is ClusterIP unless it is an edge. A rollout must stick if readiness fails. I want a known crash signal (`OOMKilled` versus config) documented for the on-call. Namespace quotas and a network policy are part of the platform ticket, not optional extras the team might skip.

**Follow-up:** Which of those would you waive for an internal batch job, and why?

**Weak answer to avoid:** "If it runs on my laptop in Docker, it is ready for the cluster."
