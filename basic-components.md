### System design Basic Concepts & Components
---

### 🛠️ **1. APIs (Application Programming Interfaces)**  
APIs enable communication between different software systems.  
- **REST**: Stateless, resource-oriented, HTTP-based.  
- **GraphQL**: Flexible queries, single endpoint.  
- **gRPC**: High-performance, binary protocol, uses HTTP/2.  

🛠️ **AWS Services**: API Gateway, AppSync.  

---

### 🌐 **2. Content Delivery Network (CDN)**  
A network of geographically distributed servers to cache and deliver content closer to users.  
- Reduces latency, improves performance.  
- **Edge Locations**: Cache content for faster delivery.  

🛠️ **AWS Service**: CloudFront.  

---

### 🔄 **3. Proxy vs Reverse Proxy**  
- **Proxy**: Forwards client requests to external services (e.g., VPN).  
- **Reverse Proxy**: Forwards requests from clients to backend servers (e.g., Nginx).  

🛠️ **AWS Services**: AWS Global Accelerator, CloudFront.  

---

### 📡 **4. Domain Name System (DNS)**  
Resolves domain names to IP addresses.  
- **A Record**: Maps domain to IPv4.  
- **CNAME**: Maps one domain to another.  
- **TTL (Time-to-Live)**: Cache duration for DNS responses.  

🛠️ **AWS Service**: Route 53.  

---

### ⚡ **5. Caching**  
Temporarily stores frequently accessed data to improve performance.  
- **Read Cache**: Reduces database reads.  
- **Write Cache**: Buffers writes for performance.  

🛠️ **AWS Services**: ElastiCache (Redis/Memcached).  

---

### 🔍 **6. Caching Strategies**  
- **Write-through**: Write to cache and database simultaneously.  
- **Read-through**: Cache fetches data on cache miss.  
- **Lazy Loading**: Cache only when requested.  
- **Cache Eviction Policies**: LRU (Least Recently Used), LFU (Least Frequently Used).  

---

### 🌎 **7. Distributed Caching**  
Cache spread across multiple nodes for scalability.  
- **Consistent Hashing** used to manage cache distribution.  

🛠️ **AWS Services**: ElastiCache for Redis Cluster Mode.  

---

### 🚪 **8. API Gateway**  
Manages, monitors, and secures APIs.  
- **Request Throttling**, **Rate Limiting**, **Authentication** (OAuth2, JWT).  

🛠️ **AWS Service**: API Gateway, AppSync.  

---

### ⚖️ **9. Load Balancing**  
Distributes traffic across multiple servers to ensure availability and performance.  
- **Types**:  
  - **Layer 4 (TCP/UDP)**: Network-level (e.g., NLB).  
  - **Layer 7 (HTTP/S)**: Application-level (e.g., ALB).  
- **Strategies**: Round-robin, Least Connections, IP Hash.  

🛠️ **AWS Service**: Elastic Load Balancer (ALB/NLB).  

---

### 🗂️ **10. Database Types**  
- **Relational (SQL)**: Structured tables, ACID-compliant (e.g., PostgreSQL).  
- **NoSQL**: Flexible schemas (e.g., DynamoDB).  
- **Time-Series**: Metrics over time (e.g., InfluxDB).  
- **Key-Value Stores**: Fast lookups (e.g., Redis).  
- **Graph Databases**: Relationships (e.g., Neptune).  

---

### ⚔️ **11. SQL vs NoSQL**  
| **Aspect**      | **SQL**               | **NoSQL**              |
|------------------|----------------------|------------------------|
| **Structure**    | Tables (fixed schema) | Flexible schema         |
| **Scalability**  | Vertical              | Horizontal              |
| **Consistency**  | Strong consistency    | Eventual consistency    |
| **Examples**     | RDS (PostgreSQL)      | DynamoDB, MongoDB       |

---

### 🔍 **12. Database Indexes**  
Indexes speed up queries by organizing data for efficient lookups.  
- **B-Tree**: General-purpose indexing.  
- **Bitmap Index**: Efficient for low-cardinality columns.  
- **Hash Index**: Key-based lookups.  

