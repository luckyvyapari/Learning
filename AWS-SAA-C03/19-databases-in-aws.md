# 19 — Databases in AWS (Choosing the Right One)

## Choosing — questions to ask

Read/write/balanced? Throughput changes? Data size + growth + object size + access pattern? Durability / source of truth? Latency + concurrent users? Data model (joins, structured/semi-structured)? Schema strictness vs flexibility, reporting, search? RDBMS vs NoSQL? License cost → move to cloud-native (Aurora)?

## Database Types

| Need | Service |
|---|---|
| **RDBMS (SQL / OLTP)**, joins | **RDS, Aurora** |
| **NoSQL** (no joins/SQL) | **DynamoDB** (~JSON), **ElastiCache** (key/value), **Neptune** (graph), **DocumentDB** (MongoDB), **Keyspaces** (Cassandra) |
| **Object store** | **S3** (big objects), **Glacier** (backup/archive) |
| **Data warehouse (OLAP / BI)** | **Redshift**, **Athena**, **EMR** |
| **Search** (free text, unstructured) | **OpenSearch** |
| **Graph** | **Neptune** |
| **Ledger** | **QLDB** |
| **Time series** | **Timestream** |

## Summaries

### RDS
Managed PostgreSQL/MySQL/Oracle/SQL Server/DB2/MariaDB/Custom. Choose instance size + EBS type/size, **storage auto-scaling**, **Read Replicas + Multi-AZ**, security via IAM/SG/KMS/SSL, backups with **PITR up to 35 days** + manual snapshots, **scheduled maintenance (with downtime)**, IAM auth + Secrets Manager, **RDS Custom** (Oracle & SQL Server). Use: relational data, SQL, transactions.

### Aurora
Compatible API for PostgreSQL/MySQL, **storage separated from compute**; **6 replicas over 3 AZ**, self-healing, auto-scaling storage; cluster with **writer/reader/custom endpoints**, replica auto-scaling. **Serverless** (unpredictable load), **Global** (up to 16 read instances per region, < 1 s replication), **Machine Learning** (SageMaker, Comprehend), **Cloning**. Use: like RDS but less maintenance, more performance/features.

### ElastiCache
Managed Redis/Memcached; **sub-millisecond**; clustering + Multi-AZ + read replicas (Redis); IAM/SG/KMS/Redis AUTH; backup/PITR; **needs code changes**. Use: key/value, read-heavy, cache DB query results, session store, no SQL.

### DynamoDB
Proprietary, **serverless NoSQL**, ms latency; provisioned (+auto-scaling) or on-demand; can **replace ElastiCache** as key/value store (TTL for sessions); Multi-AZ, transactions; **DAX** (µs reads); IAM-only security; **Streams** → Lambda/Kinesis; **Global Tables** (active-active); PITR 35 days (restore to new table) + on-demand backups; **export to S3 with no RCU**, import with no WCU; evolving schemas. Use: serverless apps, small documents (100s KB), distributed serverless cache.

### S3
Key/value object store; **great for big objects, not many small ones**; serverless, infinite scale, **max 50 TB**, versioning. Tiers: Standard, IA, Intelligent, Glacier + lifecycle. Features: versioning, encryption, replication, MFA Delete, access logs. Security: IAM, bucket policies, ACL, Access Points, Object Lambda, CORS, Object/Vault Lock. Encryption: SSE-S3, SSE-KMS, SSE-C, client-side, TLS, default. **S3 Batch**, **S3 Inventory**, **Transfer Acceleration**, multi-part, **S3 Select**. Events → SNS/SQS/Lambda/EventBridge.

## Other Databases

| Service | Key points |
|---|---|
| **DocumentDB** | "Aurora for **MongoDB**" — store/query/index **JSON**; managed, 3-AZ replication, storage grows in **10 GB** steps, millions req/s |
| **Neptune** | **Graph DB**; 3 AZ, up to **15 read replicas**, billions of relations at ms latency. Use: social networks, knowledge graphs (Wikipedia), **fraud detection**, recommendation engines. **Neptune Streams**: ordered real-time change log (no duplicates, strict order) via HTTP REST API → sync to S3/OpenSearch/ElastiCache, notifications, cross-region replication |
| **Keyspaces** | Managed **Apache Cassandra**-compatible; serverless; tables replicated **3x** across AZs; **CQL**; on-demand or provisioned+auto-scaling; encryption; **PITR 35 days**. Use: IoT, time-series |
| **Timestream** | Serverless **time series**; trillions of events/day; 1000s x faster, 1/10 cost vs relational; memory tier for recent data + cost-optimized for history; built-in time-series analytics; scheduled queries. Use: IoT, operational apps, real-time analytics. Integrates with IoT, Kinesis, Lambda, Prometheus, QuickSight, SageMaker |
| **QLDB** | Immutable **ledger** DB |

## Exam Hints

- Joins + transactions → **RDS / Aurora**
- MongoDB workload → **DocumentDB**; Cassandra → **Keyspaces**
- Highly connected data / fraud / recommendations → **Neptune**
- IoT time series → **Timestream**
- Serverless key/value with µs reads → **DynamoDB + DAX**
- Free-text search over any field → **OpenSearch**
