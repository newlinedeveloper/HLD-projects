 **System Design** for a **Distributed Cache System** like **Memcached** or **Redis**
---

## 🧩 1. Functional Requirements

* Store and retrieve key-value pairs with low latency
* TTL (Time to Live) support for automatic expiration
* Support for **basic data structures** (string, list, set, hash, sorted set)
* Eviction policies for memory management
* Data replication for high availability
* Pub/Sub mechanism (optional, for event-driven systems)
* Cache invalidation
* Support for horizontal scalability (sharding)

---

## 📌 2. Non-Functional Requirements

* **High Availability** (99.99%)
* **Low Latency** (< 1ms for GET, \~5ms for SET)
* **Horizontal Scalability**
* **Fault Tolerance**: No single point of failure
* **Consistency**: Eventual or strong (depending on use case)
* **Durability**: Optional (for persistent cache)
* **Security**: Auth, TLS encryption

---

## 📐 3. Back-of-the-Envelope Estimation

* **Users**: 50 million DAU
* **Cache Reads/User**: 100 per day → 5B reads/day
* **Cache Writes/User**: 10 per day → 500M writes/day
* **QPS**:

  * Reads: \~57,870 QPS
  * Writes: \~5,787 QPS
* **Average object size**: 512 bytes
* **Total daily data transfer**: 5B \* 512B = \~2.5 TB/day
* **Total storage** (hot cache): 500 GB - 2 TB depending on retention and TTL

---

## 🗄️ 4. Database Design

As it’s a cache, a traditional RDBMS schema is unnecessary. But we can describe **data formats**:

### Key-Value Store Schema (in memory)

| Field       | Type      | Description                  |
| ----------- | --------- | ---------------------------- |
| key         | String    | Cache key (e.g., `user:123`) |
| value       | Any       | Cached value                 |
| ttl         | Timestamp | Expiration time              |
| created\_at | Timestamp | Insert time (optional)       |
| version     | Integer   | For optimistic locking       |

### Optional Persistent Backup (RDB/AOF for Redis)

---

## 🧪 5. API Design

### **Key Operations**

| Method | Endpoint            | Description              |
| ------ | ------------------- | ------------------------ |
| GET    | `/cache/:key`       | Get cached value         |
| POST   | `/cache`            | Set key-value with TTL   |
| DELETE | `/cache/:key`       | Delete key               |
| POST   | `/cache/invalidate` | Invalidate multiple keys |
| GET    | `/cache/stats`      | Get hit/miss stats       |

### Sample Request (POST `/cache`)

```json
{
  "key": "user:123",
  "value": {"name": "John"},
  "ttl": 300
}
```

---

## 🏗️ 6. High-Level Components (AWS Architecture)

```
[ Clients ]
    |
    ▼
[ API Gateway ]
    |
    ▼
[ AWS Lambda or EC2 App ]
    |
    ▼
[ Elasticache (Redis/Memcached) ]
    |          \
    ▼           ▼
[ CloudWatch ] [ S3 (AOF/RDB Backup) ]
```

### AWS Services Used:

| Component        | AWS Service                              |
| ---------------- | ---------------------------------------- |
| In-memory Cache  | **Amazon ElastiCache** (Redis/Memcached) |
| API Gateway      | **Amazon API Gateway** or **ALB**        |
| App Logic        | **EC2 / ECS / Lambda**                   |
| Monitoring       | **CloudWatch**                           |
| Backup           | **Amazon S3** for Redis AOF/RDB          |
| Replication & HA | **Multi-AZ Redis Replication**           |
| Security         | **IAM, VPC, KMS, TLS**                   |

---

## 🧠 7. Addressing Key Issues

### ✅ **1. Eviction Policies**

When memory is full:

* **LRU (Least Recently Used)** – Default in Redis
* **LFU (Least Frequently Used)**
* **TTL Expiry** – Auto-expire based on time
* **Random** – Removes random keys under memory pressure

**Solution**: Choose eviction strategy per use case. Redis allows configuring eviction policy with `maxmemory-policy`.

---

### ✅ **2. Replication & High Availability**

* Use **master-replica replication**
* Automatic **failover** using Redis Sentinel or AWS managed Redis
* Write to **primary**, read from **replicas**

**Solution**:

* Redis Cluster mode
* AWS Multi-AZ ElastiCache
* Sentinel or Raft for automatic leader election

---

### ✅ **3. Consistency**

* Redis: Eventual consistency with replicas
* Use **strong consistency** if app only reads/writes from master
* Use **optimistic locking (WATCH/MULTI/EXEC)** for concurrency control

**Solution**:

* Cache aside pattern ensures source of truth is always DB
* Add versioning to handle concurrent writes

---

### ✅ **4. Cache Miss Handling**

* On **miss**, fetch data from persistent DB (Postgres/DynamoDB)
* Populate cache with fresh data (`write-back` or `write-through`)
* Add **negative caching** (e.g., "user not found") to reduce DB hits

---

### ✅ **5. Persistence and Durability**

* Redis: Offers AOF (append-only file) and RDB (snapshotting)
* Memcached: No persistence
* AWS: Use **S3 backup**, CloudWatch alarms on failure

---

### ✅ **6. Security**

* Use **encryption in transit** (TLS)
* Enable **AUTH** for clients
* Host inside **VPC**, restrict access using security groups and IAM

---

### ✅ **7. Monitoring and Alerting**

* **Cache hit/miss ratio**
* **Evictions, memory usage**
* **QPS (requests per second)**
* Use **CloudWatch** or Prometheus + Grafana

---

## ✅ Patterns Supported

* **Cache-aside**: App manages population
* **Write-through**: Cache updated on every write
* **Write-back**: Lazy write to DB (complex, less durable)

---
