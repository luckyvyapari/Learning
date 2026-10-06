# 14 — AWS Storage Extras

## Snow Family — offline data migration & edge computing

| Device | vCPU | RAM | Storage |
|---|---|---|---|
| **Snowball Edge Storage Optimized** | 104 | 416 GB | **210 TB** |
| **Snowball Edge Compute Optimized** | 104 | 416 GB | 28 TB |

Rule of thumb: **> 1 week to transfer over the network → use Snowball.**

| Data | 100 Mbps | 1 Gbps | 10 Gbps |
|---|---|---|---|
| 10 TB | 12 days | 30 h | 3 h |
| 100 TB | 124 days | 12 days | 30 h |
| 1 PB | 3 years | 124 days | 12 days |

Challenges solved: limited connectivity/bandwidth, high network cost, shared bandwidth, unstable connections.

Flow: order device → load data → ship to AWS → imported into **S3**.

**Edge computing**: process data where created (truck, ship, mine) with poor internet. Run **EC2 instances or Lambda** on device. Uses: preprocessing, ML, media transcoding.

**Snowball → Glacier**: cannot import directly. Import to **S3**, then **lifecycle policy** → Glacier.

Others in family: Snowcone, Snowmobile (exabyte scale, truck).

## Amazon FSx — managed 3rd-party file systems

| FSx for… | Protocol | Highlights |
|---|---|---|
| **Windows File Server** | **SMB**, NTFS | **Active Directory** integration, ACLs, quotas, DFS Namespaces; mountable on Linux; SSD or HDD; on-prem via VPN/Direct Connect; **Multi-AZ**; daily backup to S3 |
| **Lustre** | Parallel distributed (Linux + cluster) | **ML, HPC**, video processing, financial modeling, EDA; 100s GB/s, millions IOPS, sub-ms; **integrates with S3** (read S3 as FS, write results back) |
| **NetApp ONTAP** | **NFS, SMB, iSCSI** | Move NAS/ONTAP workloads; broad OS compatibility (Linux, Windows, macOS, VMware Cloud, WorkSpaces, AppStream, EC2/ECS/EKS); auto-grow/shrink, snapshots, replication, compression, dedup, instant cloning |
| **OpenZFS** | **NFS** (v3–4.2) | Move ZFS workloads; up to **1M IOPS, <0.5 ms**; snapshots, compression, instant cloning |

**Lustre deployment options**
- **Scratch**: temporary, **not replicated**, high burst (6x, 200 MBps/TiB); short-term processing, cost optimized
- **Persistent**: long-term, replicated within the **same AZ**, replaces failed files in minutes; sensitive data

## Hybrid Storage: AWS Storage Gateway

S3 is proprietary (not NFS/SMB) → Storage Gateway bridges **on-prem ↔ cloud** (DR, backup/restore, tiered storage, local cache for low latency). Deployed as VM (VMware, Hyper-V, KVM) or hardware.

| Gateway | Protocol | Backed by | Notes |
|---|---|---|---|
| **S3 File Gateway** | **NFS / SMB** | S3 (Standard, IA, One Zone-IA, Intelligent-Tiering; Glacier via lifecycle) | Recently used data cached; IAM role per gateway; SMB integrates with **AD** |
| **Volume Gateway** | **iSCSI** block | S3 + **EBS snapshots** | **Cached volumes**: low-latency recent data. **Stored volumes**: entire dataset on-prem, scheduled backup to S3 |
| **Tape Gateway** | iSCSI **VTL** | S3 + Glacier | Keep existing tape backup processes; works with major backup software |

## AWS Transfer Family

Managed **FTP / FTPS / SFTP** into/out of **S3 or EFS**. Multi-AZ, scalable. Pay per **provisioned endpoint per hour + GB transferred**. Users stored in service or integrate with **AD, LDAP, Okta, Cognito, custom**. Use: sharing files, public datasets, CRM, ERP.

## AWS DataSync

Move large data **on-prem/other cloud → AWS** (NFS, SMB, HDFS, S3 API — **needs an agent**) or **AWS → AWS** (**no agent**).
- Targets: **S3 (any class incl. Glacier), EFS, FSx** (all types)
- Scheduled hourly/daily/weekly
- **Preserves permissions + metadata** (POSIX, SMB)
- One agent task up to **10 Gbps**, bandwidth throttling possible

## Storage Comparison

| Service | Type |
|---|---|
| S3 | Object |
| Glacier | Object archival |
| EBS | Network block, **1 EC2 at a time** |
| Instance Store | Physical, high IOPS, ephemeral |
| EFS | NFS for **Linux**, POSIX |
| FSx Windows | SMB/Windows |
| FSx Lustre | HPC Linux |
| FSx ONTAP | High OS compatibility |
| FSx OpenZFS | Managed ZFS |
| Storage Gateway | Hybrid access to S3/FSx/EBS/tape |
| Transfer Family | FTP/FTPS/SFTP on S3/EFS |
| DataSync | Scheduled sync on-prem→AWS or AWS→AWS |
| Snow family | Physical migration |

## Exam Hints

- Windows shared drive / AD → **FSx for Windows**
- HPC / ML with S3 → **FSx for Lustre**
- On-prem NFS/SMB to S3 with local cache → **S3 File Gateway**
- Replace tape → **Tape Gateway**
- FTP/SFTP into S3 → **Transfer Family**
- Network too slow for petabytes → **Snowball**
- Scheduled bulk sync → **DataSync**
