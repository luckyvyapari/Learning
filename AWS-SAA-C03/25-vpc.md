# 25 — Amazon VPC (Networking)

## CIDR (IPv4)

Classless Inter-Domain Routing — defines an IP range. **Base IP + subnet mask** (`/N`).

| CIDR | IPs | Range example |
|---|---|---|
| `/32` | 1 (2⁰) | single IP |
| `/31` | 2 | .0–.1 |
| `/30` | 4 | .0–.3 |
| `/29` | 8 | .0–.7 |
| `/28` | 16 | .0–.15 |
| `/27` | 32 | .0–.31 |
| `/26` | 64 | .0–.63 |
| `/25` | 128 | .0–.127 |
| `/24` | 256 | `192.168.0.0–192.168.0.255` |
| `/16` | 65,536 | `192.168.0.0–192.168.255.255` |
| `/0` | all IPs | `0.0.0.0/0` |

Memo: `/32` no octet changes · `/24` last octet · `/16` last 2 octets · `/8` last 3 octets · `/0` all.

**Private ranges (IANA)**: `10.0.0.0/8` (big networks) · `172.16.0.0/12` (**AWS default VPC**) · `192.168.0.0/16` (home). Everything else is public.

## VPC Basics

- **Default VPC**: every new account has one; instances launch into it if no subnet specified; has internet connectivity, public IPv4s and DNS names
- **Max 5 VPCs per region** (soft limit); **max 5 CIDRs per VPC**; each CIDR **min /28, max /16**
- Only **private IPv4 ranges** allowed; **don't overlap** with other networks (corporate)

### Subnets
- Tied to **one AZ**; has a CIDR
- **AWS reserves 5 IPs per subnet** (first 4 + last 1). For `10.0.0.0/24`: `.0` network, `.1` VPC router, `.2` Amazon DNS, `.3` future use, `.255` broadcast (unsupported)
- Exam: need **29 IPs** → `/27` (32−5=27) too small → choose **`/26`** (64−5=59)

### Internet Gateway (IGW)
Horizontally scaled, HA, **created separately** from VPC; **1 VPC ↔ 1 IGW**. **Doesn't give internet access alone — edit route tables** (`0.0.0.0/0 → igw`).

### Bastion Host
Public-subnet EC2 to **SSH into private instances**. Bastion SG: allow port 22 from **restricted CIDR** (corporate public CIDR). Private instance SG: allow **bastion's SG** (or its private IP).

## NAT

| | NAT Gateway | NAT Instance (outdated, still on exam) |
|---|---|---|
| Managed | **AWS** | You |
| HA | HA **within an AZ** (create one per AZ; no cross-AZ failover needed) | Script + multi-AZ ASG |
| Bandwidth | **5 Gbps → auto 100 Gbps** | Depends on instance type |
| Cost | Per hour + data | Per hour + instance + network |
| Security Groups | **None** to manage | You manage |
| Elastic IP | Required (uses EIP) | Must attach EIP |
| Bastion use | No | Yes |
| Setup | Created in **public subnet**, **requires IGW** (private → NATGW → IGW); **can't be used by EC2 in the same subnet** | Public subnet; **disable Source/Destination check** |

- **Regional NAT Gateway**: one HA NAT shared across AZs, own route tables, no need for public subnets, auto-expands to new AZs

## Security Groups vs NACLs

| | Security Group | NACL |
|---|---|---|
| Level | **Instance** (ENI) | **Subnet** |
| Rules | **Allow only** | **Allow and Deny** |
| State | **Stateful** (return traffic auto-allowed) | **Stateless** (must allow return traffic — **ephemeral ports**) |
| Evaluation | **All rules** evaluated | **In order, lowest number first, first match wins** |
| Applies | When assigned to an instance | **All instances in subnet** automatically |

