

For a 15-year Architect-level role, interviewers expect breadth across the stack plus depth in trade-off reasoning. Here's what to prepare, organized by priority.

## Core Java & Spring (must be airtight)
- Java internals: GC algorithms (G1, ZGC, Shenandoah), memory model, JIT, virtual threads (Project Loom), records, sealed classes, pattern matching
- Spring: IoC/DI internals, AOP, bean lifecycle, proxying (CGLIB vs JDK), Spring Boot auto-configuration internals, Spring Cloud (Config, Gateway, Sleuth/Micrometer Tracing)
- Concurrency: ExecutorService, CompletableFuture, reactive (WebFlux, Project Reactor, backpressure)
- Hibernate/JPA: N+1, lazy vs eager, second-level cache, batch fetching, optimistic vs pessimistic locking, entity lifecycle

## Architecture Patterns (this is what separates architect from senior dev)

- Microservices decomposition strategies, Strangler Fig, Bulkhead, Circuit Breaker, Saga (orchestration vs choreography — you already have this), CQRS, Event Sourcing
- Domain-Driven Design: bounded contexts, aggregates, anti-corruption layers
- API design: REST maturity, GraphQL, gRPC, idempotency, versioning strategies
- Hexagonal/Clean Architecture, Strangler pattern for legacy modernization

## Data & Messaging
- PostgreSQL/Oracle: indexing strategies, partitioning, replication, sharding, query plan analysis, isolation levels
- Redis: caching patterns (cache-aside, write-through), distributed locks, pub/sub, Redis Cluster
- Kafka: partitioning strategy, exactly-once semantics, consumer group rebalancing, schema registry (Avro/Protobuf)
- Distributed transactions: 2PC vs Saga vs eventual consistency — be ready to defend trade-offs with real scenarios

## Cloud & Infra
- Kubernetes: deployments, StatefulSets, HPA/VPA, service mesh (Istio/Linkerd), operators
- Docker: multi-stage builds, image security scanning
- AWS/Azure/GCP (pick one deeply): API Gateway, Lambda, ECS/EKS, SQS/SNS, S3, IAM
- Infrastructure as Code: Terraform, Helm
- Observability: distributed tracing (OpenTelemetry, Jaeger), Prometheus/Grafana, ELK/EFK

## Security
- OAuth2/OIDC, JWT, mTLS, Zero Trust architecture, secrets management (Vault, AWS Secrets Manager)
- API security: rate limiting, WAF concepts

## Non-Functional / Architect-Specific Skills
- System design: capacity planning, scalability (horizontal vs vertical), availability (SLA/SLO/SLI), disaster recovery, multi-region architecture
- Trade-off articulation: consistency vs availability (CAP), latency vs throughput, build vs buy
- Cost optimization, technical debt management, migration strategies (monolith → microservices)
- Soft skills: stakeholder communication, mentoring, ADRs (Architecture Decision Records), technical roadmaps

## What interviewers will actually probe
- "Design a system that does X" (e-commerce checkout, ride-sharing, notification system) — practice 4-5 of these end-to-end
- "Tell me about a time you made an architectural decision that was wrong" — they want accountability and learning, not perfection
- Deep dives into one project you led — be ready to defend every technology choice

Want me to build out a mock system design question (with your stack) so you can practice articulating trade-offs, or create a structured prep document/checklist you can save and work through?
