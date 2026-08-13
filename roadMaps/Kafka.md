# Kafka Roadmap — Calibrated for a Java/Spring Backend Engineer

**Assumption:** ~6–8 hrs/week study time. Adjust the timeline linearly if your actual bandwidth differs (e.g., 3–4 hrs/week ≈ double the weeks).

---

## 0. What you can skip or skim — and why

| Topic | Skip/Skim reason |
|---|---|
| "What is a distributed system" basics | You already reason about this daily via microservices, OpenShift, and banking-grade uptime/consistency needs. |
| REST vs async messaging motivation | You've felt the pain of tight coupling and synchronous chains in a microservices/ERM context — you don't need convincing on why async matters. |
| JSON/Avro serialization concepts | You already know serialization from Spring/Jackson; Kafka just swaps the transport. |
| Basic Java concurrency vocab (threads, thread pools) | Your multithreading experience carries over directly to consumer thread models. |
| CI/CD basics, containerization basics | You already run Jenkins/Harness/OpenShift — Kafka's deployment story reuses this, not replaces it. |
| Testing philosophy (unit vs integration, TDD) | Fully transferable; only the tooling (embedded Kafka, Testcontainers) is new. |

You are **not** starting from zero on distributed systems thinking — you're starting from zero on Kafka's specific abstractions (topic/partition/offset/consumer group) and its operational model. That's a much smaller gap than a typical "beginner" roadmap assumes.

---

## Stage 1 — Beginner: Core Abstractions & Local Mechanics
**Timeline: Week 1–2**

### What to learn
- Topics, partitions, offsets, brokers, replication factor
- Producer/consumer basics: keys, partitioning strategy, delivery semantics (at-most-once/at-least-once/exactly-once at a conceptual level)
- Consumer groups and partition assignment
- Kafka CLI tools (`kafka-topics`, `kafka-console-producer/consumer`)

### Why it matters
Every advanced topic (ordering guarantees, scaling, rebalancing) is just consequences of these primitives. Get this mental model solid before touching Spring Kafka, or you'll cargo-cult config later.

### Java/Spring shortcut
Think of a **partition** like a single-threaded, append-only log — similar to how you'd reason about a JMS queue but with the added twist that a topic is a *set* of independent ordered logs, not one global queue. Your existing intuition for "one consumer thread per unit of parallelism" maps directly onto "one consumer per partition."

### Hands-on exercise
Spin up Kafka locally (KRaft mode, no ZooKeeper needed anymore), create a 3-partition topic, and use the CLI producer/consumer to observe how keyed messages land on the same partition every time, and how two consumers in the same group split partitions between them.

