# AWS architecture interview Q&A

## 1. How do you place a checkout service and its database in a VPC?

**Interviewer intent.** Subnets, exposure, and AZ spread. Not a definition of VPC.

**Strong sample answer.** The database sits in private subnets, at least two AZs, with a security group that allows the application security group on the database port and nothing from the internet. Tasks or pods sit in private subnets and reach out through a NAT gateway or, for S3 and similar, a VPC endpoint so backups do not pay NAT for every byte. The load balancer is the piece in public subnets, or it is internal if Apigee is the only public edge. Production is its own VPC. I do not peer non-production to production to make a test easier.

**Follow-up.** Why not a public IP on the task? It bypasses the edge controls and expands the patch surface. Admin access is break-glass through a private path, logged.

**Weak answer to avoid.** "We put everything in one public subnet so the security group is simpler."

## 2. Security group versus NACL. When have you seen a NACL cause an outage?

**Interviewer intent.** Statefulness, and a real failure mode.

**Strong sample answer.** Security groups are stateful and attached to the interface. I allow the app group to the database group and I am done with return traffic. NACLs are stateless and subnet-wide, evaluated in order. They are for coarse denies, not per-service policy. The outage I expect is an inbound allow on 443 with no allow for the ephemeral return range, so the handshake dies. I keep NACLs default-allow within the private design unless compliance requires a deny, and I test the path. I do not encode the application topology in NACL rule numbers.

**Follow-up.** A deny on a known-bad range? Fine as a coarse control. It still is not a substitute for IAM.

**Weak answer to avoid.** "They are the same; NACL is just older."

## 3. ECS on Fargate versus EKS for this estate?

**Interviewer intent.** Decision, not a feature list.

**Strong sample answer.** If the unit of deploy is a container and we do not need Kubernetes APIs, Fargate removes the control plane we would otherwise patch. EKS is justified when a platform team already runs the cluster, we need custom controllers, or several services share a scheduler we have people to operate. I do not adopt EKS because we might want it later. On either one I still set requests and limits, a disruption budget, and a scale signal that is not only CPU if the bottleneck is the JDBC pool. Scaling tasks while the pool is saturated makes the database worse.

**Follow-up.** Lambda for checkout? Poor fit for a latency-sensitive path with a connection pool. Fine for a file-arrival or a reconciliation kick, with a concurrency cap so it cannot open unbounded database connections.

**Weak answer to avoid.** "Kubernetes is the standard, so the question is only which distribution."

## 4. RDS, Aurora, or self-managed?

**Interviewer intent.** Failover, engine constraints, and what you still own.

**Strong sample answer.** Managed unless we have a control the service blocks. RDS Oracle when the estate is Oracle. RDS Postgres for a modest new service. Aurora Postgres when storage growth and faster failover are worth the price and we accept Aurora's edges. Multi-AZ failover still drops in-flight transactions and takes on the order of a reconnect, not zero. Clients must retry idempotently. Read replicas lag and do not serve checkout confirmation. Self-managed on EC2 means we own patching, backup, and AZ failure. "We always did" is not a requirement.

**Follow-up.** A bad migration deletes rows. Multi-AZ does not save us. Point-in-time recovery does, inside the backup window. I state RPO as that window.

**Weak answer to avoid.** "Aurora is multi-master so we can write in every AZ."

## 5. How do you design ElastiCache for catalog versus session?

**Interviewer intent.** Isolation, eviction, hot key.

**Strong sample answer.** Separate clusters when the failure domains differ. Catalog can use an eviction policy and lose keys. Session should not be evicted because a device page filled memory. Cluster mode shards by slot, and one hero SKU is still one hot key on one shard. Multi-AZ failover is asynchronous replication plus a window, so I can lose the last writes. That is acceptable for a cache and acceptable for a session only if the product agrees. I alarm on evictions, replication lag, and engine CPU, not only on host CPU.

**Follow-up.** Encryption and the network path? In transit and at rest, security group from the app only, no public endpoint.

