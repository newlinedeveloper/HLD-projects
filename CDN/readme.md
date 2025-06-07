# System Design: Content Delivery Network (CDN)

## ✅ 1. Functional Requirements

* **Static Content Delivery**: Deliver JS, CSS, images, videos, etc.
* **Dynamic Content Acceleration**: Optimize personalized API responses (e.g., HTML pages).
* **Geo-Distribution**: Route users to the nearest Point of Presence (PoP).
* **Caching**: Store content closer to end users to reduce origin load.
* **Content Invalidation & Purging**: Support cache refresh or invalidation.
* **HTTPS Support**: Secure content transmission.
* **Access Control**: Token-based access or signed URLs.
* **Analytics & Logging**: Monitor usage, cache hit/miss, latency.

---

## ✅ 2. Non-Functional Requirements

* **High Availability**: 99.99% uptime via redundant edge locations.
* **Low Latency**: Target <50ms content delivery in most regions.
* **Scalability**: Handle millions of requests per second.
* **Security**: TLS, DDoS protection, origin shielding.
* **Cost-Efficient**: Minimize egress & compute with optimal caching.
* **Fault Tolerance**: Automatic fallback to backup edge/origin.

---

## ✅ 3. Back-of-the-Envelope Estimation

* **Users**: 100M DAUs
* **Avg Requests/User/Day**: 50
* **Total Requests/Day**: 5B
* **Peak RPS**: \~150,000 (conservative estimate)
* **Content Size**: 500KB avg → \~2.5PB/day in bandwidth
* **Edge Cache Hit Ratio**: 90% → Only 10% reach origin

---

## ✅ 4. Database Design

While CDN is mostly file-serving, **metadata, logging**, and **analytics** use persistent storage.

### Tables

#### `content_metadata`

| Field         | Type          |
| ------------- | ------------- |
| content\_id   | UUID (PK)     |
| url           | TEXT          |
| checksum      | STRING        |
| ttl           | INT (seconds) |
| created\_at   | Timestamp     |
| region        | TEXT          |
| cache\_status | ENUM          |

#### `access_logs`

| Field         | Type      |
| ------------- | --------- |
| log\_id       | UUID      |
| timestamp     | TIMESTAMP |
| user\_ip      | STRING    |
| content\_id   | UUID      |
| geo\_location | STRING    |
| status        | INT       |

#### `analytics`

| Field          | Type      |
| -------------- | --------- |
| content\_id    | UUID      |
| hits           | INT       |
| misses         | INT       |
| bandwidth      | BIGINT    |
| last\_accessed | TIMESTAMP |

---

## ✅ 5. API Design

### 📦 Content Management

* `POST /api/content/upload` – Upload new content
* `DELETE /api/content/:id` – Delete content
* `PUT /api/content/:id` – Update metadata (e.g., TTL)

### ⚙️ Cache Management

* `POST /api/cache/purge` – Purge cache globally
* `POST /api/cache/invalidate` – Selective invalidation

### 📈 Analytics & Logging

* `GET /api/logs` – Fetch access logs
* `GET /api/analytics/:id` – Stats for content

### 🔐 Access Control

* `POST /api/security/token` – Generate signed URL
* `POST /api/security/policy` – Set geo/IP restrictions

---

## ✅ 6. High-Level Architecture (with AWS Services)

```
                  +------------------------+
                  |      End Users         |
                  +------------------------+
                            |
                    DNS Lookup (Route53)
                            |
               +------------▼------------+
               | AWS CloudFront (CDN PoPs)|
               +------------+------------+
                            |
      ┌─────────────────────┴─────────────────────┐
      |                                           |
+-------------+                         +-----------------+
|  Regional   |  Cache Hit: Serve       |   Cache Miss:   |
|  Edge Cache |  from edge              |   Forward to:   |
| (CloudFront)|                         |   Origin Server |
+-------------+                         +-----------------+
                                                |
                                 +--------------▼---------------+
                                 |      Origin Servers (S3)     |
                                 |      / ALB + ECS App Server  |
                                 +--------------+---------------+
                                                |
                               +----------------▼---------------+
                               |     Aurora / DynamoDB (Meta)   |
                               +--------------------------------+
```

---

### AWS Services Used

| Feature                         | AWS Service                         |
| ------------------------------- | ----------------------------------- |
| Global DNS                      | Route53                             |
| Edge Caching                    | CloudFront                          |
| Origin Content Storage          | S3                                  |
| Metadata DB                     | DynamoDB or Aurora                  |
| Custom Backend for Dynamic Data | ECS / Lambda / API Gateway          |
| Logging & Analytics             | CloudWatch, Athena, Kinesis         |
| Queueing                        | SQS (e.g., async invalidation jobs) |
| Access Tokens                   | Cognito / Lambda for signed URLs    |
| Monitoring                      | CloudWatch, X-Ray                   |

---

## ✅ 7. Key Issues and Solutions

### 📌 A. **Caching Strategy**

**Challenges**:

* Content freshness
* Cache invalidation
* Regional preferences

**Solutions**:

* Use TTL-based cache expiry (e.g., `Cache-Control: max-age`)
* Use content versioning (e.g., `/v2/image.jpg`)
* Provide purge API to invalidate outdated content
* Use **origin shield** to minimize repeated origin fetches

---

### 📌 B. **Routing & Latency Optimization**

**Challenges**:

* Route users to nearest PoP
* Network congestion

**Solutions**:

* Use **Anycast IP** and **GeoDNS (Route53)** for regional routing
* Implement request coalescing and pre-warming
* Enable compression: Brotli/Gzip
* Leverage CloudFront Regional Edge Caches

---

### 📌 C. **Content Replication**

**Challenges**:

* Propagate updates to edge locations
* Optimize bandwidth and consistency

**Solutions**:

* Use **origin push** to edge caches (on upload or update)
* Store in S3 with **S3 Transfer Acceleration**
* Use **multi-region replication** for hot content

---

### 📌 D. **Handling Failures**

**Challenges**:

* PoP or origin failures

**Solutions**:

* Use **multiple redundant PoPs (CloudFront)**
* Origin failover: S3 → ECS or backup S3 bucket
* Retry with exponential backoff on fetch failure
* Monitor health using AWS CloudWatch alarms and Route53 failover

---

### 📌 E. **Security**

**Challenges**:

* Prevent unauthorized access
* Mitigate DDoS attacks

**Solutions**:

* Enforce HTTPS with TLS certs (ACM + CloudFront)
* Use **signed URLs** or **signed cookies**
* Integrate WAF with IP rate limiting, Geo restrictions
* Use Shield Advanced for DDoS mitigation

---


