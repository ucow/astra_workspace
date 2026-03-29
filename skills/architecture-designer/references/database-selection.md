# Database Selection Guide

## Decision Framework

### Questions to Ask
1. What is the data shape? (Structured / Semi-structured / Unstructured)
2. What are the access patterns? (Read-heavy / Write-heavy / Mixed)
3. What consistency model is needed? (Strong / Eventual)
4. What scale is expected? (Data volume / Query volume)
5. What queries are needed? (Simple lookups / Complex joins / Full-text search / Graph traversal)

## Database Types

### Relational (PostgreSQL, MySQL)
- **Best for**: Structured data, complex queries, ACID transactions
- **Avoid when**: Schema changes frequently, massive horizontal scaling needed
- **Patterns**: Normalized data, JOINs, foreign keys

### Document (MongoDB, Firestore)
- **Best for**: Semi-structured data, schema flexibility, rapid iteration
- **Avoid when**: Complex relationships, strong consistency required
- **Patterns**: Aggregate-oriented design, denormalization

### Key-Value (Redis, DynamoDB)
- **Best for**: Caching, session storage, high-throughput lookups
- **Avoid when**: Complex queries, relationships between data
- **Patterns**: Single-key access, TTL-based expiration

### Graph (Neo4j, Neptune)
- **Best for**: Social networks, recommendation engines, fraud detection
- **Avoid when**: Simple CRUD, no relationship traversal needed
- **Patterns**: Node-edge-property model, traversals

### Search (Elasticsearch, Meilisearch)
- **Best for**: Full-text search, log analytics, faceted navigation
- **Avoid when**: Primary data store, transactional writes
- **Patterns**: Index denormalized documents, near-real-time

### Time-Series (TimescaleDB, InfluxDB)
- **Best for**: Metrics, IoT data, monitoring
- **Avoid when**: General-purpose OLTP
- **Patterns**: Append-only, downsampling, retention policies

## Quick Decision Table

| Need | Recommendation |
|------|---------------|
| ACID transactions | PostgreSQL |
| Schema flexibility | MongoDB |
| Sub-ms reads | Redis |
| Relationship queries | Neo4j |
| Full-text search | Elasticsearch |
| Time-series data | TimescaleDB |
| Serverless scale | DynamoDB / Firestore |
| Analytics | ClickHouse / BigQuery |

## Polyglot Persistence
- Most systems need more than one database
- Use the right tool for each access pattern
- Keep data synchronized via events or CDC
