# 11 — Amazon S3 Advanced

## Lifecycle Rules

| Action | Example |
|---|---|
| **Transition** | → Standard-IA after 60 days; → Glacier after 6 months |
| **Expiration** | Delete access logs after 365 days; delete **old versions**; delete **incomplete multi-part uploads** |

Rules can target a **prefix** (`mp3/*`) or **object tags** (`Department: Finance`).

### Scenarios
1. **Thumbnails (re-creatable, keep 60 days) + source images (instant for 60 d, then wait ≤ 6 h OK)**
   - Source: **Standard → Glacier** after 60 days
   - Thumbnails: **One Zone-IA**, **expire** after 60 days
2. **Recover deleted objects immediately for 30 days, within 48 h up to 365 days**
   - Enable **Versioning** (delete = delete marker)
   - Transition **noncurrent versions** → **Standard-IA**, later → **Glacier Deep Archive**

## S3 Analytics — Storage Class Analysis

Recommends when to transition **Standard → Standard-IA**. Does **NOT** work for One Zone-IA or Glacier. Daily CSV report, starts after 24–48 h. Good first step before writing lifecycle rules.

## Requester Pays

Requester (not owner) pays request + download cost. Share large datasets. **Requester must be authenticated** (no anonymous).

## Event Notifications

Events: `ObjectCreated`, `ObjectRemoved`, `ObjectRestore`, `Replication`… Filter by name (`*.jpg`). Destinations: **SNS, SQS, Lambda** — each needs a **resource policy** allowing S3. Delivery in seconds (sometimes a minute+).

**With EventBridge**: all events, JSON rule filtering (metadata, size, name), **18+ destinations**, archive/replay, reliable delivery.

## Performance

| Topic | Fact |
|---|---|
| Baseline | Latency 100–200 ms; **3,500 PUT/COPY/POST/DELETE** or **5,500 GET/HEAD per second per prefix**; unlimited prefixes (4 prefixes → 22,000 GET/s) |
| **Multi-part upload** | Recommended > 100 MB, **required > 5 GB**; parallel uploads |
| **Transfer Acceleration** | Upload to nearest **edge location** → private AWS network to target region; compatible with multi-part |
| **Byte-range fetches** | Parallel GETs of byte ranges; speeds downloads, resilient; fetch only part (e.g. file header) |

## S3 Batch Operations

Bulk ops on existing objects with one request: modify metadata/properties, copy between buckets, **encrypt unencrypted objects**, modify ACLs/tags, **restore from Glacier**, invoke **Lambda** per object. Handles retries, progress, reports.
Get object list via **S3 Inventory** + filter with **Athena**.

## S3 Storage Lens

Analyze/optimize storage across the whole **AWS Organization** (30 days usage/activity). Aggregate by org, account, region, bucket, prefix. Default dashboard (can't delete, can disable). Export daily to S3 (CSV/Parquet).

| Metric group | Examples |
|---|---|
| Summary | StorageBytes, ObjectCount |
| Cost-optimization | NonCurrentVersionStorageBytes, IncompleteMultipartUploadStorageBytes |
| Data-protection | VersioningEnabledBucketCount, MFADeleteEnabled, SSEKMSEnabled, CRR rule count |
| Access-management | Object Ownership settings |
| Event | EventNotificationEnabledBucketCount |
| Performance | TransferAccelerationEnabledBucketCount |
| Activity | AllRequests, GetRequests, BytesDownloaded |
| Status code | 200OK, 403Forbidden, 404NotFound counts |

| | Free | Advanced (paid) |
|---|---|---|
| Metrics | ~28 usage metrics | Activity, advanced cost optimization, advanced data protection, status code |
| Retention | **14 days** | **15 months** |
| Extras | — | CloudWatch publishing, **prefix aggregation** |

## Exam Hints

- Auto-delete incomplete multi-part uploads → **lifecycle expiration**
- Speed global uploads → **Transfer Acceleration**; large files → **multi-part**
- Encrypt millions of existing objects → **Batch Operations**
- Which class to transition to → **Storage Class Analysis**
- Org-wide S3 visibility → **Storage Lens**
