# 03 — EC2 Basics

**EC2 = Elastic Compute Cloud = IaaS.** Rent VMs (EC2), store data on virtual drives (EBS), distribute load (ELB), scale (ASG).

## Configuration Options

| Option | Choices |
|---|---|
| OS | Linux, Windows, macOS |
| CPU / RAM | Instance type |
| Storage | Network-attached (EBS, EFS) or hardware (Instance Store) |
| Network | Card speed, public IP |
| Firewall | Security group |
| Bootstrap | **EC2 User Data** |

**User Data**: script run **only once, at first start**, as **root**. Used for updates, installing software, downloading files.

## Instance Types

Naming: `m5.2xlarge` = **m** (class) · **5** (generation) · **2xlarge** (size)

| Family | Good for | Examples of use |
|---|---|---|
| **General Purpose** (t, m) | Balanced CPU/RAM/network | Web servers, code repos |
| **Compute Optimized** (c) | High-performance CPU | Batch, media transcoding, HPC, ML, game servers |
| **Memory Optimized** (r, x) | Large in-memory datasets | In-memory DBs, caches, BI, real-time big data |
| **Storage Optimized** (i, d) | High sequential I/O on local storage | OLTP, NoSQL, data warehouse, distributed FS |
| Accelerated / HPC | GPUs, special hardware | ML, HPC |

Course uses `t2.micro` (general purpose).

## Security Groups

Firewall around EC2. **Rules only (allow only)**.

| Fact | Detail |
|---|---|
| Controls | Ports, IPv4/IPv6 ranges, inbound + outbound |
| Default | **All inbound blocked, all outbound allowed** |
| Reference | By IP **or by another security group** |
| Scope | Region + VPC combination |
| Attach | Many SGs per instance, one SG to many instances |
| Location | Lives **outside** EC2 — blocked traffic never reaches the instance |

**Debug rule:**
- **Timeout** → security group problem
- **Connection refused** → application error / not running

Tip: keep a separate SG just for SSH.

## Classic Ports

| Port | Protocol |
|---|---|
| 22 | SSH (Linux login) + SFTP |
| 21 | FTP |
| 80 | HTTP |
| 443 | HTTPS |
| 3389 | RDP (Windows login) |

## Connecting

| Method | Notes |
|---|---|
| SSH (Mac/Linux/Win10+) | Port 22, key file |
| PuTTY (Windows <10) | Free SSH client |
| **EC2 Instance Connect** | Browser-based, AWS pushes a temp key; works out-of-box on **Amazon Linux 2**; port 22 must be open |

## Purchasing Options

| Option | Discount | Commitment | Use for |
|---|---|---|---|
| **On-Demand** | none (highest cost) | none; per-second billing (Linux/Windows) | Short, unpredictable |
| **Reserved (1 or 3 yr)** | up to **72%** | Specific type/region/OS/tenancy | Steady-state (DB) |
| **Convertible RI** | up to 66% | Can change type/family/OS/scope | Flexible long-term |
| **Savings Plan (1 or 3 yr)** | up to 72% | $/hour commitment, locked to family + region | Long workloads, flexible size/OS/tenancy |
| **Spot** | up to **90%** | Can be lost anytime | Batch, analysis, image processing, fault-tolerant |
| **Dedicated Host** | most expensive | Entire physical server | BYOL licensing, compliance |
| **Dedicated Instance** | — | Your hardware, may share with your own account's instances; no placement control | Hardware isolation |
| **Capacity Reservation** | none | Reserve capacity in one AZ, any duration; charged On-Demand rate even if unused | Must-have AZ capacity |

Reserved details: payment No/Partial/All Upfront (more upfront = more discount); scope Regional or Zonal; can sell on RI Marketplace.

Hotel analogy: On-demand = walk in; Reserved = plan ahead; Savings Plan = pay per hour any room; Spot = bid for empty rooms (can be kicked out); Dedicated Host = whole building; Capacity Reservation = pay for a room you may not use.

## Spot In Depth

- Set **max price**; instance runs while spot price < max
- Price rises above max → **stop or terminate**, with **2 minute** warning
- **Cancel the Spot request first, then terminate instances** (cancelling the request does not terminate the instance). Only open/active/disabled requests can be cancelled
- Not for critical jobs or databases

### Spot Fleet
Set of Spot (+ optional On-Demand) instances meeting target capacity within price limits, across multiple launch pools.

| Strategy | Meaning |
|---|---|
| `lowestPrice` | Cheapest pool — short workloads |
| `diversified` | Spread across pools — availability, long workloads |
| `capacityOptimized` | Pool with best capacity |
| `priceCapacityOptimized` | **Recommended** — highest capacity first, then lowest price |

## Exam Hints

- Timeout vs connection refused → SG vs app
- Database / steady 24x7 → Reserved / Savings Plan
- Fault-tolerant batch → Spot
- BYOL / per-socket license → Dedicated Host
- Unpredictable short → On-Demand
