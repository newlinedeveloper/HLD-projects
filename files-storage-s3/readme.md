Here's a comprehensive **System Design** for building a **distributed file storage system like Amazon S3**, touching all major aspects: **functional**, **non-functional**, **estimates**, **database design**, **API design**, **HLD**, and **key issue resolutions**.

---

## ✅ 1. Functional Requirements

| Feature                        | Description |
|-------------------------------|-------------|
| **Upload File**               | Users can upload objects (files, images, videos, etc.) |
| **Download File**             | Users can retrieve stored objects |
| **Delete File**               | Users can delete stored objects |
| **Versioning** *(optional)*   | Ability to maintain object versions |
| **Buckets**                   | Logical containers to group objects |
| **Authentication & ACLs**     | Support object-level access control |
| **Metadata Support**          | Attach key-value metadata to objects |

---

## ✅ 2. Non-Functional Requirements

| Requirement            | Details |
|------------------------|---------|
| **Scalability**         | Handle petabytes of data & millions of users/files |
| **Availability**        | ≥ 99.99% uptime (use multi-zone replication) |
| **Durability**          | ≥ 99.999999999% (11 9s) durability via replication |
| **Low Latency**         | ≤ 100 ms for object access in typical use |
| **Security**            | Encryption (at rest & in transit), ACLs |
| **Fault Tolerance**     | Detect and recover from node failures |
| **Consistency**         | Eventual or strong consistency depending on API |

---

## ✅ 3. Back of the Envelope Estimation

| Metric                        | Estimate |
|------------------------------|----------|
| **Daily uploads**             | 10 million objects/day |
| **Avg object size**           | 1 MB |
| **Daily storage need**        | 10 TB/day |
| **1-year storage**            | ~3.6 PB |
| **QPS (uploads/downloads)**   | 10,000–50,000 |
| **Metadata per object**       | 1 KB |

---

## ✅ 4. Database Design (Metadata Store)

We separate **file storage (blobs)** from **metadata storage**.

### 📘 Object Metadata Table

| Field         | Type        | Description                       |
|---------------|-------------|-----------------------------------|
| `object_id`   | UUID (PK)   | Unique ID                         |
| `bucket_id`   | FK          | Refers to user’s logical bucket   |
| `filename`    | VARCHAR     | Original file name                |
| `version_id`  | UUID        | Optional for versioning           |
| `size_bytes`  | BIGINT      | File size                         |
| `content_type`| VARCHAR     | MIME type                         |
| `created_at`  | TIMESTAMP   | Upload time                       |
| `checksum`    | STRING      | SHA256 or MD5 hash                |
| `replicas`    | JSONB       | List of storage node locations    |
| `is_deleted`  | BOOL        | For soft deletes                  |

---

## ✅ 5. RESTful API Design

| Method | Endpoint                      | Description                     |
|--------|-------------------------------|---------------------------------|
| POST   | `/buckets/:bucket/upload`     | Upload object                   |
| GET    | `/buckets/:bucket/:file`      | Download object                 |
| DELETE | `/buckets/:bucket/:file`      | Delete object                   |
| GET    | `/buckets/:bucket/list`       | List objects in a bucket        |
| GET    | `/buckets/:bucket/meta/:file` | Get metadata                    |
| PUT    | `/buckets/:bucket/acl`        | Set access control              |

---

## ✅ 6. High-Level Components

```
           +---------------------+
           |    Load Balancer    |
           +---------------------+
                    |
        +-----------+-----------+
        |                       |
+----------------+     +----------------+
| API Gateway     |     | Auth Service   |
+----------------+     +----------------+
        |
        v
+-----------------------+
|  Metadata Service     | <---> SQL/NoSQL DB
+-----------------------+
        |
        v
+----------------------------+
| Storage Manager (Scheduler)| <-- Track replication, healing
+----------------------------+
        |
        v
+-----------+  +-----------+  +-----------+
| Blob Node |  | Blob Node |  | Blob Node |
| (S3-like) |  | (S3-like) |  | (S3-like) |
+-----------+  +-----------+  +-----------+

```

---

## ✅ 7. Key Issues & Solutions

| Issue                        | Explanation & Solution |
|-----------------------------|------------------------|
| **Data durability**         | Use **replication** across availability zones (e.g., 3 copies) or **erasure coding** |
| **Scalability**             | Use **sharding by bucket/object**, scale blob nodes horizontally |
| **Metadata bottleneck**     | Cache frequently accessed metadata in **Redis**, distribute DB by bucket |
| **File deduplication**      | Use **content-based hashing (SHA256)** to detect duplicate content |
| **Large file upload**       | Use **multi-part upload**, merge parts server-side |
| **Concurrency**             | Use optimistic locking or versioning |
| **Geo-redundancy**          | Replicate to another region asynchronously |
| **Authentication & ACLs**  | Integrate with OAuth, JWT; Store ACLs with metadata |
| **Indexing/Search**         | Use ElasticSearch or S3 Inventory-like service for large-scale listing |
| **Garbage Collection**      | Mark-and-sweep logic for deleted/unreferenced blobs |
| **Monitoring & Metrics**    | Expose Prometheus metrics, alert on replication lags or failures |

---

## ✅ 8. Storage Strategy

- **Write Path**:
  - Client → API Gateway → Store metadata → Upload to 3 blob nodes → Return success
- **Read Path**:
  - Client → API Gateway → Lookup metadata → Read from nearest blob node
- **Delete**:
  - Mark in metadata → Async deletion in blob nodes
- **Replication**:
  - Each blob node replicates its chunks to 2 others asynchronously
- **Multi-Part Upload**:
  - Chunk files, parallel upload → commit at server

---

## ✅ 9. Tech Stack

| Layer                  | Stack |
|------------------------|-------|
| API Layer              | Golang + Gin |
| Auth                   | JWT / OAuth |
| Metadata DB            | PostgreSQL / DynamoDB |
| Object Storage         | Custom Blob nodes (Golang, minIO-like) |
| Cache                  | Redis |
| Queue (async ops)      | Kafka or RabbitMQ |
| Monitoring             | Prometheus + Grafana |
| Load Balancer          | AWS ALB / NGINX |

---

## 🧪 Sample Scenario: Upload File

1. Client POSTs file to `/buckets/photos/upload`
2. API Gateway authenticates JWT token
3. Metadata Service:
   - Stores filename, checksum, content-type
   - Generates `object_id`
   - Picks 3 blob nodes for replication
4. Client uploads to selected blob nodes (direct pre-signed URLs or via proxy)
5. Metadata is marked as **active** when upload completes

---

Would you like a basic prototype (e.g., using Gin + MinIO backend) to simulate this architecture?
