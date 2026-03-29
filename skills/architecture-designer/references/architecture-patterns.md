# Architecture Patterns

## Monolith vs Microservices

### Monolith
- **When to use**: Small team, early-stage product, simple domain
- **Pros**: Simple deployment, easy debugging, no distributed complexity
- **Cons**: Scaling bottlenecks, deployment coupling, tech stack lock-in

### Microservices
- **When to use**: Large team, complex domain, independent scaling needs
- **Pros**: Independent deployment, tech diversity, fault isolation
- **Cons**: Distributed complexity, network latency, data consistency challenges

### Decision Framework
| Factor | Monolith | Microservices |
|--------|----------|---------------|
| Team size | < 10 | > 10 |
| Domain complexity | Low | High |
| Scale requirements | Uniform | Per-service |
| Deployment frequency | Low | High |
| Time to market | Critical | Flexible |

## Event-Driven Architecture

### Patterns
- **Event Sourcing**: Store events as source of truth, reconstruct state
- **CQRS**: Separate read and write models for optimization
- **Saga**: Manage distributed transactions across services
- **Pub/Sub**: Decouple producers and consumers

### When to Use
- Asynchronous workflows are natural
- High throughput with eventual consistency acceptable
- Need to react to state changes across services

## Layered Architecture

```
┌──────────────────────────────────────┐
│ Presentation Layer                   │ ← Controllers, Views
├──────────────────────────────────────┤
│ Application Layer                    │ ← Use Cases, Orchestration
├──────────────────────────────────────┤
│ Domain Layer                         │ ← Business Logic, Entities
├──────────────────────────────────────┤
│ Infrastructure Layer                 │ ← DB, External Services
└──────────────────────────────────────┘
```

- Dependency rule: outer layers depend on inner, never reverse
- Domain layer has zero external dependencies

## Hexagonal Architecture (Ports & Adapters)

- **Ports**: Interfaces defining what the domain needs
- **Adapters**: Implementations (swappable, testable)
- **Core**: Business logic independent of technology

## Common Patterns

| Pattern | Use Case |
|---------|----------|
| API Gateway | Single entry point for clients |
| Circuit Breaker | Prevent cascading failures |
| Sidecar | Cross-cutting concerns (logging, proxy) |
| Strangler Fig | Incremental monolith migration |
| BFF | Client-specific API aggregation |
