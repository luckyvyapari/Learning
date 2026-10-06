# 18 — Serverless & Decoupled Architectures

## Mobile app: MyTodoList

Requirements: REST HTTPS API, serverless, users access **their own S3 folder**, managed serverless auth, read-heavy to-dos.

Build-up:
1. **API Gateway → Lambda → DynamoDB** (REST API), clients authenticate via **Cognito User Pools**, API Gateway verifies token
2. **Cognito Identity Pool** gives temporary credentials → client uses **restricted S3 policy** (per-user folder) directly
3. Add **DAX** between Lambda and DynamoDB for high read throughput
4. Add **API Gateway response caching**

Lessons: serverless REST API; Cognito for temp credentials to S3 (pattern also works for DynamoDB, Lambda); DAX caching; API Gateway caching; Cognito auth.

## Serverless Hosted Website: MyBlog.com

Global scale, rarely written / often read, static + dynamic, cache everywhere, welcome email, thumbnails.

| Need | Solution |
|---|---|
| Static content, globally | **CloudFront + S3**, secured with **OAC + bucket policy** |
| Dynamic REST API | **API Gateway → Lambda → DynamoDB** (+ **DAX**); no Cognito needed (public) |
| Global data | **DynamoDB Global Tables** (or Aurora Global Database) |
| Welcome email | **DynamoDB Streams → Lambda → SES** (Lambda IAM role allows SES) |
| Thumbnails | Upload (optionally via **Transfer Acceleration**) → S3 → trigger Lambda (via SQS/SNS/Lambda) → thumbnail in S3 |

S3 can trigger **SQS / SNS / Lambda** for event notifications.

## Microservices Architecture

Many services talk via REST; each service may use a different stack; goal = leaner lifecycle per service.

- Example: Route 53 → `service1` (ELB + ECS + DynamoDB), `service2` (API Gateway + Lambda + ElastiCache), `service3` (ELB + EC2 ASG + RDS)
- **Synchronous**: API Gateway, load balancers. **Asynchronous**: SQS, SNS, Kinesis, Lambda triggers (S3)
- **Challenges**: repeated setup overhead, server density/utilization, multiple versions running, client-side integration code
- **Serverless helps**: API Gateway + Lambda auto-scale and pay per use; clone APIs/environments easily; generate client SDKs via Swagger

## Software Updates Offloading

EC2 app (ASG across AZs + EFS) distributes software updates; spikes are costly. Fix **without changing the app**: put **CloudFront** in front.
- Update files are **static** → cached at the edge
- CloudFront is serverless and scales automatically; ASG scales far less → big **EC2 savings**, less bandwidth cost, more availability
- Easy way to make an existing app **more scalable and cheaper**

## Exam Hints

- Static, rarely-changing content hit by huge traffic → **CloudFront**
- Direct user access to own S3 prefix → **Cognito Identity Pool** + IAM policy with `${cognito-identity.amazonaws.com:sub}`
- Cache at both API and DB level → **API Gateway cache + DAX**
- Welcome email on new user → **DynamoDB Streams + Lambda + SES**
