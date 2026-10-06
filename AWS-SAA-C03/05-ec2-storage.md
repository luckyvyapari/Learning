# 05 — EC2 Instance Storage (EBS, AMI, Instance Store, EFS)

## EBS (Elastic Block Store)

"Network USB stick." Persists data after instance termination.

| Fact | Detail |
|---|---|
| Attach | **One instance at a time** (exception: io1/io2 Multi-Attach) |
| Scope | **Locked to one AZ** — to move: snapshot → restore in other AZ |
| Capacity | Provisioned (GB + IOPS), billed on provisioned amount, can grow over time |
| Network drive | Slight latency; detach/attach quickly |

### Delete on Termination
- **Root volume**: deleted by default
- **Other volumes**: kept by default
- Changeable (console/CLI) — e.g. keep root after terminate

### Snapshots
- Point-in-time backup; detaching recommended but not required
- Copy across **AZ or Region**
- **Snapshot Archive**: ~75% cheaper, restore takes **24–72 h**
- **Recycle Bin**: retain deleted snapshots, retention **1 day – 1 year**
- **Fast Snapshot Restore (FSR)**: fully initialize snapshot, no first-use latency (costly)

## AMI (Amazon Machine Image)

Customized instance image (OS + software + config + monitoring). Faster boot since everything is pre-packaged.
- **Region-specific** (can copy across regions)
- Sources: public (AWS), your own, AWS Marketplace
- Process: launch + customize → stop (data integrity) → create AMI (creates EBS snapshots) → launch from AMI anywhere

## EC2 Instance Store

Physical high-performance disk attached to host.
- Best I/O, very high IOPS
- **Ephemeral**: data lost on stop/hardware failure
- Use for buffer/cache/scratch/temp data
- Backup/replication is **your responsibility**

## EBS Volume Types

Characterized by **size · throughput · IOPS**. **Only gp2/gp3 and io1/io2 can be boot volumes.**

| Type | Class | Size | IOPS | Use |
|---|---|---|---|---|
| **gp3** | General SSD | 1 GiB–64 TiB | Baseline 3,000, up to 80,000 (independent of size); 125 MiB/s up to 2,000 MiB/s | Boot, dev/test, mid DBs |
| **gp2** | General SSD | 1 GiB–16 TiB | 3 IOPS/GiB, max 16,000; small volumes burst to 3,000 | Same; IOPS tied to size |
| **io1** | Provisioned IOPS SSD | 4 GiB–16 TiB | Max 64,000 (Nitro) / 32,000; 50:1 IOPS:GiB | Critical DBs |
| **io2 Block Express** | Provisioned IOPS SSD | 4 GiB–64 TiB | Max 256,000; 1,000:1 ratio; sub-ms latency | Highest performance, **supports Multi-Attach** |
| **st1** | Throughput HDD | 125 GiB–16 TiB | Max 500 IOPS / 500 MiB/s | Big data, data warehouse, logs |
| **sc1** | Cold HDD | 125 GiB–16 TiB | Max 250 IOPS / 250 MiB/s | Infrequent access, lowest cost |

- Need **>16,000 IOPS** / sustained IOPS / critical DB → **io1/io2**
- Cheap throughput for sequential big data → **st1**; archive-ish → **sc1** (HDDs can't boot)

### Multi-Attach (io1/io2)
- Same volume to **up to 16 EC2 instances**, **same AZ**
- Each instance gets full read/write
- Needs a **cluster-aware file system** (not XFS/EXT4); app must manage concurrent writes
- Use: clustered Linux apps (e.g. Teradata), higher availability

### EBS Encryption
- Data at rest, in flight (instance ↔ volume), snapshots, and volumes made from snapshots are all encrypted
- Transparent, minimal latency impact, uses **KMS (AES-256)**
- Snapshots of encrypted volumes are encrypted
- **Encrypt an existing unencrypted volume**: snapshot → copy snapshot with encryption → create volume from it → attach

## EFS (Elastic File System)

Managed **NFS** shared by many EC2 instances across **multi-AZ**.

| Fact | Detail |
|---|---|
| Protocol | NFSv4.1, POSIX file system |
| OS | **Linux only** (not Windows) |
| Access control | Security group |
| Encryption | At rest with KMS |
| Scaling | Automatic, pay per use, petabyte-scale, no capacity planning |
| Cost | ~3x gp2 |
| Use cases | Content mgmt, web serving, data sharing, WordPress |

**Performance mode** (set at creation):
- **General Purpose** (default) — latency-sensitive (web, CMS)
- **Max I/O** — higher latency, high throughput, highly parallel (big data, media)

**Throughput mode**:
- **Bursting** — 1 TB = 50 MiB/s + burst to 100 MiB/s
- **Provisioned** — set throughput regardless of storage size
- **Elastic** — auto scales (up to 3 GiB/s read, 1 GiB/s write); unpredictable workloads

**Storage classes** (lifecycle policy moves files after N days):
- Standard (frequent) · **IA** (cheaper store, retrieval fee) · **Archive** (rare, ~50% cheaper)
- Availability: **Standard = multi-AZ** (prod) · **One Zone** (dev, backups on by default, supports One Zone-IA) — >90% savings

## EBS vs EFS vs Instance Store

| | EBS | EFS | Instance Store |
|---|---|---|---|
| Attached to | 1 instance (or Multi-Attach) | 100s of instances | 1 instance |
| AZ | Locked to 1 AZ | Multi-AZ | Host-bound |
| OS | Any | Linux only | Any |
| Persistence | Yes | Yes | **Ephemeral** |
| Cost | Lower | Higher | Included |
| Migrate | Snapshot → restore | Already shared | N/A |

- gp2 IOPS grow with size; gp3/io1 IOPS independent of size
- Don't run EBS snapshots under heavy traffic (uses I/O)