NACL details: one NACL per subnet (new subnets get the **Default NACL**), rules numbered **1–32766**, last rule `*` denies, add in increments of **100**. **New custom NACLs deny everything**; **Default NACL allows everything** (don't modify it). Great for **blocking a specific IP** at subnet level.

**Ephemeral ports**: client connects to fixed port (443), response returns on ephemeral port — Windows 10 / IANA **49152–65535**, many Linux **32768–60999**. Web→DB example: web NACL allows outbound 3306 to DB CIDR + inbound 1024–65535 from DB CIDR; DB NACL mirrors it. Create rules per target subnet CIDR.

Request path: NACL inbound → SG inbound → instance → SG outbound → NACL outbound.

## VPC Peering

Privately connect **two VPCs** via AWS network (behave as one). **No overlapping CIDRs**, **NOT transitive** (A–B, B–C ≠ A–C; need a connection per pair). **Update route tables** in both. Works **cross-account and cross-region**. Can reference a **security group in a peered VPC** (cross-account, **same region**).

## VPC Endpoints (PrivateLink)

Private access to AWS services **without IGW/NATGW/public internet**; redundant, scale horizontally. Troubleshoot: DNS resolution settings, route tables.

| | Interface Endpoint | Gateway Endpoint |
|---|---|---|
| Mechanism | **ENI** with private IP (attach **Security Group**) | Target in **route table** (no SG) |
| Services | **Most** AWS services (SNS, SQS, CloudFormation, SSM…) | **S3 and DynamoDB only** |
| Cost | $ per hour + per GB | **Free** |

**S3: Gateway endpoint usually the exam answer** (free). **Interface endpoint** when access needed **from on-prem (S2S VPN/Direct Connect), another VPC, or another region**.

**Lambda in VPC → DynamoDB**: option 1 NAT GW + IGW (costly); **option 2 (better, free): Gateway endpoint for DynamoDB + route table update**.

## VPC Flow Logs

Capture IP traffic for **VPC, subnet, or ENI** level; also AWS-managed interfaces (ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway). Send to **S3, CloudWatch Logs, Firehose**.

Fields: `srcaddr, dstaddr, srcport, dstport, protocol, packets, bytes, start, end, action (ACCEPT/REJECT), log-status`. Analyze with **Athena** (S3) or **CloudWatch Logs Insights**. Needs IAM service role with `logs:CreateLogGroup/CreateLogStream/PutLogEvents`.

**Troubleshoot using ACTION**:
| Result | Cause |
|---|---|
| Inbound REJECT | NACL **or** SG |
| Inbound ACCEPT + Outbound REJECT | **NACL** (SG is stateful) |
| Outbound REJECT | NACL or SG |
| Outbound ACCEPT + Inbound REJECT | **NACL** |

Architectures: Flow Logs → CloudWatch Logs → **Contributor Insights** (top-10 IPs) / **Metric Filter → Alarm → SNS** (SSH/RDP) · Flow Logs → S3 → **Athena → QuickSight**.

## Hybrid Connectivity

### Site-to-Site VPN
- **Virtual Private Gateway (VGW)** = VPN concentrator on AWS side (attached to VPC; custom ASN)
- **Customer Gateway (CGW)** = device/software on customer side. Use **public internet-routable IP**, or NAT device's public IP if behind NAT-T
- **Enable Route Propagation** for VGW in the subnet route table
- To **ping EC2 from on-prem** → allow **ICMP** inbound in SG
- Goes over **public internet**, encrypted

### VPN CloudHub
Low-cost **hub-and-spoke** VPN between **multiple sites** — multiple VPN connections on the **same VGW**, dynamic routing, route tables. Over public internet.

### Direct Connect (DX)
**Dedicated private connection** from your DC to AWS Direct Connect location → VPC (needs **VGW**). Access **public (S3) and private (EC2)** on the same connection. Use: high bandwidth for large data (lower cost), consistent latency (real-time feeds), hybrid. IPv4 + IPv6.
- **Private virtual interface** (VPC) and **public virtual interface** (S3/Glacier)
- **Direct Connect Gateway**: DX to **VPCs in multiple regions** (same account)
- **Connection types**: **Dedicated** (1–400 Gbps, physical port, request to AWS then partner) vs **Hosted** (50 Mbps–25 Gbps, via partners, capacity on demand); lead time often **> 1 month**
- **Encryption**: **not encrypted by default** (private). DX + **VPN = IPsec-encrypted** private connection
- **Resiliency**: High = one connection at multiple locations; **Maximum** = separate connections on separate devices in multiple locations
- **Backup**: second DX (expensive) or **Site-to-Site VPN**

### Transit Gateway
**Transitive** hub-and-spoke between **thousands of VPCs and on-prem**. Regional, works cross-region (peer TGWs), share cross-account with **RAM**, **route tables** control which VPC talks to which, works with DX Gateway + VPN. **Only service supporting IP Multicast.**
- **Site-to-Site VPN ECMP** (equal-cost multi-path): multiple VPN tunnels aggregate bandwidth. VGW: **1.25 Gbps per VPN**; TGW with ECMP: **2.5 Gbps per VPN, scales (2x=5, 3x=7.5 Gbps)**. Pay per GB processed
- **Share DX** between accounts: Transit VIF → DX Gateway → TGW, share TGW with RAM

### Other
- **PrivateLink / VPC Endpoint Services**: expose your service privately to customers' VPCs — no peering, internet, NAT, or route tables; needs **NLB + ENI**
- **ClassicLink**: EC2-Classic ↔ VPC (legacy)
- **Traffic Mirroring**: copy traffic from **ENIs** to **ENI or NLB** (security appliances) for inspection/threat monitoring; same or peered VPCs; optional filter/truncate

## IPv6

- IPv4: 4.3 billion; IPv6: 3.4×10³⁸. **Every IPv6 address in AWS is public/internet-routable** (no private range). Format: 8 hex segments (`2001:db8:3333:4444:5555:6666:7777:8888`); `::` compresses zeros
- **IPv4 can't be disabled in a VPC** → dual-stack. Can't launch an instance? Not IPv6 (huge space) → **no free IPv4 in subnet** → add a new IPv4 CIDR
- **Egress-only Internet Gateway**: **IPv6 only**, like NAT Gateway for IPv6 — outbound allowed, inbound initiation blocked; update route tables (`::/0 → eigw`)
- Public subnet routes: `0.0.0.0/0 → igw`, `::/0 → igw`; private: `0.0.0.0/0 → nat`, `::/0 → eigw`

## Networking Costs

- **Same AZ + private IP = free**; **cross-AZ ~ $0.01/GB** (private IP); **public/Elastic IP ~ $0.02/GB**; **inter-region ~ $0.02/GB**
- Use **private IP** over public for savings + performance; same AZ for max savings (at cost of HA)
- **Minimize egress** (AWS → outside is costly, ingress is free); keep traffic within AWS; Direct Connect co-located in same region lowers egress
- **S3**: ingress free; S3 → internet **$0.09/GB**; Transfer Acceleration **+$0.04–0.08**; **S3 → CloudFront $0**; CloudFront → internet $0.085 (~7x cheaper requests); CRR $0.02/GB
- **NAT Gateway vs Gateway Endpoint for S3**: NAT = **$0.045/hour + $0.045/GB processed**; **Gateway endpoint = no cost**

## AWS Network Firewall

Protect the **entire VPC**, **Layer 3 to 7**, any direction (VPC↔VPC, outbound, inbound, to/from DX & S2S VPN). Internally uses **Gateway Load Balancer**; rules managed centrally with **Firewall Manager**. 1000s of rules: IP/port (10,000s of IPs), protocol (block SMB outbound), **stateful domain list** (allow only `*.mycorp.com`), regex, allow/drop/alert, active flow inspection (**intrusion prevention**); logs → S3, CloudWatch Logs, Firehose.

**Network protection toolbox**: NACLs, SGs, WAF, Shield, Firewall Manager, **Network Firewall** (sophisticated, VPC-wide).

## Exam Hints

- Private instances need internet → **NAT Gateway** (public subnet, IGW); IPv6 → **Egress-only IGW**
- Private S3/DynamoDB access → **Gateway Endpoint** (free); other services / on-prem → **Interface Endpoint**
- Block one IP → **NACL**; stateful instance firewall → **SG**
- Many VPCs + on-prem, transitive → **Transit Gateway**; two VPCs → **Peering** (non-transitive)
- Dedicated private line → **Direct Connect**; quick encrypted → **Site-to-Site VPN**; encrypted DX → **DX + VPN**
- Investigate REJECTs → **VPC Flow Logs**
- Expose service to other VPCs privately → **PrivateLink** (NLB)