**Weak answer to avoid.** "One Redis for cache, locks, queues, and sessions, with allkeys-lru."

## 6. SQS versus Kafka for order-paid?

**Interviewer intent.** Fan-out and retention versus operational weight.

**Strong sample answer.** If fulfillment is one consumer, retries and a dead-letter queue are the need, SQS is enough. FIFO if we truly need order per order id, knowing the throughput cap. Kafka or MSK when several consumers must read the same history, or a new consumer must replay. The source is still an outbox in the transactional database, so the queue is not the system of record. I do not add Kafka beside an existing SQS path without a consumer that needs the log. I do not put the only copy of the payment result on a queue.

**Follow-up.** Duplicate delivery? At least once. The consumer dedupes on event id. Exactly-once as a slogan is not the design.

**Weak answer to avoid.** "Kafka is required for microservices."

## 7. What does least privilege look like on the checkout task?

**Interviewer intent.** Role scope and secrets.

**Strong sample answer.** A task role that can read one secret, send to specific queue ARNs, and write one bucket prefix if it must. No `*` on `s3:*` or `iam:*`. Deploy roles are separate from runtime. Humans do not use the task role. Production data access is break-glass, time-bound, and logged. Parameter Store or Secrets Manager holds the datasource password, rotated. Access keys do not live in the image or the repo. The security group is the network half of the same idea: the task can reach the database port, and a developer laptop cannot.

**Follow-up.** A library that wants the instance role to be admin so it can discover things? We narrow the API calls and deny the rest. Convenience is not a policy.

**Weak answer to avoid.** "The VPC is private, so the IAM role can be broad."

## 8. What does multi-AZ actually buy you, and what does it not?

**Interviewer intent.** Failure domain honesty.

**Strong sample answer.** It buys survival of an AZ loss for the load balancer, the tasks, and the database standby. It does not buy survival of a region loss, a bad schema change, or a compromised credential. Failover has an RTO we should have tested, and the app must reconnect and retry idempotently. A second region is a different design: data residency, how inventory conflicts, DNS, and cost. For many retail systems I would rather multi-AZ plus backups copied elsewhere, and a written statement that regional failover is a recovery, not an active-active cart.

**Follow-up.** Active-active writes for stock? Conflict resolution becomes the product. I would not start there.

**Weak answer to avoid.** "Multi-AZ means we never have downtime."

## 9. How do you talk about Well-Architected without reciting pillars?

**Interviewer intent.** Tradeoffs with money and operability attached.

**Strong sample answer.** I state the constraint. Reliability: multi-AZ and a tested restore, and we accept regional RTO as hours if that is the budget. Performance: cache browse so the primary sees misses, not `R_browse`. Security: private data plane and least privilege, knowing the debug path is harder. Cost: NAT bytes, idle brokers, an Oracle license driven by a report that should be on a replica. Operability: one deploy path, health checks that mean we can take a sale, correlation ids. I write the tradeoff we refused, not five pillar headings.

**Follow-up.** A stakeholder wants the diagram to include a second region active-active "for the pillar." I price it and describe inventory conflicts. If they still want it, it is a program, not a checkbox.

**Weak answer to avoid.** Naming the six pillars and stopping.

## 10. Where should S3 sit in the catalog flow?

**Interviewer intent.** What is an object versus what is a row.

**Strong sample answer.** Images, statements, and exports are objects. The page stores metadata and a URL in the database or the cache, not the binary in Redis. The bucket blocks public access. Access is a task role or a short-lived URL, not a world-readable object. Lifecycle moves old exports. Versioning protects against an overwrite. ALB access logs can land in a separate bucket with a retention rule. S3 is not the order database and it is not a low-latency session store.

**Follow-up.** A large catalog feed arrives as a file. Land it in S3, process with a capped worker, and load the database in batches. Do not parse a giant file on the checkout task.

**Weak answer to avoid.** "We base64 the image into Postgres and also into Redis."
