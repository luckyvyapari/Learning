# 04 — EC2 (Associate Level)

## Public vs Private IP (IPv4)

| | Public IP | Private IP |
|---|---|---|
| Identifies on | Internet (WWW) | Private network only |
| Uniqueness | Globally unique | Unique within the network (two companies can reuse same ranges) |
| Geo-locatable | Yes | No |

Machines reach the internet via **NAT + Internet Gateway**. By default an EC2 gets a private IP (internal) and a public IP (WWW). SSH from outside must use the **public IP**. **Stop → start can change the public IP.**

## Elastic IP

- Fixed public IPv4 you **own until deleted**
- Attach to **one instance at a time**
- Mask a failure by remapping to another instance
- **Max 5 per account** (can ask to raise)
- **Avoid**: usually poor architecture. Prefer random public IP + DNS name, or better a **Load Balancer**

## Placement Groups

| Strategy | Layout | Pros | Cons | Use case |
|---|---|---|---|---|
| **Cluster** | Same AZ, same rack/low-latency group | 10 Gbps between instances | AZ failure = all fail | Big data job needing speed, ultra-low latency |
| **Spread** | Different hardware, can span AZs | Reduced simultaneous failure | **Max 7 instances per AZ per group** | Critical apps needing max availability |
| **Partition** | Partitions = different racks, within/across AZs | Scales to 100s of instances; partition failure isolated | — | Hadoop/HDFS, HBase, Cassandra, Kafka |
| **Precision time** | Hardware with direct high-precision time sources | PTP clock, enhanced time sync | — | Distributed DBs, financial timestamping |

- Partition: up to **7 partitions per AZ**; instance can read its partition info via metadata.

## ENI (Elastic Network Interface)

Virtual network card in a VPC. Attributes:
- Primary private IPv4 + secondary IPv4s
- One Elastic IP per private IPv4
- One public IPv4
- One or more security groups
- MAC address

Create independently, **attach/move between instances** for failover. **Bound to one AZ.**

## EC2 Hibernate

| | Stop | Terminate | Hibernate |
|---|---|---|---|
| Disk (EBS) | Kept | Root deleted (if set) | Kept |
| RAM | Lost | Lost | **Saved to root EBS file** |
| Boot | OS boots again | — | Fast — OS not restarted, apps resume |

Normal start = OS boot, then app start and cache warm-up (slow). Hibernate keeps in-memory state.

Requirements:
- Root volume **EBS, encrypted**, not instance store, large enough
- RAM **< 150 GB**
- Not for bare metal
- Works with On-Demand, Reserved, Spot
- Max hibernation **60 days**
- Supported OS: Amazon Linux 2, Ubuntu, RHEL, CentOS, Windows…

Use cases: long-running processing, save RAM state, slow-initializing services.

## Exam Hints

- Need fixed IP → Elastic IP (but DNS/ELB is better)
- Low latency HPC → Cluster; HA critical → Spread; big distributed data stores → Partition
- Fast boot with warm cache → Hibernate (encrypted root EBS)
- Failover network identity → move an ENI
