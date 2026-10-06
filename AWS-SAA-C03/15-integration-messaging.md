# 15 — Integration & Messaging (SQS, SNS, Kinesis, MQ)

Applications communicate **synchronously** (app → app) or **asynchronously / event-based** (app → queue → app). Sync breaks under sudden spikes → **decouple** with:
- **SQS** — queue model
- **SNS** — pub/sub
- **Kinesis** — real-time streaming

They scale independently of your apps.

## SQS (Simple Queue Service)

### Standard Queue
- Oldest service, fully managed
- **Unlimited throughput + unlimited messages**
- Retention: **default 4 days, max 14 days**
- Latency < 10 ms
- Max **1,024 KB** per message
- **At-least-once** delivery (can have duplicates) and **best-effort ordering** (can be out of order)

### Producing / Consuming
- Produce: `SendMessage` (SDK); persists until a consumer deletes it
- Consume: EC2 / servers / **Lambda** **poll**, receive **up to 10 messages**, process, then call **`DeleteMessage`**
- Multiple consumers process in parallel → scale horizontally
- **ASG + CloudWatch alarm on `ApproximateNumberOfMessages`** (queue length) scales consumers
- Use SQS to **decouple tiers** (web front end → queue → back-end processing) and as a **buffer for DB writes** (avoid lost transactions on spikes)

### Security
- In flight: HTTPS · at rest: **KMS** · client-side optional
- IAM policies + **SQS access policies** (cross-account, allow SNS/S3 to write)

### Visibility Timeout
- After a message is polled it's **invisible** to others — default **30 s**
- Not processed in time → becomes visible again → **processed twice**
- Consumer can call **`ChangeMessageVisibility`** for more time
- Too high: crash → slow reprocess. Too low: duplicates

### Long Polling
- Consumer **waits** for messages when queue empty → fewer API calls, lower latency
- Wait **1–20 s** (20 preferred); set at **queue level** or API (`WaitTimeSeconds`)
- Preferred over short polling

### FIFO Queue
- Ordered (First In First Out)
- Throughput: **300 msg/s** (3,000 with batching)
- **Exactly-once** send via **Deduplication ID**
- Ordering by **Message Group ID** (mandatory)

## SNS (Simple Notification Service)

One message → **many receivers** (pub/sub).
- Producer publishes to **one topic**; each **subscriber gets all messages** (unless filtered)
- Up to **12,500,000 subscriptions per topic**, **100,000 topics**
- Subscribers: **SQS, Lambda, Kinesis Data Firehose, Email, SMS & mobile push, HTTP(S) endpoints**
- Many AWS services publish to SNS: CloudWatch Alarms, Budgets, ASG, S3, DynamoDB, CloudFormation, DMS, RDS events…
- Publish: **Topic publish** (create topic → subscribe → publish) or **Direct publish** for mobile (platform application + endpoint; GCM, APNS, ADM)
- Security: HTTPS, KMS, IAM + **SNS access policies** (cross-account, allow S3 to write)

### SNS + SQS Fan-Out
Push once to SNS → all subscribed SQS queues receive it. **Fully decoupled, no data loss**, persistence/retry via SQS, add subscribers anytime, **cross-region** delivery. Queue access policy must allow SNS.
- Same S3 event type + prefix allows only **one** S3 event rule → use **fan-out** to reach many queues
- **SNS → Kinesis Data Firehose → S3 / other destination**

### SNS FIFO Topic
Ordering by Message Group ID, dedup by Dedup ID or content-based, SQS Standard + FIFO subscribers, same throughput as SQS FIFO. **SNS FIFO + SQS FIFO** = fan-out + ordering + dedup.

### Message Filtering
JSON **filter policy** per subscription (e.g. `State: Placed`). No filter policy = receives everything.

## Kinesis

### Kinesis Data Streams (real-time)
- Collect/store streaming data; producers: apps, clients, SDK, **KPL**, Kinesis Agent; consumers: apps (**KCL**), Lambda, Firehose, Managed Flink
- **Retention up to 365 days**, **replay** supported, data **can't be deleted** until expiry
- Record up to **10 MiB** (usually many small records)
- Order guaranteed per **Partition ID** (per shard)
- KMS at rest, HTTPS in flight

| Capacity mode | How |
|---|---|
| **Provisioned** | Choose shards; each shard **1 MB/s or 1,000 rec/s in**, **2 MB/s out**; manual scaling; pay per shard-hour |
| **On-demand** | Auto scales (default 4 MB/s or 4,000 rec/s) from last 30 days' peak; pay per stream-hour + GB |

### Amazon Data Firehose (formerly Kinesis Data Firehose)
- Fully managed, serverless, auto scaling, pay per use, **near real-time** (buffer by size/time)
- Destinations: **S3, Redshift, OpenSearch**, 3rd-party (Splunk, MongoDB, Datadog, New Relic…), custom HTTP endpoint
- Formats: CSV, JSON, Parquet, Avro, text, binary; **convert to Parquet/ORC**, compress gzip/snappy
- **Lambda** for custom transformation (e.g. CSV→JSON); optional S3 backup of all/failed data

### Data Streams vs Firehose

| | Data Streams | Firehose |
|---|---|---|
| Latency | **Real-time** | **Near real-time** |
| Code | Custom producer + consumer | Load into destinations, no code |
| Scaling | Provisioned / on-demand | Automatic |
| Storage | **Up to 365 days** | **None** |
| Replay | **Yes** | No |

## SQS vs SNS vs Kinesis

| | SQS | SNS | Kinesis |
|---|---|---|---|
| Model | Queue, consumers **pull** | Pub/sub, **push** | Streaming; standard **pull** (2 MB/shard); **enhanced fan-out push** (2 MB/shard/consumer) |
| Persistence | Deleted after consumption | **Not persisted** if undelivered | **Replay**; expires after X days |
| Ordering | Only **FIFO** queues | FIFO topics | **Per shard** |
| Scaling | No provisioning | No provisioning | Provisioned or on-demand |
| Use | Decoupling, buffering | Fan-out, notifications | Real-time big data, analytics, ETL |

## Amazon MQ

Managed broker for **ActiveMQ / RabbitMQ**. For **migrating on-prem apps** that use open protocols (**MQTT, AMQP, STOMP, OpenWire, WSS**) without re-engineering to SQS/SNS. Has both **queue** (~SQS) and **topic** (~SNS) features. **Doesn't scale as much** as SQS/SNS; runs on servers; **Multi-AZ** with **active/standby** failover (EFS storage).

## Exam Hints

- Decouple + buffer spikes → **SQS**
- One event → many consumers → **SNS + SQS fan-out**
- Ordered, deduplicated → **FIFO**
- Real-time analytics with replay → **Kinesis Data Streams**
- Load streaming data into S3/Redshift with no code → **Firehose**
- Migrate on-prem MQTT/AMQP app → **Amazon MQ**
- Duplicate processing → raise **visibility timeout**