### Resources
- [Kafka: The Definitive Guide (2nd ed.), Ch. 1–4](https://www.confluent.io/resources/kafka-the-definitive-guide-v2/) — free from Confluent
- [Apache Kafka Quickstart docs](https://kafka.apache.org/documentation/#quickstart)
- [Confluent Developer — Kafka fundamentals course](https://developer.confluent.io/courses/apache-kafka/events/)

---

## Stage 2 — Intermediate: Spring Boot Integration
**Timeline: Week 3–4**

### What to learn
- `spring-kafka`: `KafkaTemplate`, `@KafkaListener`, `ConsumerFactory`/`ProducerFactory` config
- Serializers/deserializers (String, JSON, Avro + Schema Registry)
- Error handling: `DefaultErrorHandler`, retry topics, dead-letter topics (DLT)
- Manual vs auto offset commits, ack modes (`AckMode.MANUAL`, `RECORD`, `BATCH`)
- Idempotent producers (`enable.idempotence=true`)

### Why it matters
This is where your existing Spring Data JPA/RBAC/REST muscle memory becomes directly applicable — Spring Kafka follows the same "annotate + configure a factory bean" pattern you already know from `@Transactional` and `@RestController`.

### Java/Spring shortcut
`@KafkaListener` feels like `@RestController` for events — same annotation-driven dispatch model. Error handling with retry topics + DLT is conceptually the same pattern as a dead-letter queue in JMS, or how you already handle failed batch jobs — isolate the poison message, don't block the pipeline.

### Hands-on exercise
Build a small Spring Boot producer/consumer pair (e.g., simulate a "risk event ingestion" service resembling your ERM domain): producer publishes JSON risk events keyed by `entityId`; consumer processes them idempotently and pushes malformed events to a DLT. Add JUnit tests using embedded Kafka.

### Resources
- [Spring Kafka reference docs](https://docs.spring.io/spring-kafka/reference/html/)
- [Confluent Developer — Spring Boot + Kafka tutorials](https://developer.confluent.io/tutorials/)
- [Baeldung — Spring Kafka guides](https://www.baeldung.com/spring-kafka)

---

## Stage 3 — Advanced: Guarantees, Scaling, and Stream Processing
**Timeline: Week 5–7**

### What to learn
- Exactly-once semantics (EOS): transactional producer + `read_committed` consumer, `KafkaTransactionManager`
- Rebalancing behavior and cooperative sticky assignor; static membership to avoid rebalance storms
- Partition count/throughput tradeoffs; consumer lag monitoring
- Schema Registry + Avro/Protobuf for contract evolution (compatibility modes: backward/forward/full)
- Kafka Streams or ksqlDB basics (stateful processing, `KTable` vs `KStream`, windowing) — at least conceptually, even if you don't productionize it yet

### Why it matters
Banking/ERM systems care deeply about exactly-once correctness and auditability — this stage is where Kafka stops being "a queue" and becomes a system you can reason about for financial-grade guarantees.

### Java/Spring shortcut
EOS transactions map almost 1:1 to your existing mental model of DB transactions and `@Transactional` boundaries — except now the "transaction" can span a DB write *and* a Kafka publish via the outbox pattern, which you'll want to learn explicitly (it solves the classic "dual write" problem you've likely hit with REST + DB before).

### Hands-on exercise
Implement the **transactional outbox pattern**: a Spring Boot service writes to an `outbox` table in the same DB transaction as a business write (use your Oracle/PL-SQL background here), then a separate poller/CDC-style process publishes to Kafka exactly-once. This is a real pattern used in banking-grade systems.

### Resources
- [Confluent — Exactly Once Semantics blog series](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/)
- [Kafka Streams docs](https://kafka.apache.org/documentation/streams/)
- [microservices.io — Transactional Outbox pattern](https://microservices.io/patterns/data/transactional-outbox.html)

---

## Stage 4 — Production-Ready: Ops, Reliability, and Observability
**Timeline: Week 8–10**

### What to learn
- Broker sizing, replication factor vs `min.insync.replicas`, unclean leader election tradeoffs
- Monitoring: consumer lag, under-replicated partitions, ISR shrink/expand — via JMX metrics/Prometheus + Grafana
- Security: SASL/SCRAM or mTLS, ACLs per topic (this will feel familiar given your RBAC experience)
- Capacity planning: partition count vs consumer parallelism, retention policies (time vs size-based, compacted topics)
- Deploying Kafka clients on OpenShift; readiness/liveness probes for consumer pods; graceful shutdown (`consumer.close()` semantics on pod eviction)
- Testcontainers-based integration tests replacing/complementing embedded Kafka

### Why it matters
This is the gap between "I can write a producer/consumer" and "I can be on-call for this system" — the part most tutorials skip entirely.

### Java/Spring shortcut
Kafka ACLs are conceptually identical to the RBAC model you already implement at the API layer — principals, resources, and permitted operations. Your OpenShift/Jenkins/Harness experience means you already know *how* to deploy and gate a service; you're just learning Kafka-specific health signals (lag, ISR) to plug into that existing pipeline.

### Hands-on exercise
Take the outbox-pattern service from Stage 3 and productionize it: add Prometheus metrics for consumer lag, deploy to a local OpenShift/Minikube cluster with proper readiness probes, write a Testcontainers-based integration test suite, and simulate a broker failure to confirm your consumer group rebalances and resumes correctly without data loss.

### Resources
- [Confluent — Kafka Operations best practices](https://docs.confluent.io/platform/current/kafka/deployment.html)
- [Testcontainers — Kafka module docs](https://java.testcontainers.org/modules/kafka/)
- [Strimzi (Kafka on Kubernetes/OpenShift) docs](https://strimzi.io/documentation/)

---

## Summary Timeline (≈10 weeks @ 6–8 hrs/week)

| Week | Stage | Focus |
|---|---|---|
| 1–2 | Beginner | Core abstractions, CLI, local cluster |
| 3–4 | Intermediate | Spring Kafka, error handling, DLT |
| 5–7 | Advanced | EOS, outbox pattern, schema evolution, streams basics |
| 8–10 | Production | Ops, security/ACLs, monitoring, OpenShift deployment |

Given your background, most of the "extra" time beyond a generic roadmap should go into **Stage 3–4**, not Stage 1 — that's where the actual new engineering judgment lives for someone at your level.