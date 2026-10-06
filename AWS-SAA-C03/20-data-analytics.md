# 20 — Data & Analytics

## Athena

Serverless **SQL on S3** (built on **Presto**). CSV, JSON, ORC, Avro, Parquet. **$5 per TB scanned.** Pairs with **QuickSight**. Use: BI/reporting, query **VPC Flow Logs, ELB logs, CloudTrail**.

**Exam tip: analyze S3 data with serverless SQL → Athena.**

Performance/cost:
- **Columnar formats (Parquet / ORC)** → less scanning (use **Glue** to convert)
- **Compress** (gzip, snappy, zstd…)
- **Partition** (`year=1991/month=1/day=1/`)
- Use **larger files (> 128 MB)**

**Federated Query**: SQL across relational, non-relational, object, custom sources (ElastiCache, DocumentDB, DynamoDB, RDS, Redshift, CloudWatch Logs, on-prem HBase…) using **Lambda data source connectors**; results stored in S3.

## Redshift

Based on PostgreSQL but **OLAP** (analytics / data warehouse), **not OLTP**. 10x better performance, PB scale, **columnar storage + parallel query engine**, provisioned or serverless, SQL, BI tools (QuickSight, Tableau). vs Athena: faster joins/aggregations thanks to indexes.

- **Cluster**: **Leader node** (planning, aggregation) + **Compute nodes** (run queries). Provisioned mode: pick instance types, can reserve
- **Snapshots & DR**: Multi-AZ for some clusters; incremental snapshots stored in S3; restore into **new cluster**; automated every **8 h / 5 GB / schedule**, retention 1–35 days; manual kept until deleted; **auto-copy snapshots to another region**
- **Loading data**: Kinesis Data Firehose (via S3 COPY), **S3 `COPY` command**, JDBC from EC2. **Large batch inserts are much better**. **Enhanced VPC Routing** keeps traffic inside VPC
- **Redshift Spectrum**: query data **in S3 without loading it**; requires a Redshift cluster; fans out to thousands of Spectrum nodes

## OpenSearch (successor to Elasticsearch)

Search **any field, partial matches** (DynamoDB only queries by key/index). Usually a **complement** to another DB. Managed cluster or serverless. SQL via plugin. Ingest from **Firehose, IoT, CloudWatch Logs**. Security: Cognito, IAM, KMS, TLS. **OpenSearch Dashboards**.

Patterns:
- **DynamoDB → Stream → Lambda → OpenSearch** (search API + item retrieve API)
- **CloudWatch Logs → Subscription Filter → Lambda** (real time) or **→ Firehose** (near real time) → OpenSearch
- **Kinesis Data Streams / Firehose** (with Lambda transformation) → OpenSearch

## EMR (Elastic MapReduce)

Managed **Hadoop** clusters (100s of EC2) bundled with **Spark, HBase, Presto, Flink**. Auto-scaling, Spot integration. Use: data processing, ML, web indexing, big data.

| Node | Role |
|---|---|
| **Master** | Manage cluster, coordinate, health — long running |
| **Core** | Run tasks **and store data** — long running |
| **Task** (optional) | Run tasks only — usually **Spot** |

Purchasing: On-Demand (reliable), **Reserved** (min 1 yr; used automatically), Spot (cheap, can terminate). Cluster can be **long-running or transient**.

## QuickSight

Serverless **ML-powered BI** dashboards; fast, scalable, embeddable, **per-session pricing**. Sources: RDS, Aurora, Athena, Redshift, S3, OpenSearch, Timestream, on-prem JDBC, SaaS, files. **SPICE** in-memory engine when importing data. Enterprise: **column-level security (CLS)**.

- Users (standard) and **Groups (enterprise)** live **only in QuickSight, not IAM**
- **Dashboard** = read-only snapshot of an analysis (keeps filters/params/controls/sort); must **publish** to share; viewers can see underlying data

## AWS Glue

Serverless managed **ETL** (extract, transform, load).
- Convert CSV → **Parquet** (S3 PUT event → Lambda/EventBridge → Glue ETL job → output bucket → Athena)
- **Glue Data Catalog**: metadata catalog of datasets; **Glue Data Crawler** discovers/writes metadata; used by Athena, Redshift Spectrum, EMR
- Glue **Job Bookmarks** (avoid reprocessing), **DataBrew** (clean/normalize with pre-built transforms), **Studio** (GUI), **Streaming ETL** (Spark Structured Streaming; Kinesis, Kafka, MSK)

## Lake Formation

Fully managed **data lake** (central place for all analytics data) in days, built on **Glue**. Discover, cleanse, transform, ingest; dedupe via ML transforms; structured + unstructured; blueprints for S3, RDS, relational/NoSQL. **Fine-grained access control: row and column level**, centralized permissions across Athena, Redshift, QuickSight, EMR.

## Managed Service for Apache Flink (formerly Kinesis Data Analytics for Flink)

Run **Flink** (Java/Scala/SQL) stream processing on managed cluster; reads from **Kinesis Data Streams** and **MSK**; auto scaling, checkpoints/snapshots. **Flink does NOT read from Firehose.**

## Amazon MSK (Managed Streaming for Apache Kafka)

Alternative to Kinesis; managed **Kafka** (brokers + Zookeeper) in your VPC, **multi-AZ (up to 3)**, auto recovery, data on **EBS** as long as you want. **MSK Serverless** auto provisions/scales.

| Kinesis Data Streams | MSK |
|---|---|
| 1 MB message limit | 1 MB default, configurable (e.g. 10 MB) |
| **Shards** (split/merge) | **Partitions** (can only add) |
| TLS in-flight | PLAINTEXT or TLS |
| KMS at rest | KMS at rest |

MSK consumers: Flink, Glue Streaming ETL, Lambda, apps on EC2/ECS/EKS.

## Big Data Ingestion Pipeline (fully serverless)

IoT devices → **IoT Core** → **Kinesis Data Streams** (real time) → **Firehose** (near real time, ~1 min; **Lambda** transforms) → **S3 ingestion bucket** → **S3 event → SQS → Lambda** (every 1 min) → **Athena** (SQL) → **S3 reporting bucket** → **QuickSight** / **Redshift**.

## Exam Hints

- Serverless SQL on S3 → **Athena**; data warehouse with joins → **Redshift**
- Query S3 from Redshift without loading → **Spectrum**
- Full-text/partial search → **OpenSearch**
- Hadoop/Spark big data → **EMR**; managed ETL → **Glue**; data lake with row/column security → **Lake Formation**
- Dashboards → **QuickSight**
- Kafka → **MSK**; Flink → **Managed Flink**
- Cheaper Athena → Parquet + partition + compress
