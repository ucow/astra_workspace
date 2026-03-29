# Non-Functional Requirements (NFR) Checklist

## Performance
- [ ] Response time targets defined (p50, p95, p99)
- [ ] Throughput requirements (QPS / TPS)
- [ ] Resource utilization limits (CPU, memory, I/O)
- [ ] Batch processing windows defined
- [ ] Page load time targets (if applicable)

## Scalability
- [ ] Expected user growth trajectory (current → 1yr → 3yr)
- [ ] Horizontal vs vertical scaling strategy
- [ ] Stateless service design where possible
- [ ] Database scaling plan (read replicas, sharding)
- [ ] CDN strategy for static assets
- [ ] Auto-scaling policies defined

## Availability & Reliability
- [ ] Availability SLA target (e.g., 99.9%)
- [ ] RPO (Recovery Point Objective) defined
- [ ] RTO (Recovery Time Objective) defined
- [ ] Single points of failure identified and mitigated
- [ ] Graceful degradation strategy
- [ ] Health check endpoints implemented
- [ ] Circuit breaker patterns in place

## Security
- [ ] Authentication mechanism defined
- [ ] Authorization model (RBAC / ABAC / custom)
- [ ] Data encryption at rest
- [ ] Data encryption in transit (TLS)
- [ ] Input validation strategy
- [ ] Rate limiting / DDoS protection
- [ ] Audit logging for sensitive operations
- [ ] Compliance requirements (GDPR, HIPAA, etc.)

## Observability
- [ ] Logging strategy (structured logs, log levels)
- [ ] Metrics collection (application + infrastructure)
- [ ] Distributed tracing for microservices
- [ ] Alerting rules defined
- [ ] Dashboard for key metrics
- [ ] Error tracking (Sentry, etc.)

## Maintainability
- [ ] Code organization / module boundaries
- [ ] API versioning strategy
- [ ] Configuration management approach
- [ ] CI/CD pipeline design
- [ ] Documentation requirements
- [ ] Technical debt management process

## Cost
- [ ] Infrastructure cost estimate
- [ ] Cost per user / per transaction
- [ ] Reserved vs on-demand resources
- [ ] Data storage cost projections
- [ ] Third-party service costs
- [ ] Cost monitoring and budgeting

## Compatibility & Integration
- [ ] API backward compatibility
- [ ] Browser/device support (if applicable)
- [ ] Integration protocol (REST / gRPC / GraphQL)
- [ ] Data format standards (JSON / Protobuf)
- [ ] External service dependencies documented

## Operational
- [ ] Deployment strategy (blue-green / canary / rolling)
- [ ] Rollback plan
- [ ] Database migration strategy
- [ ] Feature flag system
- [ ] Runbook for common incidents
- [ ] On-call rotation defined
