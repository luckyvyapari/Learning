# 07 — RDS, Aurora & ElastiCache

## RDS (Relational Database Service)

Managed SQL DB: **Postgres, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, Aurora**.

Managed benefits: auto provisioning + OS patching, continuous backups + **point-in-time restore**, monitoring, read replicas, Multi-AZ, maintenance windows, vertical + horizontal scaling, EBS-backed storage. **No SSH** (except RDS Custom).

### Storage Auto Scaling
- Set **Maximum Storage Threshold**
- Triggers if: free storage < **10%** AND low-storage ≥ **5 min** AND **6 h** since last modification
- Good for unpredictable workloads; all engines

### Read Replicas vs Multi-AZ

| | Read Replicas | Multi-AZ |
|---|---|---|
| Purpose | **Scale reads** | **Disaster recovery / HA** |
| Replication | **ASYNC** (eventually consistent) | **SYNC** |
| Count | Up to **15** | 1 standby |
| Location | Same AZ, cross-AZ, cross-region | Different AZ |
| Access | Apps must use replica endpoint; SELECT only | One DNS name, **automatic failover**; standby not readable |
| Promotion | Can be promoted to own DB | Standby takes over |
| Network cost | Free within same region; **$$ cross-region** | — |

- Replicas can themselves be set up as Multi-AZ for DR
- Use case for replica: reporting/analytics without hitting production
- **Single-AZ → Multi-AZ**: zero downtime ("modify"); snapshot → restore in new AZ → sync

### RDS Custom
Oracle & SQL Server with **OS + DB customization** (SSH / SSM Session Manager, patches, native features). Deactivate Automation Mode to customize; snapshot first.

### Backups

| | RDS | Aurora |
|---|---|---|
| Automated | Daily full + transaction logs every **5 min** → restore to any point; **1–35 days** (0 = disabled) | **1–35 days, cannot be disabled** |
| Manual snapshots | Keep as long as you want | Same |

- Stopped RDS DB still **pays for storage** → for long stops, snapshot + restore
- Restore always creates a **new DB**
- MySQL RDS: restore from backup in **S3**; Aurora MySQL: from **Percona XtraBackup** in S3

### Security (RDS & Aurora)
- At-rest encryption with **KMS**, set at **launch time**; unencrypted master → replicas can't be encrypted. To encrypt existing DB: snapshot → restore as encrypted
- In-flight: TLS by default
- **IAM authentication** (instead of user/password)
- Security groups for network access
- Audit logs → CloudWatch Logs

### RDS Proxy
- Managed, serverless, autoscaling, multi-AZ **connection pooling**
- Reduces DB CPU/RAM load and open connections; **cuts failover time up to 66%**
- Supports RDS (MySQL, PostgreSQL, MariaDB, SQL Server) and Aurora (MySQL, PostgreSQL)
- Enforce IAM auth, credentials in **Secrets Manager**
- **Never publicly accessible** (VPC only)
- Best fit: **Lambda** functions (many short connections)

## Aurora

AWS proprietary; **MySQL & PostgreSQL compatible**. ~5x MySQL / 3x Postgres performance; ~20% more expensive than RDS.

| Feature | Detail |
|---|---|
| Storage | Auto-grows in 10 GB steps up to **256 TB** |
| Copies | **6 copies across 3 AZ**; **4/6 for writes, 3/6 for reads**; self-healing |
| Replicas | Up to **15**, sub-10 ms lag |
| Failover | Master failover < **30 s**, HA native |
| Endpoints | **Writer endpoint** (master), **Reader endpoint** (load-balanced replicas), **Custom endpoints** (subset, e.g. analytics on big instances) |
| Replica auto scaling | Based on CPU |
| Backtrack | Rewind to any point **without backups** |

### Aurora Variants

| Variant | What |
|---|---|
| **Serverless** | Auto instantiation + scaling by usage; pay per second; infrequent/unpredictable workloads |
| **Global Database** (recommended for cross-region) | 1 primary region (R/W) + up to **10 read-only regions**, lag **< 1 s**, up to 16 replicas per secondary; DR **RTO < 1 min** |
| **Cross-Region Read Replicas** | Simpler DR option |
| **Machine Learning** | ML predictions via SQL — SageMaker, Comprehend (fraud, ads, sentiment, recommendations) |
| **Babelfish** | Aurora PostgreSQL understands **T-SQL** — SQL Server apps migrate with little change (use SCT + DMS) |
| **Cloning** | Copy-on-write clone of cluster; faster than snapshot/restore; create staging from production |

## ElastiCache

Managed **Redis / Memcached** — in-memory, low latency. Reduces DB load, makes apps stateless. **Needs app code changes.**

| | Redis | Memcached |
|---|---|---|
| HA | **Multi-AZ + auto-failover**, read replicas | None |
| Persistence | **AOF** durability | Non-persistent |
| Backup/restore | Yes | Serverless only |
| Scaling | Replicas + sharding | Multi-node sharding |
| Data types | Sets, **Sorted Sets** | Simple |
| Threading | — | Multi-threaded |

### Security
- Redis: **IAM auth** (API-level only), **Redis AUTH** password/token, security groups, SSL in-flight
- Memcached: SASL-based auth

### Patterns
- **Lazy Loading** (cache-aside): cache on read miss; can go **stale**
- **Write-Through**: update cache on every DB write; **no stale data**
- **Session Store**: temporary sessions with **TTL** → stateless apps
- Redis **Sorted Sets**: real-time gaming leaderboards

## Exam Hints

- Scale reads → Read Replicas; survive AZ failure → Multi-AZ
- Lambda + RDS connection storms → **RDS Proxy**
- Cross-region low-latency/DR for Aurora → **Global Database**
- Unpredictable / infrequent DB load → **Aurora Serverless**
- SQL Server app → Aurora → **Babelfish**
- Quick prod clone without impact → **Aurora Cloning**
- Leaderboard / HA cache → **Redis**
