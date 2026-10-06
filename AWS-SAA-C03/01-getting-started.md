# 01 — Getting Started with AWS

## Global Infrastructure

| Term | What it is | Remember |
|---|---|---|
| **Region** | Cluster of data centers (e.g. `us-east-1`, `eu-west-3`) | Most services are **region-scoped** |
| **Availability Zone (AZ)** | 1+ isolated data centers inside a region (e.g. `ap-southeast-2a`) | 3 per region typical (min 3, max 6); low-latency links between AZs |
| **Edge Location / PoP** | Content delivery points (400+ in 90+ cities) | Used by CloudFront, Route 53, Global Accelerator |

## How to Choose a Region

| Factor | Why |
|---|---|
| **Compliance** | Data never leaves a region without your permission |
| **Latency** | Pick region close to customers |
| **Available services** | New services/features not in every region |
| **Pricing** | Varies per region |

## Global vs Regional Services

| Global | Regional |
|---|---|
| IAM | EC2 |
| Route 53 | Elastic Beanstalk |
| CloudFront | Lambda |
| WAF | Most others (S3 bucket names are global, buckets live in a region) |

## Service Models (examples)

| Model | AWS example |
|---|---|
| IaaS | EC2 |
| PaaS | Elastic Beanstalk |
| FaaS | Lambda |
| SaaS | Rekognition |