---

### ⚖️ **13. Consistency Patterns**  
- **Strong Consistency**: Immediate consistency after a write (e.g., Aurora).  
- **Eventual Consistency**: Consistency achieved over time (e.g., DynamoDB default).  
- **Causal Consistency**: Operations are seen in causal order.  

---

### ❤️ **14. Heartbeats**  
Periodic signals sent by a system to confirm it's active.  
- Used in distributed systems for failure detection.  

🛠️ **AWS Service**: CloudWatch Alarms.  

---

### ⚠️ **15. Circuit Breaker Pattern**  
Prevents system failure by stopping requests to a failing service.  
- **Open State**: Blocks traffic when failures exceed threshold.  
- **Half-Open**: Tests service after timeout.  
- **Closed**: Normal operation.  

---

### 🔁 **16. Idempotency**  
An operation that produces the same result if executed multiple times.  
- Critical for retries in distributed systems (e.g., payment transactions).  

---

### 📈 **17. Database Scaling**  
- **Vertical Scaling**: Add more resources to a single node.  
- **Horizontal Scaling**: Add more nodes (e.g., DynamoDB).  

🛠️ **AWS Services**: RDS (vertical), DynamoDB (horizontal).  

---

### 📤 **18. Data Replication**  
Copying data across different servers/regions.  
- **Synchronous**: Real-time consistency (e.g., RDS Multi-AZ).  
- **Asynchronous**: Lagged but performant (e.g., Aurora cross-region).  

---

### 🛟 **19. Data Redundancy**  
Storing duplicate copies of data for reliability.  
- **RAID Configurations**: RAID 1 (mirroring), RAID 5 (striping with parity).  

🛠️ **AWS Service**: S3 with replication.  

---

### 🧩 **20. Database Sharding**  
Partitioning a database into smaller subsets (shards).  
- **Horizontal Sharding**: Distributes rows across shards.  
- **Vertical Sharding**: Distributes columns across shards.  

---

### 🏛️ **21. Database Architectures**  
- **Monolithic**: Single centralized database.  
- **Distributed**: Multiple databases across regions.  
- **Federated**: Independent databases for specific services.  

---

### 🚨 **22. Failover**  
Switching to a backup system in case of failure.  
- **Hot Standby**: Active backup, instant switch.  
- **Warm Standby**: Partial capacity standby.  

🛠️ **AWS Service**: RDS Multi-AZ.  

---

### 🌸 **23. Bloom Filters**  
A probabilistic data structure for membership checks.  
- **Space-efficient** but may return false positives (not false negatives).  
- Used in caching layers.  

---

### 📬 **24. Message Queues**  
Decouple components by passing messages asynchronously.  
- **FIFO**: Ensures order.  
- **Standard Queue**: Best effort ordering.  

🛠️ **AWS Service**: SQS, SNS (for pub/sub).  

---

### 🔌 **25. WebSockets**  
A protocol for real-time, bidirectional communication.  
- **Use Cases**: Chat apps, live notifications.  

🛠️ **AWS Service**: API Gateway WebSocket APIs.  

---

### ✅ **26. Checksums**  
A hash computed to detect data corruption.  
- **CRC (Cyclic Redundancy Check)** commonly used.  

🛠️ **AWS Service**: S3 integrity checks.  

---

### 🧩 **27. Microservices Guidelines**  
- **Domain-Driven Design (DDD)**: Align services with business domains.  
- **Inter-service Communication**: REST, gRPC, or event-driven.  
- **Observability**: Logs, metrics, traces.  

🛠️ **AWS Service**: ECS, App Mesh, CloudWatch.  

---

### 🔐 **28. Distributed Locking**  
Ensures mutual exclusion in distributed systems.  
- **Challenges**: Clock skew, race conditions.  
- **Implementation**: **Redlock Algorithm** with Redis.  

🛠️ **AWS Service**: ElastiCache (Redis).  

---
