# 26 — Disaster Recovery & Migrations

## Disaster Recovery

Disaster = any event hurting business continuity/finances. DR types: on-prem → on-prem (traditional, very expensive), **on-prem → AWS (hybrid)**, **AWS Region A → Region B**.

| Term | Meaning |
|---|---|
| **RPO** (Recovery Point Objective) | How far back data is lost = **data loss** |
| **RTO** (Recovery Time Objective) | How long until recovered = **downtime** |

### Strategies (cheapest/slowest → priciest/fastest; faster RTO as you go down)

| Strategy | What runs | RPO / RTO | Notes |
|---|---|---|---|
| **Backup & Restore** | Nothing running; backups in S3/Glacier (Snowball / Storage Gateway from on-prem); AMIs, EBS/RDS/Redshift snapshots; lifecycle | **High RPO**, slow RTO | Cheapest |
| **Pilot Light** | **Small critical core** always running (e.g. RDS with replication running, EC2 not running) | Faster than backup/restore | Route 53 failover; app tier started on disaster |
| **Warm Standby** | **Full system up at minimum size** (ELB + ASG at min + replicated DB) | Fast | Scale to production load on disaster |
| **Hot Site / Multi-Site** | **Full production scale** running in AWS and on-prem (active-active) | **Very low RTO (minutes/seconds)** | Very expensive. All-AWS multi-region: Route 53 + two regions + **Aurora Global** |

### DR Tips
- **Backup**: EBS snapshots, RDS automated backups/snapshots, S3/IA/Glacier + lifecycle + CRR, Snowball/Storage Gateway from on-prem
- **High availability**: **Route 53** to migrate DNS region→region, **RDS/ElastiCache Multi-AZ**, EFS, S3, **S2S VPN as recovery path for Direct Connect**
- **Replication**: RDS cross-region replication, **Aurora + Global Databases**, on-prem → RDS replication, Storage Gateway
- **Automation**: **CloudFormation / Elastic Beanstalk** to re-create environment, **CloudWatch** recover/reboot, **Lambda** for custom automation
- **Chaos**: Netflix **Simian Army** randomly terminates EC2

### AWS Elastic Disaster Recovery (DRS) — formerly CloudEndure DR
Recover **physical, virtual, cloud** servers into AWS (Oracle, MySQL, SQL Server, **SAP**, ransomware protection). **Continuous block-level replication** (seconds) via replication agent to **low-cost staging EC2/EBS**; **failover** (minutes) to target EC2; **failback** supported.

## Database Migration

### DMS (Database Migration Service)
Quick, secure, **resilient, self-healing** migration; **source stays available** during migration. Runs on an **EC2 replication instance you create**.
- **Homogeneous** (Oracle → Oracle) and **heterogeneous** (SQL Server → Aurora)
- **Continuous replication with CDC** (full load + CDC)
- **Sources**: on-prem/EC2 DBs (Oracle, SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2), Azure SQL, **RDS (all, incl. Aurora)**, S3, DocumentDB
- **Targets**: on-prem/EC2 DBs, RDS, **Redshift, DynamoDB, S3**, OpenSearch, Kinesis Data Streams, Kafka, DocumentDB, Neptune, Redis, Babelfish
- **Multi-AZ**: synchronous standby replica in another AZ — redundancy, no I/O freezes, fewer latency spikes

### SCT (Schema Conversion Tool)
Convert **schema** between engines: OLTP (SQL Server/Oracle → MySQL/PostgreSQL/Aurora), OLAP (Teradata/Oracle → **Redshift**). Prefer compute-intensive instances. **Not needed for same engine** (on-prem PostgreSQL → RDS PostgreSQL).

### RDS/Aurora migrations
| From → To | Options |
|---|---|
| **RDS MySQL → Aurora MySQL** | (1) Snapshot restore as Aurora; (2) create **Aurora Read Replica**, promote when lag = 0 (time/cost) |
| **External MySQL → Aurora MySQL** | (1) **Percona XtraBackup** → S3 → create Aurora from S3; (2) `mysqldump` (slower); **DMS** if both running |
| **RDS PostgreSQL → Aurora PostgreSQL** | Snapshot restore, or Aurora Read Replica + promote |
| **External PostgreSQL → Aurora PostgreSQL** | Backup to S3 → import with **`aws_s3` extension**; **DMS** if both running |

## Server / Application Migration

- **Amazon Linux 2 AMI** downloadable as VM (VMware, KVM, VirtualBox, Hyper-V)
- **VM Import/Export**: migrate VMs into EC2, DR repository, export back
- **Application Discovery Service**: gather on-prem server data (utilization, dependency mapping), tracked in **Migration Hub**
  - **Agentless** (Discovery Connector): VM inventory, config, CPU/memory/disk history
  - **Agent-based**: system config/performance, running processes, network connections
- **Application Migration Service (MGN)**: "evolution of CloudEndure Migration" (replaces SMS) — **lift-and-shift (rehost)**, converts physical/virtual/cloud servers to run natively on AWS; **continuous replication → cutover**; minimal downtime
- **VMware Cloud on AWS**: extend/migrate vSphere workloads to AWS, hybrid, DR
- **DMS**: on-prem↔AWS, AWS↔AWS, DBs, DynamoDB…

### Large data transfer example (200 TB, 100 Mbps link)
| Method | Time |
|---|---|
| Internet / S2S VPN | ~**185 days** |
| Direct Connect 1 Gbps | ~**18.5 days** (but >1 month setup) |
| **Snowball** | ~**1 week** end-to-end (can combine with DMS) |
Ongoing replication: **S2S VPN or DX + DMS or DataSync**.

## AWS Backup

Fully managed **central, automated backups** across services — no custom scripts. Supports **EC2/EBS, S3, RDS (all engines)/Aurora/DynamoDB, DocumentDB, Neptune, EFS, FSx (Lustre, Windows), Storage Gateway (Volume)**. **Cross-region + cross-account**, **PITR**, on-demand + scheduled, **tag-based policies**.

**Backup Plan**: frequency (12 h, daily, weekly, monthly, cron), backup window, **transition to cold storage**, **retention period**.

**Backup Vault Lock**: **WORM** for backups — protects from inadvertent/malicious deletes and retention changes; **even root can't delete**.

## Exam Hints

- Cheapest DR, can tolerate hours → **Backup & Restore**; core DB replicated, rest off → **Pilot Light**; scale up on demand → **Warm Standby**; seconds RTO → **Multi-Site**
- Heterogeneous DB migration → **SCT + DMS**; same engine → **DMS only**
- Lift-and-shift servers → **MGN**; discover dependencies → **Application Discovery Service**
- Centralized backups across services/accounts → **AWS Backup**; immutable → **Vault Lock**
- Tight network, huge data → **Snowball**
