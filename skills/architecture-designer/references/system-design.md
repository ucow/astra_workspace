# System Design Template

## 1. Problem Statement
- What are we building?
- Why are we building it?
- Who are the users?

## 2. Requirements

### Functional Requirements
- FR-1: ...
- FR-2: ...

### Non-Functional Requirements
- Performance: ...
- Scalability: ...
- Availability: ...
- Security: ...
- Latency: ...

### Constraints
- Budget: ...
- Timeline: ...
- Team: ...
- Technology: ...

## 3. Capacity Estimation
- Users: Daily active / Monthly active
- Requests: QPS (peak / average)
- Storage: Data growth rate
- Bandwidth: Data transfer estimates

## 4. High-Level Architecture

### Architecture Diagram
```
[Client] → [Load Balancer] → [API Gateway] → [Services]
                                                      ↓
                                              [Cache / Queue]
                                                      ↓
                                                [Database]
```

### Component Description
| Component | Responsibility | Technology |
|-----------|---------------|------------|
| ... | ... | ... |

## 5. Data Model

### Entities
- Entity 1: {fields}
- Entity 2: {fields}

### Relationships
- Entity 1 → Entity 2: {relationship type}

### Storage Design
- Primary DB: ...
- Cache: ...
- Search: ...

## 6. API Design

### Endpoints
| Method | Path | Description |
|--------|------|-------------|
| GET | /api/v1/... | ... |
| POST | /api/v1/... | ... |

## 7. Reliability & Failure Modes

### Failure Scenarios
| Failure | Impact | Mitigation |
|---------|--------|------------|
| ... | ... | ... |

### SLA Targets
- Availability: 99.9%
- Latency p99: < 200ms
- RPO: < 1 hour
- RTO: < 15 minutes

## 8. Trade-offs & Decisions
- Reference ADRs for key decisions
- Document what we're optimizing for

## 9. Evolution Plan
- Phase 1: MVP
- Phase 2: Scale
- Phase 3: Optimize
