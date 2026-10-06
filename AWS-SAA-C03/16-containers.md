# 16 — Containers on AWS (Docker, ECS, Fargate, ECR, EKS)

## Docker

Software platform packaging apps in **containers** that run identically anywhere (no compatibility issues, predictable, easy to deploy). Use: **microservices**, **lift-and-shift** to AWS.

| | Docker container | Virtual Machine |
|---|---|---|
| Layers | App → Docker daemon → Host OS | App → Guest OS → Hypervisor → Host OS |
| Resources | **Shared with host** → many containers per server | Dedicated per VM |

Flow: **Dockerfile → build image → push/pull from repository → run container**.

**Image repositories**: **Docker Hub** (public) · **Amazon ECR** (private, plus **ECR Public Gallery**).

## Container Services on AWS

| Service | What |
|---|---|
| **ECS** | Amazon's own container orchestration platform |
| **EKS** | Managed **Kubernetes** (open source, cloud-agnostic) |
| **Fargate** | **Serverless** container compute — works with **ECS and EKS** |
| **ECR** | Store container images |

## ECS

### Launch types

| | EC2 launch type | Fargate launch type |
|---|---|---|
| Infrastructure | **You provision/maintain EC2** instances | **Serverless** — no EC2 to manage |
| Agent | Each instance runs **ECS Agent** to join cluster | — |
| Scaling | Add instances + tasks | Just **increase number of tasks** |
| You define | Task definition | Task definition (CPU/RAM) |

### IAM Roles
- **EC2 Instance Profile** (EC2 launch type only): used by the **ECS agent** — call ECS API, send logs to **CloudWatch Logs**, pull images from **ECR**, reference **Secrets Manager / SSM Parameter Store**
- **ECS Task Role**: **per-task permissions** (task A → S3, task B → DynamoDB); defined in the **task definition**

### Load balancers
- **ALB** — supported, works for most cases
- **NLB** — only for high throughput/performance, or pairing with **PrivateLink**
- **CLB** — supported, not recommended (no advanced features, no Fargate)

### Data volumes
- **EFS**: works with **EC2 and Fargate**; shared across AZs; **Fargate + EFS = serverless** persistent storage
- **S3 Files** (S3 as file system): Fargate and ECS Managed Instances (**not** EC2 launch type); changes sync to S3

### Service Auto Scaling (task level)
Uses **Application Auto Scaling**. Metrics: **service avg CPU**, **avg memory**, **ALB request count per target**. Policies: target tracking, step scaling, scheduled.
- **ECS Service Auto Scaling (tasks) ≠ EC2 Auto Scaling (instances)**
- Fargate scaling is simpler
- EC2 launch type: scale ASG on CPU, or use **ECS Cluster Capacity Provider** paired with an ASG to add instances when capacity (CPU/RAM) is missing

### Event-driven patterns
- **EventBridge → run ECS task**: e.g. S3 upload event triggers Fargate task (task role accesses S3 + DynamoDB)
- **EventBridge schedule**: run batch task every hour
- **SQS queue** consumed by ECS service; scale tasks on queue length
- **Intercept stopped tasks**: EventBridge rule on task state change → **SNS** → email admin

## ECR

Private/public Docker image registry; **backed by S3**, integrated with ECS, access via **IAM** (permission errors → policy), supports **vulnerability scanning**, versioning, tags, lifecycle.

## EKS

Managed Kubernetes. **Alternative to ECS** (same goal, different API). Pick when you already use Kubernetes on-prem/other cloud (portable). Runs on **EC2** worker nodes or **Fargate**. **One EKS cluster per region** for multi-region. Logs/metrics via **CloudWatch Container Insights**.

| Node type | Description |
|---|---|
| **Managed Node Groups** | EKS creates/manages EC2 nodes (part of ASG); On-Demand or Spot |
| **Self-Managed Nodes** | You create/register, managed by ASG; can use **EKS Optimized AMI**; On-Demand or Spot |
| **Fargate** | No nodes to maintain |

**Data volumes**: need a **StorageClass** manifest + **CSI-compliant driver**. Supports **EBS, EFS (works with Fargate), FSx for Lustre, FSx for NetApp ONTAP**.

## Exam Hints

- Run containers without managing servers → **Fargate**
- Company already on Kubernetes → **EKS**
- Per-task AWS permissions → **ECS Task Role**
- Shared persistent storage across tasks/AZs → **EFS**
- Schedule container job → **EventBridge + ECS/Fargate task**
- Arbitrary Docker image on AWS → **ECS/Fargate** (not Lambda)
