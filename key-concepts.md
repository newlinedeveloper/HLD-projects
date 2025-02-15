### System design Key concepts
---

### 1. **Scalability** 🛠️  
The ability of a system to handle increased load without performance degradation.  
- **Vertical Scaling (Scaling Up)**: Increasing resources (CPU/RAM) of a single machine.  
- **Horizontal Scaling (Scaling Out)**: Adding more machines to distribute load (e.g., AWS Auto Scaling Groups).  
- **Elasticity**: Dynamic scaling based on demand (AWS Lambda, ECS Fargate).  

🛠️ **AWS Services**: Auto Scaling, ECS, Lambda, DynamoDB (on-demand capacity).  

---

### 2. **Availability** ✅  
The ability of a system to remain operational over time.  
- **High Availability (HA)**: Ensures minimal downtime with redundancy (e.g., multi-AZ RDS).  
- **SLA (Service Level Agreement)**: Uptime guarantees (e.g., 99.99%).  

🛠️ **AWS Services**: RDS Multi-AZ, S3, Route 53 (health checks).  

---

### 3. **CAP Theorem** ⚖️  
In a distributed system, you can only guarantee **two** out of three:  
- **Consistency (C)**: Every read gets the most recent write.  
- **Availability (A)**: Every request gets a response (even if stale).  
- **Partition Tolerance (P)**: System functions despite network failures.  

🛠️ **Examples**:  
- DynamoDB (AP by default, tunable consistency).  
- RDS (CP for SQL databases).  

---

### 4. **ACID Transactions** 🧱  
Guarantees for database transactions:  
- **Atomicity**: All or nothing.  
- **Consistency**: Valid state transitions.  
- **Isolation**: Independent transactions don’t affect each other.  
- **Durability**: Committed data persists.  

🛠️ **AWS Services**: RDS, Aurora (with ACID compliance).  

---

### 5. **Consistent Hashing** 🔢  
A load balancing technique that minimizes rebalancing when nodes are added/removed.  
- Commonly used in caching (e.g., Redis clusters).  

🛠️ **AWS Services**: Elasticache (Redis), DynamoDB partitioning.  

---

### 6. **Rate Limiting** 🚦  
Controls the number of requests a client can make within a time frame to prevent abuse.  
- **Token Bucket**: Tokens refill at a constant rate.  
- **Leaky Bucket**: Processes requests at a fixed rate.  

🛠️ **AWS Services**: API Gateway throttling, WAF (Web Application Firewall).  

---

### 7. **Single Point of Failure (SPOF)** ⚠️  
A component whose failure can cause the entire system to fail.  
- **Mitigation**: Redundancy, failover mechanisms.  

🛠️ **AWS Services**: Multi-AZ deployments, Route 53 health checks.  

---

### 8. **Fault Tolerance** 🛡️  
The ability of a system to continue functioning despite failures.  
- **Techniques**: Replication, retries, circuit breakers.  

🛠️ **AWS Services**: S3 (with cross-region replication), Aurora (with failover).  

---

### 9. **Consensus Algorithms** 🤝  
Algorithms to achieve agreement in distributed systems.  
- **Paxos/Raft**: Used in distributed databases for leader election.  

🛠️ **AWS Example**: Aurora uses quorum-based replication internally.  

---

### 10. **Gossip Protocol** 🗣️  
A decentralized way for nodes to share information without a central coordinator.  
- **Use Cases**: Service discovery, cluster state synchronization.  

🛠️ **AWS Services**: ECS, DynamoDB, and internal cluster management.  

---

### 11. **Service Discovery** 🔍  
Enables services to find and communicate with each other dynamically.  
- **Client-side Discovery**: Clients query a registry.  
- **Server-side Discovery**: Load balancer acts as intermediary.  

🛠️ **AWS Services**: ECS Service Discovery, Route 53, App Mesh.  

---

### 12. **API Design** 🌐  
Principles for designing robust, scalable APIs.  
- **RESTful APIs**: Stateless, resource-oriented.  
- **GraphQL**: Flexible queries.  
- **gRPC**: High-performance, binary protocol.  

🛠️ **AWS Services**: API Gateway, AppSync.  

---

### 13. **Disaster Recovery (DR)** 🌪️  
Plans to recover from catastrophic failures.  
- **Backup and Restore**: Regular backups.  
- **Pilot Light**: Minimal standby environment.  
- **Warm Standby**: Partially active resources.  
- **Multi-site Active/Active**: Full redundancy across regions.  

🛠️ **AWS Services**: S3 Cross-Region Replication, CloudEndure.  

---

### 14. **Distributed Tracing** 🔍  
Tracks requests across distributed systems to analyze performance.  
- **Techniques**: Distributed context propagation, span/trace IDs.  

🛠️ **AWS Services**: X-Ray, CloudWatch ServiceLens.  

---
